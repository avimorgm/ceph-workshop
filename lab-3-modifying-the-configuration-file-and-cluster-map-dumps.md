# Lab 4: Modifying the Configuration File and Cluster Map Dumps

## 4.1 Deleteing a pool

1. Login to the cluster:

    ```
    ./cn cluster enter master
    ```
    
2. Create a pool:

    ```
    ceph osd pool create new_pool 32 32
    ```
    
3. To modify the configuration file, edit the `/etc/ceph/ceph.conf` file. The Ceph configuration file contains few sections, like: `[general]`, `[osds]`, `[mons]`, `[rgws]` and more.

4. In order to delete a pool, the value `mon_allow_pool_delete` should be set to `true`. You can check its current value by running:

    ```
    ceph daemon mon.$HOSTNAME config get mon_allow_pool_delete
    ```
    ```
    "mon_allow_pool_delete": "false"
    ```
    
5. To delete a pool, please modify the `[general]`, and add the following `key:value` to the section on `/etc/ceph/ceph.conf`:

    ```
    vi /etc/ceph/ceph.conf
    ```
    ```
    ...
    [global]
    mon_allow_pool_delete = true
    ...
    ```
    
6. Once you've changed it, restart the container for the changes to take effect:

    ```
    docker restart <container_name>
    ```
    
7. Check the value has been changed:

    ```
    ceph daemon mon.mon0 config get mon_allow_pool_delete
    ```
    ```
    "mon_allow_pool_delete": "true"
    ```
    
8. Now you should able to remove the pool you created at beginning of the lab:

    ```
    ceph osd pool delete new_pool new_pool --yes-i-really-really-mean-it
    ```
    
9. How to inject these values in run-time with the `ceph tell` command? Let's set the value back to false:

    ```
    ceph tell mon.$HOSTNAME injectargs '--mon_allow_pool_delete=false'
    ```
    Please remember that this value is non-persistent with `injectargs` command.
    
## 4.2 Debug level change

1. Change the debug of osd daemon. You can do it with the `injectargs`/`config set` commands or with edit the configuration file. Please note, if you make it with changing the Ceph configuration file, please change/add the lines under the global section.

2. To inject the osd in real-time, type the following command on the OSD node:

    ```
    ceph tell osd.0 injectargs "--debug_osd 10"
    ceph tell osd.0 injectargs "--debug_ms 1"
    ```
    
3. Another way is to edit the configuration file, and add the lines under the global section:

    ```
    vi /etc/ceph/ceph.conf
    ```
    ```
    ...
    [global]
    debug_osd = 10
    debug_ms = 1
    ...
    ```
    
## 4.3 Cluster Maps

1. Dump cluster maps with:

    ```
    ceph mon dump
    ```
    ```
    ceph osd dump
    ```
    ```
    ceph pg dump
    ```
    ```
    ceph fs dump
    ```
    ```
    ceph mgr dump
    ```
    ```
    ceph service dump
    ```
