* Deploying jenkins war file on ubuntu(manually):
------------------------------------------------
  * `sudo apt update`
  * `sudo apt install openjdk-11-jdk -y`
  * `wget https://updates.jenkins.io/download/war/2.384/jenkins.war`
  * we want to run jenkins application ,we need `jenkins service` file 
    `sudo vi /etc/systemd/system/jenkins.service`
  * after then start service  `sudo systemctl restart jenkins.service`

  

* Deploying jenkins war file on remote machines using `ansible-playbook`:
--------------------------------------------------------------------------
 * update and openjdk-11 installing: we want install jenkins need java, jenkins is a Java web 
   application
   * used `apt` module in ansible-playbook for `Update the repository cache and installing java-11`
      * ```
          - name: Update the repository cache and installing java
            ansible.builtin.apt:
                name: openjdk-11-jdk
                state: present
                update_cache: yes
    * used `get_url` module in ansible-playbook for `Downloading jenkins war file` in /home/web/
      * ```
          - name: Downloading jenkins war file
            ansible.builtin.get_url:
                url: https://updates.jenkins.io/download/war/2.384/jenkins.war
                dest: /home/web/
    * used `copy` module in ansible-playbook for `copy the jenkins service file` in remote machine 
      location
       `/etc/systemd/system/jenkins.service`
       * ```
           - name: Copying the jenkins service file to remote machine
             ansible.builtin.copy:
               src: jenkins.service
               dest: /etc/systemd/system/jenkins.service
    * used `systemd` module in ansible-playbook for restart the jenkins service
      * ```
          - name: restart the jenkins service
            ansible.builtin.systemd:
                state: restarted
                daemon_reload: yes
                name: jenkins.service



    * jenkins.yaml
    ---------------
       * ```
                    ---
            - name: deploying jenkins war file
            hosts: all
            become: yes
            tasks:
                - name: Update the repository cache and installing java
                ansible.builtin.apt:
                    name: openjdk-11-jdk
                    state: present
                    update_cache: yes
                - name: Downloading jenkins war file
                ansible.builtin.get_url:
                    url: https://updates.jenkins.io/download/war/2.384/jenkins.war
                    dest: /home/web/
                - name: Copying the jenkins service file to remote machine
                ansible.builtin.copy:
                    src: jenkins.service
                    dest: /etc/systemd/system/jenkins.service
                - name: restart the jenkins service
                ansible.builtin.systemd:
                    state: restarted
                    daemon_reload: yes
                    name: jenkins.service

* jenkins.service:
------------------
    * ```
                [Unit]
        Description=jenkins java application
        [Service]
        User=web
        # The configuration file application.properties should be here:

        #change this to your workspace
        WorkingDirectory=/home/web/

        #path to executable.
        #executable is a bash script which calls jar file
        ExecStart=/usr/bin/java -jar jenkins.war

        SuccessExitStatus=143
        TimeoutStopSec=10
        Restart=on-failure
        RestartSec=5

        [Install]
        WantedBy=multi-user.target

![preview](jenkins.png)