# Install guide
Install from haproxy machine

### 1) supo apt-get install git 
  Install a git to clone Repo

### 2) sudo mkdir /kube
  make directory to clone

### 3) sudo git clone https://github.com/Skatanic6996/kubespray

sudo chmod -R 777 /kube 

to read data correctly during installation

### 4) sudo nano /kube/kubespray/inventory/mycluster/hosts.yaml
  enter you names and IPs
  like:

``` js
all:
  hosts:
    my-best-master01:
      ansible_host: 192.168.0.1
      ip: 192.168.0.1
      acces_ip: 192.168.0.1
``` 
### 5) sudo nano /kube/kubespray/inventory/mycluster/group_vars/all/all.yml
  add your haproxy VIP like this :
``` js
loadbalancer_apiserver:
  address: 192.168.0.3
  port: 6443
```
### 6) Share your certs (from root)
ssh-keygen -t rsa

ssh-copy-id root@<your_node_ip>

try connect: ssh <your_node_ip>

### 7) Install requirements

sudo apt update 

sudo apt install software-properties-commosudo apt update 

sudo apt install python3.8 

sudo apt install python3-pip 

### 8) Go to main folder (cd /kube/kubespray)

### 9) Install requirements part 2
``` js
sudo pip3 install –r requirements.txt 
```

### 10) Install cluster
``` js
ansible-playbook -i inventory/mycluster/hosts.yaml  --become --become-user=root cluster.yml --extra-vars "ansible_sudo_pass=<your_root_password>" --timeout 180
```



## Notes

If you need to install the cluster using http \ https proxy - you need to add these parameters before installation.

sudo nano inventory/mycluster/group_vars/all/all.yml
``` js
http_proxy: "192.168.0.1"
https_proxy: "192.168.0.1"
```

## HAproxy keepalived

aptitude install keepalived

To allow HAProxy to bind to the shared IP address, we add the following line to /etc/sysctl.conf:

nano /etc/sysctl.conf

[...]

net.ipv4.ip_nonlocal_bind=1

... 

and run:

sysctl -p

Next we must configure keepalived (this is done through the configuration file /etc/keepalived/keepalived.conf). I want lb1 to be the active (or master) load balancer, so we use this configuration on lb1:

lb1:

nano /etc/keepalived/keepalived.conf

``` js
vrrp_script chk_haproxy {           # Requires keepalived-1.1.13
        script "killall -0 haproxy"     # cheaper than pidof
        interval 2                      # check every 2 seconds
        weight 2                        # add 2 points of prio if OK
}

vrrp_instance VI_1 {
        interface eth0
        state MASTER
        virtual_router_id 51
        priority 101                    # 101 on master, 100 on backup
        virtual_ipaddress {
            192.168.0.99
        }
        track_script {
            chk_haproxy
        }
}
```

(It is important that you use priority 101 in the above file - this makes lb1 the master!)

Then we start keepalived on lb1:

lb1:

/etc/init.d/keepalived start

Then run:

lb1:

ip addr sh eth0

lb1: ip addr sh eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN qlen 1000
    link/ether 00:0c:29:63:f7:5c brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.100/24 brd 192.168.0.255 scope global eth0
    inet 192.168.0.99/32 scope global eth0
    inet6 fe80::20c:29ff:fe63:f75c/64 scope link
       valid_lft forever preferred_lft forever
lb1:

Now we do almost the same on lb2. There's one small, but important difference - we use priority 100 instead of priority 101 in /etc/keepalived/keepalived.conf which makes lb2 the passive (slave or hot-standby) load balancer.

After setting up 2nd LB : /etc/init.d/haproxy start 
