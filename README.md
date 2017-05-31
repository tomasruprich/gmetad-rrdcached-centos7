# gmetad-rrdcached-centos7
Solving heavy I/O on gmetad server by adding rrdcached on CentOS7.

Rrdcached is part of rrdtool:
> # rpm -qa rrdtool
> rrdtool -rrdtool-1.4.8-9.el7.x86_64
> # rrdtool -v
> RRDtool 1.4.8  Copyright 1997-2013 by Tobias Oetiker <tobi@oetiker.ch>
>               Compiled Nov 20 2015 19:23:48

## Running rrdcached
Step by step from:
https://github.com/ganglia/monitor-core/wiki/Integrating-Ganglia-with-rrdcached

BUT! :-)

Apache on CentOS 7 comes with sneaky line in systemd configfile:
> PrivateTmp=true
which prevents using /tmp/ for rrdcached sockets. 

So let's move it, but we'd need suitable directory - or we can use another systemd directives and maintain our own under /run:
> RuntimeDirectory=rrdcached
> RuntimeDirectoryMode=0755

Cool. Now we need to tell gmetad to use rrdcache. Gmetad on CentOS doesn't use /etc/sysconfig/gmetad by default, so we need to enhance systemd unitfile to do so: 
> mkdir /etc/systemd/system/gmetad.service.d/
> cat > /etc/systemd/system/gmetad.service.d/gmetad-envfile.conf <<_EOF
> [Service]
> EnvironmentFile=/etc/sysconfig/gmetad
> _EOF
> systemctl daemon-reload
> systemctl restart gmetad

Last thing, we just add 
> $conf['rrdcached_socket'] = "unix:/var/run/rrdcached/rrdcached.limited.sock";
to Ganglia web conf.php. 
