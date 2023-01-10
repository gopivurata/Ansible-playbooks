
# s1:- (node1)
  * sudo vi /etc/hosts
      # HAproxy 192.168.72.157
  * sudo apt install apache2 -y
  * sudo systemctl enable apache2
  * sudo systemctl start apache2
  * echo "<H1>Hello! This is webserver1: 172.31.84.85 </H1>" | sudo tee /var/www/html/index.html
  * sudo ufw allow 80/tcp
---
# s2:- (node2)
  * sudo apt update
     * sudo vi /etc/hosts
        # HAproxy 192.168.72.157
  * sudo apt install apache2 -y
  * sudo systemctl enable apache2
  * sudo systemctl start apache2
  * echo "<H1>Hello! This is webserver2: 172.31.21.13 </H1>" | sudo tee /var/www/html/index.html
  * sudo ufw allow 80/tcp
---
# ha proxy:- (node3)
* sudo vi /etc/hosts
   172.31.21.93 HAproxy
   172.31.30.212 webserver1
   172.31.21.13 webserver2
* sudo apt-get update
* sudo apt install haproxy -y
* haproxy -v
#  sudo vi "<strong>/etc/haproxy/haproxy.cfg</strong>"
* cd /etc/haproxy/
* sudo vi /etc/haproxy/haproxy.cfg

frontend web-frontend
   bind 172.31.21.93:80
   mode http
   default_backend web-backend
backend web-backend
   balance roundrobin
   server webserver1	172.31.30.212 check
   server webserver2 172.31.21.13 check

listen stats
bind 172.31.21.93:8888
mode http
option forwardfor
option httpclose
stats enable
stats show-legends
stats refresh 5s
stats uri /stats
stats realm Haproxy\ Statistics
stats auth gopi:gopi           #Login User and Password for the monitoring
stats admin if TRUE
default_backend web-backend

* haproxy -c -f /etc/haproxy/haproxy.cfg
* sudo systemctl restart haproxy.service
* sudo systemctl status haproxy.service




 * ansible -i hosts -m setup all
     * ansible_facts:ip-172-31-21-93
         * ansible_hostname: ip-172-31-30-212 (web-server1)
         * ansible_hostname: ip-172-31-21-13 (web-server2)
         * ansible_hostname: ip-172-31-21-93 (haproxy)


         Hello! This is webserver2: 172.31.21.13
         Hello! This is webserver1: 172.31.30.212 




frontend haproxy-main
    bind *:80
    option forwardfor  
    default_backend apache_webservers    

backend apache_webservers
    balance roundrobin
    server webserver1	172.31.30.212 check
    server webserver2	172.31.21.13 check
    

listen stats
bind :8800
stats enable
stats uri /
stats hide-version
stats auth gopivurata:gopivurata
default_backend apache_webservers

