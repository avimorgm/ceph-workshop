# Lab 4: RADOS Gateway

## 4.1 s3cmd

1. Install `s3cmd` on the machine you are working on (and not on the Ceph Nano container).

    ```
    yum install -y s3cmd
    ```
    
2. Get your cluster `access key` and `secret key` by typing:

    ```
    ./cn cluster status master
    ```
    ```
    Endpoint: <rgw-endpoint>
    Dashboard: <dashboard-endpoint>
    Access key: <access-key>
    Secret key: <secret-key>
    Working directory: /tmp
    ```
    
3. Configure `s3cmd`:

    ```
    s3cmd --configure
    ```
    ```
    Enter new values or accept defaults in brackets with Enter.
    Refer to user manual for detailed description of all options.

    Access key and Secret key are your identifiers for Amazon S3. Leave them empty for using the env variables.
    Access Key: <access-key>
    Secret Key: <secret-key>
    Default Region [US]:

    Use "s3.amazonaws.com" for S3 Endpoint and not modify it to the target Amazon S3.
    S3 Endpoint [s3.amazonaws.com]: <rgw-endpoint-without-http://>

    Use "%(bucket)s.s3.amazonaws.com" to the target Amazon S3. "%(bucket)s" and "%(location)s" vars can be used
    if the target S3 system supports dns based buckets.
    DNS-style bucket+hostname:port template for accessing a bucket [%(bucket)s.s3.amazonaws.com]: [Enter]

    Encryption password is used to protect your files from reading
    by unauthorized persons while in transfer to S3
    Encryption password: <Enter>
    Path to GPG program [/usr/bin/gpg]: <Enter>

    When using secure HTTPS protocol all communication with Amazon S3
    servers is protected from 3rd party eavesdropping. This method is
    slower than plain HTTP, and can only be proxied with Python 2.7 or newer
    Use HTTPS protocol [Yes]: <No>

    On some networks all internet access must go through a HTTP proxy.
    Try setting it here if you can't connect to S3 directly
    HTTP Proxy server name: <Enter>

    New settings:
      Access Key: <access-key>
      Secret Key: <secret-key>
      Default Region: US
      S3 Endpoint: <rgw-endpoint-without-http://>
      DNS-style bucket+hostname:port template for accessing a bucket: %(bucket)s.s3.amazonaws.com
      Encryption password: <Enter>
      Path to GPG program: /usr/bin/gpg
      Use HTTPS protocol: <False>
      HTTP Proxy server name: <Enter>
      HTTP Proxy server port: 0

    Test access with supplied credentials? [Y/n] n

    Save settings? [y/N] y
    Configuration saved to '/root/.s3cfg'
    ```
    
5. Edit `/root/.s3cfg`, if not already set then change the lines where `host_base` and `host_bucket` are, as follows:

    ```
    host_base = <rgw-endpoint-without-http://>
    ```
    ```
    host_bucket = <rgw-endpoint-without-http://>
    ```
    
6. Let’s create bucket, type the command:

     ```
     s3cmd mb s3://my-bucket
     ```
     
7. How to list all the buckets:

    ```
    s3cmd ls
    ```
    
8. Create a file to upload to the cluster:

    ```
    touch file
    ```
    
9. How to upload an object to the cluster:

    ```
    s3cmd put file s3://my-bucket/
    ```
    
10. How to list all the object in a particular bucket:
    
    ```
    s3cmd ls -r s3://my-bucket
    ```
    
11. Enter the Ceph cluster and note that 2 new pools have been created: `default.rgw.buckets.data`, which is the pool where the RGW data (Object) is stored as RADOS Objects, and `default.rgw.buckets.index`, which is the pool where the RGW metadata is stored:
    
    ```
    ./cn clutser enter master
    ```
    ```
    rados lspools
    ```
    
    By default, these pools are created as replication pools.
    
