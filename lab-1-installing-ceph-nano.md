# Lab #1: Installing Ceph-Nano
1. Login to your machine.
2. Install docker:

   ```
   https://docs.docker.com/engine/install/centos/
   ```

3. Run this command to downlaod `ceph-nano`:
    ```
    curl -L https://github.com/ceph/cn/releases/download/v2.3.1/cn-v2.3.1-linux-amd64 -o cn && chmod +x cn
    ```
    
4. Create two clusters:

    ```
    ./cn cluster start -d /tmp -i registry.access.redhat.com/rhceph/rhceph-3-rhel7 master
    ./cn cluster start -d /tmp -i registry.access.redhat.com/rhceph/rhceph-3-rhel7 slave
    ```
    
    Master output should be something like (depends on your subnet):
    
    ```
    Endpoint: http://10.0.2.15:8000
    Dashboard: http://10.0.2.15:5000
    Access key: OOSBUFGMB58VI8V3AXWT
    Secret key: oisU43pyi6POmVGm0oMqRwgj1x7MmvaHiaIc0MOg
    Working directory: /tmp
    ```
    
    Slave output should be something like (depends on your subnet):
    
    ```
    Endpoint: http://10.0.2.15:8001
    Dashboard: http://10.0.2.15:5001
    Access key: U6D9I5TDCMZ79NP7N3C7
    Secret key: wM8oEhE06Qg61Pt1P99ArSy4s4G2v4QvOnO1Yc07
    Working directory: /tmp
    ```
    
5. Login to the cluster with:

    ```
    ./cn cluster enter <cluster-name>
    ```
    
    Or with:
    
    ```
    docker exec -it <container-name> bash
    ```
    
6. Check the status of the cluster with:

    ```
    ceph -s
    ```
    
7. To exit the container, type `ctrl+d`.
