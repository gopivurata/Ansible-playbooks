* Manual Steps to Install & Configure WildFly (JBoss) on Ubuntu 20.04:
-----------------------------------------------------------------------
* Step 1: Update the system. `sudo apt update`
* Step 2: Install Java. `sudo apt install openjdk-11-jdk -y`
* Step 3: Add a user & group for wildfly.
  * `sudo groupadd -r wildfly`
  * `sudo useradd -r -g wildfly -d /opt/wildfly -s /sbin/nologin wildfly`
* Step 4: Download the WildFly using wget.
  * `cd /tmp`
  * `wget https://download.jboss.org/wildfly/22.0.1.Final/wildfly-22.0.1.Final.tar.gz`
  * Extract the downloaded file.` tar xvf wildfly-22.0.1.Final.tar.gz`
  * Move the extracted folder to /opt/ `sudo mv wildfly-22.0.1.Final/ /opt/wildfly`
  * Provide the following permission: `sudo chown -RH wildfly: /opt/wildfly`
* Step 5: Create a folder. `sudo mkdir -p /etc/wildfly`
  * Copy the following files one place to another & provide executable permission to script files 
    under /opt/wildfly/bin/ folder.
      `sudo cp /opt/wildfly/docs/contrib/scripts/systemd/wildfly.conf /etc/wildfly/`
      `sudo cp /opt/wildfly/docs/contrib/scripts/systemd/launch.sh /opt/wildfly/bin/`
      `sudo sh -c 'chmod +x /opt/wildfly/bin/*.sh'`
      `sudo cp /opt/wildfly/docs/contrib/scripts/systemd/wildfly.service /etc/systemd/system/`
* Step 6: Start & Enable the WildFly service.
  * `sudo systemctl start wildfly.service`
  * `sudo systemctl enable wildfly.service`
  * Check the status of WildFly. `sudo systemctl status wildfly.service`


* Install & Configure WildFly (JBoss) on remote machine using Ansible-playbook:
-------------------------------------------------------------------------------
* update and openjdk-11 installing: we want install wildfly need `java`, WildFly is a Java web application
    * used `apt` module in ansible-playbook 
    ```
        - name: Install openjdk11
          ansible.builtin.apt:
            name: openjdk-11-jdk
            state: present
            update_cache: yes

*  used `group` module in ansible-playbook for creating group wildfly: 
    ```
       - name: creating group
         ansible.builtin.group:
           name: wildfly
           state: present
* used `user` module in ansible-playbook for creating user and adding to group:
  ```
     - name: creating user and add to group
        ansible.builtin.user:
          name: wildfly
          shell: /sbin/nologin
          group: wildfly
          home: /opt/wildfly
          system: true
          state: present
* used `get_url` module in ansible-playbook for downloading the wildfly tar file:
  ```
     - name: Download the WildFly tar file
       ansible.builtin.get_url:
         url: https://download.jboss.org/wildfly/22.0.1.Final/wildfly-22.0.1.Final.tar.gz
         dest: /tmp/wildfly-22.0.1.Final.tar.gz
         mode: '777'
* used `unarchive` module in ansible-playbook for extract the wildfly tar file:
  ```
      - name: Unarchive a file that is already on the remote machine
        ansible.builtin.unarchive:
          src: /tmp/wildfly-22.0.1.Final.tar.gz
          dest: /tmp/
          remote_src: yes
* used `copy` module in ansible-playbook for copy the wildfly configuration files to user home directory:
  ```
     - name: copy the wildfly configuration files to user home directory
       ansible.builtin.copy:
         src: /tmp/wildfly-22.0.1.Final/
         dest: /opt/wildfly
         owner: wildfly
         group: wildfly
         remote_src: yes
* used `file` module in ansible-playbook for Creating a directory:
  ```
     - name: Creating a directory
       ansible.builtin.file:
         path: /etc/wildfly
         state: directory
         mode: '777'
* used `copy` module in ansible-playbook for copy the wildfly.conf to /etc/wildfly/:
  ```
     - name: copy the wildfly.conf
      ansible.builtin.copy:
        src: /opt/wildfly/docs/contrib/scripts/systemd/wildfly.conf
        dest: /etc/wildfly/
        remote_src: yes
* used `copy` module in ansible-playbook for copy the launch.sh to /opt/wildfly/bin/:
  ```
      - name: copy the launch.sh
        ansible.builtin.copy:
          src: /opt/wildfly/docs/contrib/scripts/systemd/launch.sh
          dest: /opt/wildfly/bin/
          remote_src: yes
* used `file` module in ansible-playbook for permissions giving to all sh files
  ```
      - name: permissions given to all sh files
        ansible.builtin.file:
          path: /opt/wildfly/bin/
          owner: wildfly
          mode: '777'
          recurse: yes
* used `copy` module in ansible-playbook for copy the wildfly.service file to /etc/systemd/system/:
  ```
       - name:  copy the wildfly.service
         ansible.builtin.copy:
           src: /opt/wildfly/docs/contrib/scripts/systemd/wildfly.service
           dest: /etc/systemd/system/
           remote_src: yes
* used `systemd` module in ansible-playbook for start and enable the service:
  ```
      - name: wildfly service unit is running and Enable
        ansible.builtin.systemd:
          name: wildfly
          state: started
          enabled: yes