12. From inside the Ceph cluster, you can list all the RADOS Objects in the RGW data pool:

    ```
    rados ls -p default.rgw.buckets.data
    ```
    ```
    e1a6d73b-8958-494e-94a7-0583318de7b0.24102.1_file
    ```
    
    The output should be `bucket id` (`e1a6d73b-8958-494e-94a7-0583318de7b0.24102.1`) + `object` (`file`) separated by a `_`. In multipart uploads, Ceph will add the `part id` + the sequence number of the RADOS object + `multipart id`.
    
13. Let's create multipart upload. On your machine, generate a file with `dd`, using this command:
    
    ```
    dd if=/dev/zero of=/var/zeros bs=100M count=1
    ```
    
14. Put the file into the bucket:

    ```
    s3cmd put /var/zeros s3://my-bucket
    ```
    
    Keep in mind that if the upload gets stuck then the RGW does not delete the multipart RADOS objects that have already been created, and it keeps them until the multipart upload is completed or aborted.
    
15. List all the in-progress multipart uploads, if there are any:

    ```
    s3cmd multipart s3://my-bucket
    ```
    
16. Check for the new pool that should be created - `default.rgw.buckets.non-ec`:

    ```
    ./cn cluster enter master
    ```
    ```
    rados lspools
    ```
    
    This pool contains data on in-flight multipart uploads when there is no any multipart running now, this pool should be empty.
    
17. Let's check the index pool, `default.rgw.buckets.index`. This pool is responsible for the bucket index. Basically all the metadata of the bucket object is saved on this pool.

    ```
    rados ls -p default.rgw.buckets.index
    ```
    ```
    .dir.e1a6d73b-8958-494e-94a7-0583318de7b0.24102.1
    ```
    
    Every object begins with `.dir.` + `<bucket.id>`. This OMAP object says which objects exist on the bucket with this `bucket id`.
    
18. How can we see which objects in that bucket from looking at the index pool?

    ```
    rados listomapkeys -p default.rgw.buckets.index .dir.<bucket id>
    ```
    
19. In case there is a big OMAP (100,000 and above object per shard) we will need to reshard it. Large OMAPs affect performance:

    ```
    radosgw-admin bucket reshard --num-shards 2 --bucket=my-bucket
    ```
    
20. List the keys again over the pool:

    ```
    rados ls -p default.rgw.buckets.index
    ```
    
    Please note that now our OMAP has been cut into two pieces.
    
## 4.2 awscli

1. Download the `awscli` tool into your machine (and not on the Ceph Nano Container):

    ```
    yum install -y python3-pip
    ```
    ```
    pip3 install awscli
    ```
    
2. Get your cluster `access key` and `secret key` by typing:

    ```
    ./cn cluster status master
    ```
    ```
    Endpoint: <rgw-endpoint>
    Dashboard: <dashboard-endpoint>
    Access key: <access-key>
    Secret key: <secret-key>
    Working directory: /tmp
    ```
    
3. Configure it:

    ```
    aws configure
    ```
    ```
    AWS Access Key ID [None]: <access-key>
    AWS Secret Access Key [None]: <secret-key>
    Default region name [None]: <Enter>
    Default output format [None]: <Enter>
    ```
    
4. List all your buckets:

    ```
    aws s3 ls --endpoint-url <rgw-endpoint>
    ```
    
5. How to create a bucket using the `awscli` tool:

    ```
    aws s3 mb s3://your_bucket --endpoint-url <rgw-endpoint>
    ```
    
6. How to upload an object to S3 with public read-only permission (so that others can download it without needing credentials):

    ```
    aws s3 cp /etc/hosts s3://your_bucket/your_file --endpoint-url <rgw-endpoint> --acl public-read
    ```
    
7. How to get object from S3:

    ```
    aws s3 cp s3://your_bucket/your_file /etc/your_new_file --endpoint-url <rgw-endpoint>
    ```
    
8. How to get part of the object:

    ```
    aws s3api get-object --bucket your_bucket --key your_file --range bytes=0-2  --endpoint-url <rgw-endpoint> /tmp/your_new_file
    ```
