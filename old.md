
# Ansible: Installation demo

```shell
vagrant up webserver
vagrant up ansible
vagrant ssh ansible

python -m pip install ansible
sudo apt-get install ansible
ssh-keygen

Your identification has been saved in /home/vagrant/.ssh/id_rsa
Your public key has been saved in /home/vagrant/.ssh/id_rsa.pub

sudo echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCZ5ZfCIuLxsBib7xoGDgQfD9xVrugdmXNs9A6mF1U6S5MGCm3Bzj5JpTBRgVol0Cx6q1Hdf9llBWK70veNx9T73mIMq50xR1+cFzyd6RYOcYrPnyO7ZhMnDZY4B+6YYg4qgMJ038g41sWsKtR40eIaRXIYaSl3SZtj5iZWZuaC+6etJVx3kThz9v5rZeK9YdY1H8djOaGeliCvMN0nvnKyMBQsXhl0raFwOLz5/jdAnf6nVRfKLX6xdd7fJGYkk1t7UlMOPf7oPEXc7Ydvm1/+ztdUo4+UxdZLBXJ5KCM2ZYWGOHoy+u6KlV+9IVwHa5li1xeghfeJ5QnKUF1tBsDn vagrant@ansible' > /home/vagrant/.ssh/authorized_keys

echo '[webservers]' > hosts
echo '192.168.0.2' >> hosts

ssh-agent bash 
ssh-add .ssh/id_rsa

ansible -i hosts -u vagrant -m ping all

cp /vagrant/nginx.yml .
cp /vagrant/nginx.conf.j2 .

ansible-playbook -i hosts -u vagrant nginx.yml
```


# Install wordpress
```shell
sudo apt-get install ansible -y 
ansible --version
mkdir wordpress-ansible && cd wordpress-ansible
touch playbook.yml
touch hosts
mkdir roles && cd roles

ansible-galaxy init server 
ansible-galaxy init php 
ansible-galaxy init mysql
ansible-galaxy init wordpress


nano /vagrant/wordpress-ansible/playbook.yml
- hosts: wordpress
  roles:
    - server
    - php
    - mysql
    - wordpress
	

echo '[wordpress]' > /vagrant/wordpress-ansible/hosts
echo '127.0.0.1  ansible_connection=local' >> /vagrant/wordpress-ansible/hosts
nano /vagrant/wordpress-ansible/hosts


echo '[wordpress]' > /vagrant/wordpress-ansible/hosts
echo '127.0.0.1  ansible_connection=local' >> /vagrant/wordpress-ansible/hosts
nano /vagrant/wordpress-ansible/hosts


	
ansible -i hosts all -m ping

Server
nano /vagrant/wordpress-ansible/roles/server/tasks/main.yml

PHP
nano /vagrant/wordpress-ansible/roles/php/tasks/main.yml

MySQL
nano /vagrant/wordpress-ansible/roles/mysql/defaults/main.yml

nano /vagrant/wordpress-ansible/roles/mysql/tasks/main.yml

WordPress
nano /vagrant/wordpress-ansible/roles/wordpress/tasks/main.yml
nano /vagrant/wordpress-ansible/roles/wordpress/handlers/main.yml
nano /vagrant/wordpress-ansible/roles/wordpress/tasks/main.yml


ansible-playbook playbook.yml -i hosts -u vagrant
```

run command modules: command, shell, script, raw 


# ad-hoc commands

```shell
ansible all -m ping
ansible web -m command -a "uptime"
ansible localhost -m setup

some_host         ansible_port=2222     ansible_user=manager
aws_host          ansible_ssh_private_key_file=/home/example/.ssh/aws.pem
freebsd_host      ansible_python_interpreter=/usr/local/bin/python
ruby_module_host  ansible_ruby_interpreter=/usr/bin/ruby.1.9.3

ansible_host:     The name of the host to connect to, if different from the alias you wish to give to it.
ansible_port:     The ssh port number, if not 22
ansible_user:     The default ssh user name to use.
ansible_ssh_pass:    The ssh password to use (never store this variable in plain text; always use a vault. See Variables and Vaults)
ansible_ssh_private_key_file:     Private key file used by ssh. Useful if using multiple keys and you don’t want to use SSH agent.
```

# ad-hoc commands

```shell
ansible all -i hosts -u vagrant -m ping 
ansible all -i hosts -u vagrant -m setup
ansible webservers -i hosts -u vagrant -m yum -a "name=httpd state=present" -b
ansible webservers -i hosts -u vagrant -m yum -a "name=httpd state=absent" -b
```

- tasks: run sequentialy 
- handlers: special tasks where it run only on notification however it triggers one time at the end of the play
