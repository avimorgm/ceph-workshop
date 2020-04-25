# Lab 2: Replicated Pools and Erasure Code Pools

## 2.1 Replicated Pools

1. Login to the master container:

    ```
    ./cn cluster enter master
    ```
    
2. How to create a replicated pool?

    ```
    ceph osd pool create demo 32 32
    ```
    
    Please note that the command creates the pool with the default `crush_rule`.
    
3. How to enable tagging on the pool?

    ```
    ceph osd pool application enable demo rgw
    ```
    
4. How to list all the pools?

    ```
    ceph osd pool ls
    ```
    
    You will see the pool that you've just created.
    
5. How to rename the pool name to another name?

    ```
    ceph osd pool rename demo demotwo
    ```
    
6. How to get details about the pools?

    ```
    ceph osd pool ls detail
    ```
    
    The output of the previous command would be something like this:
    
    ```
    pool 6 'demotwo' replicated size 1 min_size 1 crush_rule 0 object_hash rjenkins pg_num 32 pgp_num 32 last_change 24 flags hashpspool stripe_width 0 application rgw
    ```
    
    Explanation:
    
    - `pool 6` - is the ID of the pool;
    - `replicated` - means that it's a replication pool;
    - `size 1` - 1 replicas;
    - `min_size 1` - the minimum number of available replicas needed for I/O to be served is 1;
    - `crush_rule 0` - is the crush rule ID;
    - `pg_num 32` - there are 32 PGs in the pool;
    - `pgp_num 32` - 32 PGs will be considered for placement by the CRUSH algorithm;
    - `application rgw` - the pool tag
    
7. In order to be able to see the contents of all the CRUSH Rules in the cluster, type the following command:

    ```
    ceph osd crush rule dump
    ```
    
    Output should look like this:
    
    ```
    [
      {
        "rule_id": 0,
        "rule_name": "replicated_rule",
        "ruleset": 0,
        "type": 1,
        "min_size": 1,
        "max_size": 10,
        "steps": [
            {
                "op": "take",
                "item": -1,
                "item_name": "default"
            },
            {
                "op": "choose_firstn",
                "num": 0,
                "type": "osd"
            },
            {
                "op": "emit"
            }
        ]
      }
    ]
    ```
    
8. You can reduce the `min_size`. This is a very risky thing to do in production, but for our purposes

    ```
    ceph osd pool set demotwo min_size 1
    ```
    
9. Verify the change has been made:

    ```
    ceph osd pool ls detail
    ```
    
10. How to create object in a specific pool:
    
    ```
    rados put hosts /etc/hosts -p demotwo
    ```
    
11. Check the location of the object using the CRUSH algorithm:

    ```
    ceph osd map demotwo hosts
    ```
    
    Output:
    
    ```
    osdmap e45 pool 'demotwo' (11) object 'hosts' -> pg 11.ea1b298e (11.e) -> up ([0], p0) acting ([0], p0)
    ```
    
    You can see that the object is located on OSD 0. In real life, we would see three OSDs here, since the pool is replica 3.
    
12. Let's have a look at the CRUSH Map and what it looks like. Export it and decompile it:

    ```
    ceph osd getcrushmap -o crush_compiled.bin
    ```
    ```
    crushtool -d crush_compiled.bin -o deco_crush
    ```
    
13. Open the file and see the contents of the CRUSH. Note the replication rule looks as follows:

    ```
    $ vi deco_crush
    ```
    ```
    rule replicated_rule {
      id 0
      type replicated
      min_size 1
      max_size 10
      step take default
      step choose firstn 0 type osd
      step emit
    }
    ```
    
14. Compile the file again (usually done after making changes to the CRUSH Map manually):

    ```
    crushtool -c deco_crush -o compiled_crush_new
    ```
    
15. Inject the new file into the Ceph cluster:

    ```
    ceph osd setcrushmap -i compiled_crush_new
    ```
    
**16. Stop the lab here and inform Avi you've finished. The next sections will be shown by Avi.**

17. Check with OSDs are running on the host:

    ```
    df -h | grep ceph
    ```
    ``` 
    tmpfs    496M   24K  496M   1% /var/lib/ceph/osd/ceph-2
    tmpfs    496M   24K  496M   1% /var/lib/ceph/osd/ceph-4
    ```
    
18. Stop the primary OSD for the pg found in step 10, i.e. `osd.4`:

    ```
    ceph osd find 4
    ```
    
    SSH to the host where osd.4 exists
    
    ```
    systemctl stop ceph-osd@4
    ```
    
