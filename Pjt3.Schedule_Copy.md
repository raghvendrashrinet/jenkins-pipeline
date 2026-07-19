1. Continous pipe from one server to another 

```
 tail -f access_log error_log | ssh user@storage-server "cat >> /usr/src/itadmin/combined_live_log.txt"
```

## Schedule Copy 
Configure it to periodically build every 10 minutes to copy the Apache logs (both access_log and error_log) from App Server 1 (stapp01) from the default logs location to location /usr/src/itadmin on the Storage Server.
1. Add credential
2. add ssh host

3. Create a project
   
- Build periodically 
- every 10 minute
  ```
   Schedule
   H/10 * * * *  
  ```

   First go to physical server and save key
```
   in app-serv1
   ssh-keygen
   ssh-copy-id natasha@ststor01
```

5. In pre /Post build set below command
```
scp -o StrictHostKeyChecking=no /var/log/httpd/access_log /var/log/httpd/error_log natasha@ststor01:/usr/src/itadmin/
```
6. Run pipeline

#### The Better Alternative: rsync
Instead of scp, production environments almost exclusively use rsync.

You can modify your script to use rsync like this:
```
 rsync -avz -e "ssh -o StrictHostKeyChecking=no" /var/log/httpd/access_log /var/log/httpd/error_log natasha@ststor01:/usr/src/itadmin/
```
