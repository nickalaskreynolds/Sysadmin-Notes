#Installation#
See Installing the Epel/Remi repositories

Install dependencies
```
yum install cpp gcc-c++ cmake git psmisc {binutils,boost,jemalloc}-devel \
{sqlite,tbb,bzip2,openldap,readline,elfutils-libelf,gmp,lz4,pcre}-devel \
lib{xslt,event,yaml,vpx,png,zip,icu,mcrypt,memcached,cap,dwarf}-devel \
{unixODBC,expat,mariadb}-devel lib{edit,curl,xml2,xslt}-devel \
glog-devel oniguruma-devel inotify-tools-devel ocaml openssl-devel gperf
```

uninstall and install specific ImageMagick binaries
```
yum remove ImageMagick
yum install ImageMagick-last\* --enablerepo=remi
```

Now we need to pull down the latest hhvm code. I prefer to do it this way because when I download the project as a zip file, it is usually missing some critical files for whatever reason. This will take a minute or two
```
cd ~
git clone https://github.com/facebook/hhvm -b master  hhvm  --recursive
cd hhvm/
```

Now we need to configure hhvm
```
cmake \
-DLIBMAGICKWAND_INCLUDE_DIRS="/usr/include/ImageMagick-6" \
-DLIBMAGICKCORE_LIBRARIES="/usr/lib64/libMagickCore-6.Q16.so" \
-DLIBMAGICKWAND_LIBRARIES="/usr/lib64/libMagickWand-6.Q16.so" .
```

Now we compile, this will take a while (averaged 60 minutes for me on my lab hardware). If this command gives you problems, just run 'make' without the rest of the command
```
make -j$(($(nproc)+1))
```

test the compiled product
```
./hphp/hhvm/hhvm --version
```

install it
```
make install
```

#Create Service File#
```
vim /usr/lib/systemd/system/hhvm.service
```
> ####Paste the contents of configs/hhvm.service####

reload the systemd daemon and enable service
```
systemctl daemon-reload
systemctl enable hhvm.service
mkdir -p /etc/hhvm/
mkdir -p /var/log/hhvm/
touch /var/log/hhvm/hhvm.log
touch /etc/hhvm/server.ini
```

#Configuration#
```
vim /etc/hhvm/server.ini
```
> ####Paste the contents of configs/server.ini####

Now restart hhvm
```
systemctl restart hhvm
systemctl status hhvm
```

#Firewall#
First we need to determine which zones are active
```
firewall-cmd --get-active-zones
```
In my dev environment, I only have one zone called `internal` active. Now we open the web ports for the target zone
```
firewall-cmd --zone=internal --add-port=80/tcp --permanent
firewall-cmd --zone=internal --add-port=443/tcp --permanent
```