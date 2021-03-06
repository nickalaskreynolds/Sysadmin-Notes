Typically on clustered webservers I will utilize two types of cluster storage. One for fast-replication (like microsoft's DFS) and one for slow replication (based on a unix crontab). The reasoning behind this is the fast-replication is a 'dumb product'. What I mean by that is that it does one thing, file replication, that's it. The slow replication product I use is more intelligent in that you can specify filters, exceptions, and actions to perform based on patterns of replaced files.

In my standard /var/www/external and /var/www/internal file structure, the Fast Replication is used for the 'external' folder as that will contain the "web code" for the websites. Changes made to the files on any webserver are immediately replicated to all other webservers.

The slow replication is used on the 'internal' folder. It will (on a schedule) replicate changes in that folder that don't change often, but when they DO change, something else typically needs to follow (such as cycling a service). A real-world example is that I'll have a directory for nginx configuration files. Making changes to those files will typically require a reload of the nginx service. I can specify 'actions' in this slow-sync product that when a file is changed and the location/name matches a pattern, to run a script or command afterwards.

If I were to make a change (or add a new) vhost file for nginx, within 5 minutes (the interval I usually schedule the slow-sync job), the new vhost file is copied to all other webservers and nginx automatically reloaded on all webservers. I don't have to manually reload every nginx service.

For the two clustered storage products, we'll assume we have two webservers

 - web1 is 192.168.1.1
 - web2 is 192.168.1.2

A side note, for the glusterfs storage, best practice is that you have a pool of servers acting as your independant storage pool and your webservers just attach to it as clients. This works fine but my experience with this is that disk I/O is slow in terms of trying to run a website from it (the average homepage request for a fresh Joomla3 install took ~1000ms). When running the same website from local storage it was down to ~50ms which, although still very slow in my opinion, is good for Joomla3.

I'll typically dual-purpose the webservers to also be members (and connect to themselves as clients) of the clustered storage. That way each webserver will have a 'local copy' of all the data that you can run the websites from. That way, disk I/O is back up to speed. I would also recommend to have at least 2 additional servers to serve as storage-only nodes so in the event you lose your webservers, you don't also lose your storage cluster.