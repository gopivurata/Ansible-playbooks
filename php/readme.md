* installing Apache2 and php on remote machine using Ansible-playbook:
  ------------------------------------------------------------------
  * we need (vms):
  -----------------
    * we want minumum two virtual servers or physical servers, 1.ansible control node and 2.node
    * one Ansible_control_node:
      * we should install ansible on Ansible_control_node
         *  `sudo apt update`
            `sudo apt install software-properties-common`
            `sudo add-apt-repository --yes --update ppa:ansible/ansible`
            `sudo apt install ansible -y`
            `ansible --version`

    * one or more nodes:
      * we should install 
         * `python3 --version`
           ` pip3 --version`
           ` python3 -m pip install --user ansible`

* Ansible Setup with Key Based Authentication between `Ansible_control_node` and `Node`:
----------------------------------------------------------------------------------------
  * Create two linux vm’s
  * Create a user on both vms with password `adduser <user-name>`
  * aws machines have passwords disabled by default ,
    so lets enable passwords`sudo vi /etc/ssh/sshd.config`---> PasswordAuthentication:`no --> yes` 
    `sudo systemctl restart sshd`
  * add user to sudoers file for sudo permissions `sudo visudo`
  * create  `ssh key pair` on Ansible control node `ssh-keygen`
  * copy the public key from Ansible control node to node `ssh-copy-id username@pvt-ip`
  * login from ansible control node to node without password `ssh <username>@pvtip`

* Create an inventory file with node private ip `echo <pvt-ip> > hosts`
* ping the inventory file `ansible -m ping -i hosts all` 

* Installing `apache server`,`php` and creating an extra `info.php`file with content page:
   * manual steps
      * `sudo apt update`
      *  `sudo apt install apache2 -y`
      *  `sudo apt install php libapache2-mod-php php-mysql`
      * `sudo nano /var/www/html/info.php`
          <?php
            phpinfo();
          ?>
      * access php in browser `<public-ip>/info.php`

   * deploy the application on node to use `ansible-playbook`
      * php.yaml:
       ----------
       ---
        - name: deploy the php
          hosts: all
          become: yes
          tasks:
            - name: Installing "apache2" 
              ansible.builtin.apt:
                name: apache2
                update_cache: yes
                state: latest
            - name: Installing "php" 
              ansible.builtin.apt:
                name:
                  - php
                  - libapache2-mod-php
                  - php-mysql
                update_cache: yes
                state: latest
            - name: coping the php.info file to node
              ansible.builtin.copy:
                src: php.info
                dest: /var/www/html/info.php
                follow: yes
      ---
  * in the php.yaml file iam used 3 ansible-modules
     * `ansible.builtin.apt:` for installing `apache2` in ubuntu os
     * `ansible.builtin.apt:` for installing `php` in ubuntu os
     * `ansible.builtin.copy:` for coping the php.info file to node
  * php.yaml file syntax checking `ansible-playbook --syntax-check -i hosts php.yaml`
  * run the playbook for deploy the php in node `ansible-playbook -i hosts php.yaml`
  * access php in browser take node public-ip `<public-ip>/info.php`


  
        