# Lab #3: Librados

1. In this lab we are going to cover the Ceph architecture. There are 3 layers in Ceph:

    ![Ceph](https://raw.githubusercontent.com/avimorgm/k8s/master/images/blockStorage_orang23e_232320x242_0.png)

  - **RADOS:** Reliable Autonomous Distributed Object Store. The Ceph storage is built on an object store. So this layer manages all of the objects.
  - **LIBRADOS:** Cpeh native API. This layer represents the ability to work directly with the RADOS layer. There are libraries available for C++, Java, Python, Ruby, Erlang, and PHP. In our Demo, we will use Python for the exercise.
  - **RGW:** Rados Gateway, the Object interface. When uploading an object to the Ceph cluster there is an interaction with this layer.
  - **RBD:** Block storage for the client. We need to remember that in the end, also the block device is cut into pieces and is saved as objects on the RADOS layer.
  - **CephFS:** A distributed POSIX-compliant file system, built on top of Ceph's distributed object store, RADOS.
  
2. Write a Python code for connecting the cluster.

3. Typically, with this API you can do things like: 

  - List pools
  - Create a pool
  - Delete a pool
  
  4. Connect to the Ceph cluster:
  
      ```
      ./cn cluster enter master
      ```
      
5. Change the directory to the Ceph directory:

    ```
    cd /etc/ceph
    ```
    
6. Create a file and copy the following there:

    ```
    $ vi librados_test.py
    ```
    ```
 import rados, sys

cluster = rados.Rados(conffile='ceph.conf')
print (cluster.version())
print (cluster.conf_get('mon initial members'))

cluster.connect()
print (cluster.get_fsid())

print ("\n\nPool Operations")
print ("===============")

print ("\nAvailable Pools")
print ("----------------")
pools = cluster.list_pools()

for pool in pools:
        print (pool)

print ("\nCreate 'oren' Pool")
print ("------------------")
try:
  cluster.create_pool('oren')
except:
  print("Pool already exists!")

print ("\nVerify 'oren' Pool Exists")
print ("-------------------------")
pools = cluster.list_pools()

for pool in pools:
        print (pool)
    ```
    
8. To run the script:

    ```
    python librados_test.py
    ```
