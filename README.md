# Kolla Ansible
### The Kolla Ansible is a deliverable project separated from Kolla project.

- Kolla Ansible deploys OpenStack services and infrastructure components in Docker containers.

### Kolla's mission statement is:

- To provide production-ready containers and deployment tools for operating OpenStack clouds.

- Kolla is highly opinionated out of the box, but allows for complete customization. This permits operators with little experience to deploy OpenStack quickly and as experience grows modify the OpenStack configuration to suit the operator's exact requirements.


### kolla-ansible auto script by suppi147
`
   #!/bin/bash
   COLOR='\033[1;36m' #cyan
   NC='\033[0m'

   if [ "$EUID" -ne 0 ]
   then echo "Please run as root"
   exit
   fi


   echo -e "${COLOR}executing: sudo apt update && sudo apt upgrade -y${NC}"
   echo -e "${COLOR}executing: sudo apt install git python3-dev libffi-dev gcc libssl-dev -y${NC}"
   echo -e "${COLOR}executing: sudo apt install python3-pip -y${NC}"
   echo -e "${COLOR}executing: pip3 install -U pip${NC}"
   sudo apt update && sudo apt upgrade -y
   sudo apt install git python3-dev libffi-dev gcc libssl-dev -y
   sudo apt install python3-pip -y
   pip3 install -U pip

   echo -e "${COLOR}executing: pip3 install 'ansible>=6,<8'${NC}"
   pip install 'ansible>=6,<8'

   echo -e "${COLOR}executing: git clone https://github.com/openstack/kolla-ansible${NC}"
   git clone https://github.com/suppi147/kolla-ansible

   echo -e "${COLOR}executing: pip install git+https://opendev.org/openstack/kolla-ansible@stable/2023.1${NC}"
   pip install git+https://opendev.org/openstack/kolla-ansible@stable/2023.1

   echo -e "${COLOR}executing: sudo apt install docker.io${NC}"
   sudo apt install docker.io

   echo -e "${COLOR}executing: kolla-ansible install-deps${NC}"
   kolla-ansible install-deps

   echo -e "${COLOR} executing: creating /etc/kolla${NC}"
   if ! [ -d "/etc/kolla" ]; then
         sudo mkdir -p /etc/kolla
         sudo chown $USER:$USER /etc/kolla
   fi
   export kolla_home=$(pwd)

   echo -e "${COLOR} executing:cp -r $kolla_home/kolla-ansible/etc/kolla/* /etc/kolla${NC}"
   cp -r $kolla_home/kolla-ansible/etc/kolla/* /etc/kolla

   echo -e "${COLOR} executing:cp $kolla_home/kolla-ansible/ansible/inventory/all-in-one .${NC}"
   cp $kolla_home/kolla-ansible/ansible/inventory/all-in-one .

   #echo -e "${COLOR} executing:kolla-ansible install-deps${NC}"
   #kolla-ansible install-deps

   echo -e "${COLOR} executing:echo '[defaults]\nhost_key_changing=False\npipelining=True\nforks=100' > ansible.cfg${NC}"
   if ! [ -d /etc/ansible/ ]; then
         mkdir /etc/ansible/
         chown $USER:$USER /etc/ansible/
         touch /etc/ansible/ansible.cfg
   fi
   cat > /etc/ansible/ansible.cfg <<EOF
   [defaults]
   host_key_changing=False
   pipelining=True
   forks=100
   EOF

   echo -e "${COLOR} executing:kolla-genpwd${NC}"
   kolla-genpwd

   touch globals.yml
   filename='globals.yml'
   cp /etc/kolla/globals.yml $kolla_home/globals.yml
   search='#kolla_base_distro: "rocky"'
   replace='kolla_base_distro: "ubuntu"'
   echo -e "${COLOR} executing:sed -i 's/$search/$replace/' $filename${NC}"
   if [[ $search != "" && $replace != "" ]]; then
   sed -i "s/$search/$replace/" $filename
   fi
   cp $kolla_home/globals.yml /etc/kolla/globals.yml
   search='#network_interface: "eth0"'
   read -p "Enter the base interface(enp0s8 for example): " interface1
   replace='network_interface: "'$interface1'"'
   echo -e "${COLOR} executing:sed -i 's/$search/$replace/' $filename${NC}"
   if [[ $search != "" && $replace != "" ]]; then
   sed -i "s/$search/$replace/" $filename
   fi
   cp $kolla_home/globals.yml /etc/kolla/globals.yml
   search='#neutron_external_interface: "eth1"'
   read -p "Enter the neutron interface(enp0s3 for example): " interface2
   replace='neutron_external_interface:: "'$interface2'"'
   echo -e "${COLOR} executing:sed -i 's/$search/$replace/' $filename${NC}"
   if [[ $search != "" && $replace != "" ]]; then
   sed -i "s/$search/$replace/" $filename
   fi
   cp $kolla_home/globals.yml /etc/kolla/globals.yml
   ip_address=$(echo $(ip addr show $interface2) | grep -oP 'inet \K[\d.]+')
   search='#kolla_internal_vip_address: "10.10.10.254"'
   replace='kolla_internal_vip_address: "'$ip_address'"'
   echo -e "${COLOR} executing:sed -i 's/$search/$replace/' $filename${NC}"
   if [[ $search != "" && $replace != "" ]]; then
   sed -i "s/$search/$replace/" $filename
   fi
   cp $kolla_home/globals.yml /etc/kolla/globals.yml
   search='#enable_cinder: "no"'
   replace='enable_cinder: "no"'
   echo -e "${COLOR} executing:sed -i 's/$search/$replace/' $filename${NC}"
   if [[ $search != "" && $replace != "" ]]; then
   sed -i "s/$search/$replace/" $filename
   fi
   cp $kolla_home/globals.yml /etc/kolla/globals.yml
   search='#enable_haproxy: "yes"'
   replace='enable_haproxy: "no"'
   echo -e "${COLOR} executing:sed -i 's/$search/$replace/' $filename${NC}"
   if [[ $search != "" && $replace != "" ]]; then
   sed -i "s/$search/$replace/" $filename
   fi
   cp $kolla_home/globals.yml /etc/kolla/globals.yml

   echo -e "${COLOR} executing: kolla-ansible -i ./all-in-one bootstrap-servers${NC}"
   kolla-ansible -i ./all-in-one bootstrap-servers
   `