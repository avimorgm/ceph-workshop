# Lab 5: Purge and Create OSD

## 5.1 Create OSD in Ceph

1. Instsall JQ on the machine:

    ```
    ceph osd pool create rbd 32 32
    ```
2. Create a new pool:

    ```
    ceph osd pool create repl_pool 32 32 
    ```
    
3. rados put -p replication hosts /etc/hosts
  
     ```
    ceph osd pool create repl_pool 32 32 
    ```
4. List the pool to see the object was successfully created

     ```
    rados ls -p repl_pool
    ```
5. Collect the OSDs lvm paths for this specific host

    ```
    HOST_LV_PATHS=$(for OSD_INFO in $(ceph-volume lvm list --format json | jq -c '.[]');do echo ${OSD_INFO} | jq -r      '.[].lv_path' ;done) repl_pool
    ```
6. Stop the OSDs and mark them out

     ```
    for OSD_ID in `ceph osd ls-tree $HOSTNAME`;do systemctl stop ceph-osd@${OSD_ID}; ceph osd out ${OSD_ID};done
    ```
7. Try downloading the object once again to verify that although ⅓ of the data is unavailable, you can still access it.

     ```
    rados get -p repl_pool hosts hosts_file   
    ```
    
8. Purge osds from the clusters, you should 4 OSDs now

    ```
    for OSD_ID in $HOST_LV_PATHS;do ceph-volume lvm zap ${OSD_ID};done  
    ```
9. Create the OSDs once again given the previously collected lvm paths

    ```
     for OSD_ID in $HOST_LV_PATHS;do ceph-volume lvm create --bluestore --data ${OSD_ID};done
    ```
10. Verify you can still read the object the you have uploaded

    ```
    rados get -p repl_pool hosts hosts_file  
    ```

    
