These instructions should be performed on ALL webservers. It's important you keep each webserver identical to one another so the runtime environments are the same no matter which server a user is load balanced on. If you want, you can do the install and clone the webserver to as many as you want. Be sure to do this BEFORE you start the Cluster Storage sections.

I prefer nginx all day over any other webserver. After installation typically I'll create the following directory structure:

Slow-sync'd directories here. This is covered under Cluster Storage. Only used for shared config files and such
```
mkdir -p /var/www/internal/conf/php-fpm/pools/
```

The 'internal' folder is used for config files and other 'portable' items that don't need realtime syncing, rather a slower sync schedule is fine. The reasoning for this is covered under Cluster Storage.