19. List the PGs on the OSD, and check for the PG found in 10 (`10.e`):

    ```
    ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-4 --op list-pgs | grep 10.e
    ```
    
20. Verify the object is located on the OSD:
 
    ```
    ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-4 --op list | grep hosts
    ```
    ```
    ["10.e",{"oid":"hosts","key":"","snapid":-2,"hash":3927648654,"max":0,"pool":10,"namespace":"","max":0}]
    ```
    
21. Start the stopped OSD:

    ```
    systemctl start ceph-osd@4
    ```
    
## 2.2 Erasure Code Pools

1. How to list all the erasure-code profiles?

    ```
    ceph osd erasure-code-profile get default
    ```
    ```
    k=2
    m=1
    plugin=jerasure
    technique=reed_sol_van  
    ```
    
2. How to create a new erasure profile?

    ```
    ceph osd erasure-code-profile set ec_rule k=4 m=2 crush-failure-domain=osd
    ```
    
3. How to inspect the new profile?

    ```
    ceph osd erasure-code-profile get ec_rule
    ```
    
4. How to create a new pool and set this profile on this pool?

    ```
    ceph osd pool create myec_pool 32 32 erasure ec_rule
    ```
    
5. Set application tagging on the pool:

    ```
    ceph osd pool application enable myec_pool rgw
    ```
    
6. Check the osd map now:

    ```
    ceph osd map myec_pool hosts
    ```
    ```
    osdmap e38 pool 'myec_pool' (10) object 'hosts' -> pg 10.ea1b298e (10.e) -> up ([0,NONE,NONE,NONE,NONE,NONE], p0) acting ([0,NONE,NONE,NONE,NONE,NONE], p0)
    ```
    
    You can see that we have 5 OSDs with NONE and one osd.0. This is because we have only one OSD in this set-up. In the real work, we will see the other OSDs - we'd see 6 because we set the rule to be 4 (data chunks) + 2 (parity chunks).
    
 8. Let's look again at the CRUSH Map:
 
    ```
    ceph osd getcrushmap -o crush_compiled.bin
    ```
    ```
    crushtool -d crush_compiled.bin -o deco_crush
    ```
    
 9. Open the file and see the contents of the CRUSH Map. Note the Erasure Code rule looks as follows:
 
    ```
    vi deco_crush
    ```
    ```
    rule myec_pool {
      id 1
      type erasure
      min_size 3
      max_size 6
      step set_chooseleaf_tries 5
      step set_choose_tries 100
      step take default
      step choose indep 0 type osd
      step emit
    }
    ```
    
10. Compile the file again (usually done after making changes to the CRUSH Map manually):

    ```
    crushtool -c deco_crush -o compiled_crush_new
    ```
    
11. Inject the new file into the Ceph cluster:

    ```
    ceph osd setcrushmap -i compiled_crush_new
    ```
    
12. Please take a look at the Ceph cluster state:

    ```
    ceph -s
    ```
    
    You see `HEALTH_WARN` because we don't have 6 OSDs for the EC profile.
    
13. Remove the EC pool. First, inject the `mon_allow_pool_delete` value to the MON daemon:

    ```
    ceph daemon mon.$HOSTNAME config set mon_allow_pool_delete true
    ```
    
14. Remove the pool from Ceph and verify the cluster status is `HEALTH_OK` afterwards:

    ```
    ceph osd pool delete myec_pool myec_pool --yes-i-really-really-mean-it
    ```
    
15. Set again the default value:

    ```
    ceph daemon mon.$HOSTNAME config set mon_allow_pool_delete false
    ```
    
**16. Stop the lab here and inform Avi you've finished. The next sections will be shown by Avi.**

17. Stop osd.2:

    ```
    ceph osd find 2
    ```
    
    SSH to host where osd.2 exists
    
    ```
    systemctl stop ceph-osd@2
    ```
    
18. Check the PGs on the OSD, and you will see the shards:

    ```
    ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-2 --op list-pgs | grep 11.e
    ```
    ```
    11.es2
    ```
    
    This is shard number 2.
    
19. Check the object is created on this OSD:

    ```
    ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-2 --op list | grep hosts
    ```
    ```
    ["11.es2",{"oid":"hosts","key":"","snapid":-2,"hash":3927648654,"max":0,"pool":11,"namespace":"","shard_id":2,"max":0}]
    ```
    
20. Start the OSD again:

    ```
    systemctl start ceph-osd@2
    ```
