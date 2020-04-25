# Lab 6: CephFS

1. Confirm that the MDS role is enabled on your cluster:
    
    ```
    ceph -s | grep -i mds
    ```

2. Once MDS is installed, you can type the command:
    
    ```
    ceph df
    ```

    You will see these pools: `cephfs_data` and `cephfs_metadata`.

3. Create a user:
    
    ```
    ceph auth get-key client.admin | sudo tee /root/asecret
    ```

4. Create directory:
    
    ```
    mkdir /mnt/cephfs
    ```

5. Cat the file:
    
    ```
    cat /root/asecret
    
    AQDFpIle55cBABAAemiFLZKWYfBqu0YRV5kY+Q==
    ```

6. Mount the CephFs filesystem on this directory:
    
    ```
    sudo mount -t ceph 192.168.42.10:/ /mnt/cephfs -o name=admin,secretfile=/root/asecret
    ```

7. Check if the mount point is there:
    
    ```
    df -h | grep -i cephfs
    ```

8. Create 2 directories in the mount point, type the commands:

    ```
    mkdir -p /mnt/cephfs/dir1      
    mkdir -p /mnt/cephfs/dir2 
    ```

9. Create a simple file in the first directory:

    ```
    touch /mnt/cephfs/dir1/testfile
    ```

10. Create another file, with dd command:
    
    ```
    dd if=/dev/zero of=/mnt/cephfs/dir1/ddtest bs=1024 count=10000
    ```

11. Type the `df -h` command again. And you will notice that the file system contains some data.

12. How to enable snapshot on the filesystem? Type the command:

    ```
    ceph fs set cephfs allow_new_snaps true
    ```

13. Create snap directory, type the following command:

    ```
    mkdir -p /mnt/cephfs/dir1/.snap/mysnap
    ```

14. Check the mysnap directory, and you see snap of your files:

    ```
    ls /mnt/cephfs/dir1/.snap/mysnap
    ```

15. Try to remove a file, from dir1 directory:

    ```
    cd  /mnt/cephfs/dir1/
    rm -rf ddtest
    ```

16. Copy from the .snap directory to root dir, type the following command:
    
    ```
    cp /mnt/cephfs/dir1/.snap/mysnap/ddtest /root/
    ```

17. Change directory to root, ls the directory:

    ```
    ls -l /root/
    ```

18. You will see the file there.
