# Lab #5: Purge and Create OSD

1. Instsall `jq` on your machine:

    ```
    yum install -y jq   
    ```
    
2. Create a new pool:

    ```
    ceph osd pool create repl_pool 32 32 
    ```
    
3. Write an object to the pool:
  
     ```
   rados put -p repl_pool hosts /etc/hosts
    ```
    
4. List the pool to see the object was successfully created:

     ```
    rados ls -p repl_pool
    ```
    
5. Collect the OSDs LVM paths for this specific host:

    ```
    HOST_LV_PATHS=$(for OSD_INFO in $(ceph-volume lvm list --format json | jq -c '.[]'); do echo ${OSD_INFO} | jq -r '.[].lv_path' ; done)
    ```
    
6. Stop the OSDs on the host and mark them out:

    ```
    for OSD_ID in `ceph osd ls-tree $HOSTNAME`; do systemctl stop ceph-osd@${OSD_ID}; ceph osd out ${OSD_ID}; done
    ```
    
7. We have now 2 avilable copies of the data from 3, but the data is still availbale. Get the object.
   In case we have more than 3 hosts, this spesific replica will create on another host and the "Backfill" proccess will
   start.

    ```
    rados get -p repl_pool hosts hosts_file   
    ```
    
8. Purge the OSDs from the cluster, you should have 4 OSDs afterwards:

    ```
    for OSD_ID in `ceph osd ls-tree $HOSTNAME`; do ceph osd purge ${OSD_ID} --yes-i-really-mean-it; done   
    ```
    
9. Zap the OSDs lvms, this task unmounts the OSD directory and deletes all data from the block device:
    
      ```
    for OSD_ID in $HOST_LV_PATHS; do ceph-volume lvm zap ${OSD_ID}; done  
    ```
    
10. Create the OSDs once again, using the previously collected LVM paths:

    ```
    for OSD_ID in $HOST_LV_PATHS; do ceph-volume lvm create --bluestore --data ${OSD_ID}; done  
    ```

11. Verify you can still read the object the you have uploaded

    ```
    rados get -p repl_pool hosts hosts_file  
    ```
