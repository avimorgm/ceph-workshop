# Lab 5: RBD

## 5.1 RBD in Ceph

1. Create an RBD pool, type the command:

    ```
    ceph osd pool create rbd 32 32
    ```
    
2. Set the RBD application on this pool:

    ```
    ceph osd pool application enable rbd rbd
    ```
    
3. Create an image in the RBD pool:

    ```
    rbd create --size 2G rbd/rbd_image
    ```
    
4. Create a user that will be able to connect to the RBD pool:

    ```
    ceph auth get-or-create client.rbduser mon 'profile rbd' osd 'profile rbd pool=rbd'
    ```
    
5. Export the key to the Ceph folder:

    ```
    ceph auth get client.rbduser -o /etc/ceph/ceph.client.rbduser.keyring
    ```
    
    Please note, if you are using another client machine, copy this key to the machine you are working on.
    
6. Disable RBD features to work with krbd kernel module:

    ```
    rbd feature disable rbd_image object-map fast-diff deep-flatten
    ```
    
7. List all the RBD images of the created user:

    ```
    rbd ls --user rbduser -p rbd
    ```
    
8. How to map the image onto a device on your machine:

    ```
    rbd map --user rbduser rbd/rbd_image
    ```
    
9. Create a filesystem on the device:

    ```
    mkfs.ext4 /dev/rbd0
    ```
    
10. Make a dir in `/mnt` and mount the device to the directory:

    ```
    mkdir /mnt/mountrbd && mount -t ext4 /dev/rbd0 /mnt/mountrbd
    ```
    
11. Now, you can see the block device on your system:

    ```
    df -h | grep /mnt/mountrbd
    ```
    
## 5.2 Workload test using FIO

1. Let’s create a FIO file for workload testing. Here is the configuration file for FIO:

    ```
    $ vi rbd.fio
    ```
    ```
    [global]
    ioengine=rbd
    clientname=admin
    pool=rbd
    rbdname=rbd_image
    rw=randwrite
    bs=4k
    buffered=0
    runtime=15
    direct=1
    numjobs=1

    [4k-randwrite]
    iodepth=1
    time_based
    ```
    
2. Install FIO:

    ```
    yum install -y fio --skip-broken
    ```
    
3. Run:

    ```
    fio rbd.fio
    ```
    
4. Take note of the latency and IOPs stats.
