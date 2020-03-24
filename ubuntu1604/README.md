## Ansible Installation

### Ubuntu

### Node

```shell
apt update
apt install -y vim git openssh-server
```

### Kubespray

```shell
sudo apt update
sudo apt install -y vim git openssh-server
sudo apt install -y \
  python3 python-dev python3-dev \
  build-essential libssl-dev libffi-dev \
  libxml2-dev libxslt1-dev zlib1g-dev \
  python-pip python3-pip sshpass

git clone https://github.com/kubernetes-sigs/kubespray.git

cd kubespray
sudo pip3 install -r requirements.txt
sudo pip3 install --upgrade cryptography

# Copy ``inventory/sample`` as ``inventory/mycluster``
cp -rfp inventory/sample inventory/mycluster

# Update Ansible inventory file with inventory builder
declare -a IPS=(10.10.1.3 10.10.1.4 10.10.1.5)
CONFIG_FILE=inventory/mycluster/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}

# Review and change parameters under ``inventory/mycluster/group_vars``
# vim inventory/mycluster/group_vars/all/all.yml
# vim inventory/mycluster/group_vars/k8s-cluster/k8s-cluster.yml
# vim inventory/mycluster/group_vars/k8s-cluster/addons.yml
sed -i '/helm_enabled:/s/false/true/g' inventory/mycluster/group_vars/k8s-cluster/addons.yml
sed -i '/metrics_server_enabled:/s/false/true/g' inventory/mycluster/group_vars/k8s-cluster/addons.yml
sed -i '/ingress_nginx_enabled:/s/false/true/g' inventory/mycluster/group_vars/k8s-cluster/addons.yml
vim inventory/mycluster/inventory.ini
# node1 ansible_host=95.54.0.12 ansible_user=ubuntu ansible_ssh_pass="password" ansible_become_pass="password"

ansible-playbook -i inventory/mycluster/inventory.ini --become --become-user=root ../run.yml

# Deploy Kubespray with Ansible Playbook - run the playbook as root
# The option `--become` is required, as for example writing SSL keys in /etc/,
# installing packages and interacting with various systemd daemons.
# Without --become the playbook will fail to run!
ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root cluster.yml
```


