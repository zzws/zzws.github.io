华数的keepalived 一个节点上4个实例，任意一个实例切换，其他三个实例会一同切换。

cat keepalived.sh 
```
#!/bin/bash
/usr/local/keepalived/sbin/keepalived -f /usr/local/keepalived/etc/keepalived/keepalived.conf 
```

 cat  /usr/local/keepalived/etc/keepalived/keepalived.conf 
```
[root@localhost ~]# cat  /usr/local/keepalived/etc/keepalived/keepalived.conf 
vrrp_script chk_mysql_3305 {
    script "/etc/checkMySQL/checkMySQL3305.sh"
    interval 30
}
vrrp_instance VI_5 {
    state BACKUP
    interface eth0
    virtual_router_id 150
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    track_script {
        chk_mysql_3305
   }
}

vrrp_script chk_mysql_3306 {
    script "/etc/checkMySQL/checkMySQL3306.sh"
    interval 30
}
vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 151
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    track_script {
        chk_mysql_3306
   }
    virtual_ipaddress {
        10.255.54.118/24  dev eth0 label eth0:2
    }
}

vrrp_script chk_mysql_3307 {
    script "/etc/checkMySQL/checkMySQL3307.sh"
    interval 30
}
vrrp_instance VI_2 {
    state BACKUP
    interface eth0
    virtual_router_id 152
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    track_script {
        chk_mysql_3307
   }
    virtual_ipaddress {
        10.255.54.119/24  dev eth0 label eth0:3
    }
}
vrrp_script chk_mysql_3308 {
    script "/etc/checkMySQL/checkMySQL3308.sh"
    interval 30
}
vrrp_instance VI_3 {
    state BACKUP
    interface eth0
    virtual_router_id 153
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    track_script {
        chk_mysql_3308
   }
    virtual_ipaddress {
        10.255.54.120/24  dev eth0 label eth0:4
    }
}
vrrp_script chk_mysql_3309 {
    script "/etc/checkMySQL/checkMySQL3309.sh"
    interval 30
}
vrrp_instance VI_4 {
    state BACKUP
    interface eth0
    virtual_router_id 154
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    track_script {
        chk_mysql_3309
   }
    virtual_ipaddress {
        10.255.54.121/24  dev eth0 label eth0:5
    }
}
```

cat /etc/checkMySQL/checkMySQL3306.sh
```
[root@localhost ~]# cat /etc/checkMySQL/checkMySQL3306.sh
#!/bin/bash
MYSQL=/usr/local/mysql/bin/mysql
MYSQL_HOST=127.0.0.1
MYSQL_USER=root
MYSQL_PASSWORD=123456
MYSQL_PORT=3306
CHECK_TIME=3

#mysql  is working MYSQL_OK is 1 , mysql down MYSQL_OK is 0

MYSQL_OK=1

function check_mysql_helth (){
    $MYSQL -h $MYSQL_HOST -u $MYSQL_USER -p${MYSQL_PASSWORD} -P $MYSQL_PORT -e "show status;" >/dev/null 2>&1
    if [ $? = 0 ] ;then
    MYSQL_OK=1
    else
    MYSQL_OK=0
    fi
    return $MYSQL_OK
}
while [ $CHECK_TIME -ne 0 ]
do
    let "CHECK_TIME -= 1"
    check_mysql_helth
if [ $MYSQL_OK = 1 ] ; then
    CHECK_TIME=0
    exit 0
fi
if [ $MYSQL_OK -eq 0 ] &&  [ $CHECK_TIME -eq 0 ]
then
    bash /root/keepalivedstop.sh
    exit 1
fi
sleep 1
done
```
cat /root/keepalivedstop.sh

```
[root@localhost ~]# cat /root/keepalivedstop.sh
#!/bin/bash
/usr/bin/pkill keepalived 
```