# Lab 10: Consul on top of the RGW

1. Install git on your machine (not on the Ceph container), using:

    ```
    yum -y install git
    ```
    
2. Once git is installed, download the following repo to your machine from Avi's GitHub:

    ```
    git clone https://github.com/avimorgm/consul.git
    ```
    
3. Install docker-compose on your machine:

    ```
    sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose 
    ```
    
4. Change the permissions:

    ```
    sudo chmod +x /usr/local/bin/docker-compose
    ```
    
5. Verify the docker-compose is installed correctly:
  
    ```
    docker-compose --version
    ```
    
6. On your machine, create the following directories:

    ```
    mkdir /etc/consul.d
    ```
    ```
    mkdir /etc/consul
    ```
    ```
    mkdir /tmp/consul
    ```
    ```
    mkdir /root/consul
    ```
    
7. Change the IP address and port number on `rgw1.json` to be in accordance to your cluster and then copy the files to the Consul directories:

    ```
    cd /root/consul/consul-demo-master
    ```
    ```
    vi rgw1.json
    ```
    ```
    { 
      "service": 
      { 
        "id": "ceph-nano-ceph", 
        "name": "rgws-floating-name", 
        "address": "<rgw-endpoint-without-http-and-without-port>", 
        "port": <rgw-endpoint-port>, 
        "check": 
        { 
          "http": "<rgw-endpoint>", 
          "interval": "10s", 
          "timeout": "1s" 
        } 
      } 
    }
    ```
    
    ```
    cp rgw1.json default.json /etc/consul.d
    ```
    ```
    cp docker-compose.yml /root/consul
    ```
    
8. Run the docker-compose file:

    ```
    cd /root/consul
    ```
    ```
    docker-compose up -d
    ```
    
9. Edit the `resolv.conf` file and keep only the following line (hash - # - the rest):

    ```
    vi /etc/resolv.conf
    ```
    ```
    nameserver 127.0.0.1
    ```
    
10. Check that a resolve exists:

    ```
    ping rgws-floating-name.service.domain1
    ```
    
    The `rgws-floating-name` is the service name that can have a lot of RGWs behind it. The suffix is part of Consul tagging it as a service and the domain name is from the `default.json`.
    
11. Stop the container, try again to ping the address. Now, the ping will not work:

    ```
    docker stop <container-name>
    ```
    
    Generally you have a lot of RGWs behind `rgws-floating-name`. So if one of the RGWs is down, the others should respond.
    
12. Start again the Ceph container, the ping should be back:

    ```
    docker restart <container-name>
    ```
