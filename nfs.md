### CentOS
config (root@10.240.217.115)
```
[root@localhost ~]# vim /etc/exports
  1 /root/workspace/StorMgmtAgent/sdsagent 10.240.217.0/24(rw)
  2 /root/workspace/StorMgmtClientM/cephmgmtclient 10.240.217.0/24(rw)
  3 /root/workspace/StorMgmtServerM/storagemgmt 192.168.80.0/24(rw) 10.240.217.0/24(rw)
  4 /root/workspace/horizon/thinkcloud_portal/api 192.168.80.0/24(rw) 10.240.217.0/24(rw)
  5 /root/workspace/horizon/openstack_dashboard/api 192.168.80.0/24(rw) 10.240.217.0/24(rw)
  6 /root/workspace/ceilometer/ceilometer 10.20.10.0/24(rw) 10.20.0.0/24(rw)
  7 /root/workspace/nova/nova 10.20.10.0/24(rw) 10.20.0.0/24(rw) 192.168.80.0/24(rw)
  8 /root/workspace/keystonemiddleware 10.20.10.0/24(rw) 10.20.0.0/24(rw) 192.168.80.0/24(rw)
  9 /root/test 10.20.10.0/24(rw) 10.20.0.0/24(rw) 192.168.80.0/24(rw)
```
restart service
```
[root@localhost ~]# service nfs restart
```
check export directory
```
[root@localhost ~]# exportfs
/root/workspace/StorMgmtAgent/sdsagent
                10.240.217.0/24
/root/workspace/StorMgmtClientM/cephmgmtclient
                10.240.217.0/24
/root/workspace/StorMgmtServerM/storagemgmt
                192.168.80.0/24
/root/workspace/StorMgmtServerM/storagemgmt
                10.240.217.0/24
/root/workspace/horizon/thinkcloud_portal/api
                192.168.80.0/24
/root/workspace/horizon/thinkcloud_portal/api
                10.240.217.0/24
/root/workspace/horizon/openstack_dashboard/api
                192.168.80.0/24
/root/workspace/horizon/openstack_dashboard/api
                10.240.217.0/24
/root/workspace/ceilometer/ceilometer
                10.20.10.0/24
/root/workspace/ceilometer/ceilometer
                10.20.0.0/24
/root/workspace/nova/nova
                10.20.10.0/24
/root/workspace/nova/nova
                10.20.0.0/24
/root/workspace/nova/nova
                192.168.80.0/24
/root/workspace/keystonemiddleware
                10.20.10.0/24
/root/workspace/keystonemiddleware
                10.20.0.0/24
/root/workspace/keystonemiddleware
                192.168.80.0/24
/root/test      10.20.10.0/24
/root/test      10.20.0.0/24
/root/test      192.168.80.0/24
```
mount (root@10.240.217.111) (root@10.240.217.112)
```
[root@controller ~]# mount 10.240.217.115:/root/workspace/StorMgmtServerM/storagemgmt /usr/lib/python2.7/site-packages/storagemgmt
```
```
[root@node-1 ~]# mount 10.240.217.115:/root/workspace/StorMgmtAgent/sdsagent /usr/lib/python2.7/site-packages/sdsagent
```
