* Apache Kafka install on Ubuntu -Manual steps:
--------------------------------------
  * Step 1: Update the System and Install Java:
     *` sudo apt update`
     * `sudo apt install openjdk-11-jdk -y`
  * Step 2: Download the Apache Kafka:
     * `wget https://downloads.apache.org/kafka/3.3.1/kafka_2.13-3.3.1.tgz`
  * Extract the downloaded file:
     * `tar xzf kafka_2.13-3.3.1.tgz`
  * Step 3: Move the extracted folder to /usr/local/:
     * `sudo mv kafka_2.13-3.3.1 /usr/local/kafka`
  * Step 4: Create a new systemd unit files for Zookeeper service:
     * `sudo vi /etc/systemd/system/zookeeper.service`
  * Step 5: Create a new systemd unit files for Kafka service:
     * `sudo vi /etc/systemd/system/kafka.service`
  * Step 6: Reload the systemd daemon & Start the ZooKeeper service:
     * `sudo systemctl daemon-reload`
     * `sudo systemctl start zookeeper`
  * Step 7: Start & Enable the Kafka service:
     * `sudo systemctl start kafka`
     * `sudo systemctl enable kafka`

* Apache Kafka installing on remote server using Ansible-playbook:
------------------------------------------------------------------

