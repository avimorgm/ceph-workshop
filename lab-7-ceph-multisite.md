# Lab #7: Ceph Multisite

1. For the multi site configuration, we will use the internal IP address of the container. Login to each of the containers and find the required IP address with:

    ```
    ./cn cluster enter master
    ip a | grep inet | egrep -v '127|inet6' | awk '{ print $2}'
    ```
    ```
    172.17.0.2
    ```
    ```
    ./cn cluster enter slave
    ip a | grep inet | egrep -v '127|inet6' | awk '{ print $2}'
    ```
    ```
    172.17.0.3
    ```
    
2. Create a new realm called gold and make it default, using this command:

    ```
    radosgw-admin realm create --rgw-realm=gold --default
    ```
    ```
    {
      "id": "d44ee8a4-1e73-4fbe-af39-7dcaf2f24a24",
      "name": "gold",
      "current_period": "08fc8930-3f32-47f9-8839-ce9033d29335",
      "epoch": 1
    }
    ```
    
3. Remove the default zonegroup, using this command:

    ```
    radosgw-admin zonegroup delete --rgw-zonegroup=default
    ```
    
4. Create a zonegroup, type this command:

    ```
    radosgw-admin zonegroup create --rgw-zonegroup=us --endpoints=http://172.17.0.2:8000 --master --default
    ```
    ```
    {
      "id": "a5d7a677-2fb5-4cb2-8ee9-16a64ada3622",
      "name": "us",
      "api_name": "us",
      "is_master": "true",
      "endpoints": [
        "http://172.17.0.2:8000"
      ],
      "hostnames": [],
      "hostnames_s3website": [],
      "master_zone": "",
      "zones": [],
      "placement_targets": [],
      "default_placement": "",
      "realm_id": "d44ee8a4-1e73-4fbe-af39-7dcaf2f24a24"
    }
    ```
    
5. Create a zone:

    ```
    radosgw-admin zone create --rgw-zonegroup=us --rgw-zone=us-east-1 --endpoints=http://172.17.0.2:8000 --access-key=test --secret=test --default --master
    ```
    ```
    {
      "id": "0ac2bc14-0a8f-4897-9954-a6edbf784011",
      "name": "us-east-1",
      "domain_root": "us-east-1.rgw.meta:root",
      "control_pool": "us-east-1.rgw.control",
      "gc_pool": "us-east-1.rgw.log:gc",
      "lc_pool": "us-east-1.rgw.log:lc",
      "log_pool": "us-east-1.rgw.log",
      "intent_log_pool": "us-east-1.rgw.log:intent",
      "usage_log_pool": "us-east-1.rgw.log:usage",
      "reshard_pool": "us-east-1.rgw.log:reshard",
      "user_keys_pool": "us-east-1.rgw.meta:users.keys",
      "user_email_pool": "us-east-1.rgw.meta:users.email",
      "user_swift_pool": "us-east-1.rgw.meta:users.swift",
      "user_uid_pool": "us-east-1.rgw.meta:users.uid",
      "system_key": {
        "access_key": "test",
        "Secret_key": "test"
      },
      "placement_pools": [
      {
        "key": "default-placement",
        "val": {
          "index_pool": "us-east-1.rgw.buckets.index",
          "data_pool": "us-east-1.rgw.buckets.data",
          "data_extra_pool": "us-east-1.rgw.buckets.non-ec",
          "index_type": 0,
          "compression": ""
        }
      }
      ],
      "metadata_heap": "",
        "tier_config": [],
      "realm_id": "d44ee8a4-1e73-4fbe-af39-7dcaf2f24a24"    
    }
    ```
    
6. Create a user that will be responsible for the replication between the clusters:

    ```
    radosgw-admin user create --uid=zone.user --display-name="Zone User" --access-key=test --secret=test --system
    ```
    ```
    {
      "user_id": "zone.user",
      "display_name": "Zone  User",
      "email": "",
      "suspended": 0,
      "max_buckets": 1000,
      "auid": 0,
      "subusers": [],
      "keys": [
      {
        "user": "zone.user",
        "access_key": "test",
        "secret_key": "test"
      }
      ],
      "swift_keys": [],
      "caps": [],
      "op_mask": "read, write, delete",
      "system": "true",
      "default_placement": "",
      "placement_tags": [],
      "bucket_quota": {
        "enabled": false,
        "check_on_raw": false,
        "max_size": -1,
        "max_size_kb": 0,
        "max_objects": -1
      },
      "user_quota": {
        "enabled": false,
        "check_on_raw": false,
        "max_size": -1,
        "max_size_kb": 0,
        "max_objects": -1
      },
      "temp_url_keys": [],
      "type": "rgw"
    }
    ```
    
