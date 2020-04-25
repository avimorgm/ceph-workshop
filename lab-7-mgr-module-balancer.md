# Lab 7: MGR Module - Balancer

1. Login to the cluster:

    ```
    ./cn cluster enter master
    ```
    
2. Type the following command to see the distribution of PGs across OSDs:

    ```
    ceph osd df
    ```
    
    Take a look at the PGs column, you can see that the PGs are not distributed well in the cluster. There are OSDs that have more PGs than others, so the balance in the cluster is not perfect.
    
4. Enable the Balancer module on the MGR:

    ```
    ceph mgr module enable balancer
    ```
    
5. Start the balancer module:

    ```
    ceph balancer on
    ```
    
6. Set minimum client `luminous` field to true:

    ```
    ceph osd set-require-min-compat-client luminous --yes-i-really-mean-it
    ```
    
7. Enable the `upmap` mode:

    ```
    ceph balancer mode upmap
    ```
    
8. Restart the MGR service:

    ```
    systemctl restart ceph-mgr@$HOSTNAME
    ```
    
9. Set balancer module off and on:

    ```
    ceph balancer off && ceph balancer on
    ```
    
10. Wait a few minutes and check again the distribution of the PGs:

    ```
    ceph osd df
    ```