wildfly.yaml
-----------
```
   ---
- name: Install & Configure WildFly
  hosts: all
  become: yes
  tasks:
    - name: Install openjdk11
      ansible.builtin.apt:
        name: openjdk-11-jdk
        state: present
        update_cache: yes
    - name: creating group
      ansible.builtin.group:
        name: wildfly
        state: present
    - name: creating user and add to group
      ansible.builtin.user:
        name: wildfly
        shell: /sbin/nologin
        group: wildfly
        home: /opt/wildfly
        system: true
        state: present
    - name: Download the WildFly tar file
      ansible.builtin.get_url:
        url: https://download.jboss.org/wildfly/22.0.1.Final/wildfly-22.0.1.Final.tar.gz
        dest: /tmp/wildfly-22.0.1.Final.tar.gz
        mode: '777'
    - name: Unarchive a file that is already on the remote machine
      ansible.builtin.unarchive:
        src: /tmp/wildfly-22.0.1.Final.tar.gz
        dest: /tmp/
        remote_src: yes
    - name: copy files to user home directory
      ansible.builtin.copy:
        src: /tmp/wildfly-22.0.1.Final/
        dest: /opt/wildfly
        owner: wildfly
        group: wildfly
        remote_src: yes
    - name: Creating a directory
      ansible.builtin.file:
        path: /etc/wildfly
        state: directory
        mode: '777'
    - name: copy the wildfly.conf
      ansible.builtin.copy:
        src: /opt/wildfly/docs/contrib/scripts/systemd/wildfly.conf
        dest: /etc/wildfly/
        remote_src: yes
    - name: copy the launch.sh
      ansible.builtin.copy:
        src: /opt/wildfly/docs/contrib/scripts/systemd/launch.sh
        dest: /opt/wildfly/bin/
        remote_src: yes
    - name: permissions given to all sh files
      ansible.builtin.file:
        path: /opt/wildfly/bin/
        owner: wildfly
        mode: '777'
        recurse: yes
    - name:  copy the wildfly.service
      ansible.builtin.copy:
        src: /opt/wildfly/docs/contrib/scripts/systemd/wildfly.service
        dest: /etc/systemd/system/
        remote_src: yes
    - name: wildfly service unit is running and Enable
      ansible.builtin.systemd:
        name: wildfly
        state: started
        enabled: yes
        ---
        
with vars: centos-wildfly.yaml
-------------------------------

---
- name: Install & Configure WildFly
  hosts: all
  become: yes
  vars:
    java_version : java-11-openjdk-devel
    wildfly_user: wildfly #user-name and group-name same
    wildfly_home: /opt/wildfly
  tasks:
    - name: Install openjdk11
      ansible.builtin.yum:
        name: "{{ java_version }}"
        state: present
        update_cache: yes
    - name: creating group
      ansible.builtin.group:
        name: "{{ wildfly_user }}"
        state: present
    - name: creating user and add to group
      ansible.builtin.user:
        name: "{{ wildfly_user }}"
        shell: /sbin/nologin
        group: "{{ wildfly_user }}"
        home: "{{ wildfly_home }}"
        system: true
        state: present
    - name: Download the WildFly tar file
      ansible.builtin.get_url:
        url: https://download.jboss.org/wildfly/22.0.1.Final/wildfly-22.0.1.Final.tar.gz
        dest: /tmp/wildfly-22.0.1.Final.tar.gz
        mode: '777'
    - name: Unarchive a file that is already on the remote machine
      ansible.builtin.unarchive:
        src: /tmp/wildfly-22.0.1.Final.tar.gz
        dest: /tmp/
        remote_src: yes
    - name: copy files to user home directory
      ansible.builtin.copy:
        src: /tmp/wildfly-22.0.1.Final/
        dest: "{{ wildfly_home }}"
        owner: "{{ wildfly_user }}"
        group: "{{ wildfly_user }}"
        remote_src: yes
    - name: Creating a directory
      ansible.builtin.file:
        path: /etc/wildfly
        state: directory
        mode: '777'
    - name: copy the wildfly.conf
      ansible.builtin.copy:
        src: /opt/wildfly/docs/contrib/scripts/systemd/wildfly.conf
        dest: /etc/wildfly/
        remote_src: yes
    - name: copy the launch.sh
      ansible.builtin.copy:
        src: /opt/wildfly/docs/contrib/scripts/systemd/launch.sh
        dest: /opt/wildfly/bin/
        remote_src: yes
    - name: permissions given to all sh files
      ansible.builtin.file:
        path: /opt/wildfly/bin/
        owner: "{{ wildfly_user }}"
        mode: '777'
        recurse: yes
    - name:  copy the wildfly.service
      ansible.builtin.copy:
        src: /opt/wildfly/docs/contrib/scripts/systemd/wildfly.service
        dest: /etc/systemd/system/
        remote_src: yes
    - name: wildfly service unit is running and Enable
      ansible.builtin.systemd:
        name: "{{ wildfly_user }}"
        state: started
        enabled: yes

* Now access the wildfly application : `<node-public_ip>@8080`


            