7. Update the period:

    ```
    radosgw-admin period update --commit
    ```
    
8. Edit the `/etc/ceph/ceph.conf` file, using this command, and add the last line under the RGW section:

    ```
    vi /etc/ceph/ceph.conf
    ```
    ```
    [client.rgw.ceph-nano-master-faa32aebf00b]
    rgw dns name = ceph-nano-master-faa32aebf00b
    rgw enable usage log = true
    rgw usage log tick interval = 1
    rgw usage log flush threshold = 1
    rgw usage max shards = 32
    rgw usage max user shards = 1
    log file = /var/log/ceph/client.rgw.ceph-nano-master-faa32aebf00b.log
    rgw frontends = civetweb  port=0.0.0.0:8000
    rgw zone = us-east-1
    ```
    
9. Restart the Ceph container:

    ```
    docker restart <master-container-name>
    ```
    
10. Connect to the slave cluster:

    ```
    ./cn cluster enter slave
    ```
    
11. Pull the configuration from the master cluster, using the realm pull and the user that we created before:

    ```
    radosgw-admin realm pull --url=http://172.17.0.2:8000 --access-key=test --secret=test
    ```
    ```
    {
      "id": "d44ee8a4-1e73-4fbe-af39-7dcaf2f24a24",
      "name": "gold",
      "current_period": "3f0159da-8fc0-4cff-8275-82ccea94df40",
      "epoch": 2
    }  
    ```
    
12. Create a new zone, under US zonegroup:

    ```
    radosgw-admin zone create --rgw-zonegroup=us --rgw-zone=us-west --access-key=test --secret=test --endpoints=http://172.17.0.3:8001 --default
    ```
    ```
    {
      "id": "06275c5e-4cfb-4e00-a765-b903707edb1b",
      "name": "us-west",
      "domain_root": "us-west.rgw.meta:root",
      "control_pool": "us-west.rgw.control",
      "gc_pool": "us-west.rgw.log:gc",
      "lc_pool": "us-west.rgw.log:lc",
      "log_pool": "us-west.rgw.log",
      "intent_log_pool": "us-west.rgw.log:intent",
      "usage_log_pool": "us-west.rgw.log:usage",
      "reshard_pool": "us-west.rgw.log:reshard",
      "user_keys_pool": "us-west.rgw.meta:users.keys",
      "user_email_pool": "us-west.rgw.meta:users.email",
      "user_swift_pool": "us-west.rgw.meta:users.swift",
      "user_uid_pool": "us-west.rgw.meta:users.uid",
      "system_key": {
        "access_key": "test",
        "secret_key": "test"
      },
      "placement_pools": [
      {
        "key": "default-placement",
        "val": {
          "index_pool": "us-west.rgw.buckets.index",
          "data_pool": "us-west.rgw.buckets.data",
          "data_extra_pool": "us-west.rgw.buckets.non-ec",
          "index_type": 0,
          "compression": ""
        }
      }
      ],
      "metadata_heap": "",
      "tier_config": [],
      "realm_id": "d44ee8a4-1e73-4fbe-af39-7dcaf2f24a24"
    }
    ```
    
13. Update the period:

    ```
    radosgw-admin period update --commit
    ```
    
14. Edit the `/etc/ceph/ceph.conf` file, using this command, and add the last line under the RGW section:

    ```
    vi /etc/ceph/ceph.conf
    ```
    ```
    [client.rgw.ceph-nano-slave-faa32aebf00b]
    rgw dns name = ceph-nano-slave-faa32aebf00b
    rgw enable usage log = true
    rgw usage log tick interval = 1
    rgw usage log flush threshold = 1
    rgw usage max shards = 32
    rgw usage max user shards = 1
    log file = /var/log/ceph/client.rgw.ceph-nano-slave-faa32aebf00b.log
    rgw frontends = civetweb  port=0.0.0.0:8001
    rgw_zone=us-west
    ```
    
15. Restart the container:

    ```
    docker restart <slave-container-name>
    ```
    
16. Login to master and create a new user, type this command:

    ```
    ./cn cluster enter master
    radosgw-admin user create --uid=tester --access-key=tester --secret=tester --display-name=tester
    ```
    
17. Update the period:

    ```
    radosgw-admin period update --commit
    ```
    
18. Create a new bucket on the master, and check if the bucket exists on both clusters. Reconfigure `s3cmd`/`awscli` as you learned in a previous lab with the credentials of the new user (`access_key` = `tester`, `secret_key` = `tester`).
