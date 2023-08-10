

도메인생성

https://carpet-part1.tistory.com/676

bastion은 centos 8 stream으로 설치 한다. 

https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.8/latest/

<br/>

bastion servers는

```bash
[root@bastion named]# vi /etc/resolv.conf
# Generated by NetworkManager
search okd4.shclub.duckdns.org
nameserver 192.168.1.69
```

<br/>

haproxy 설정

<br/>

```bash
[root@bastion shclub]# cat /etc/haproxy/haproxy.cfg
# Global settings
#---------------------------------------------------------------------
global
    maxconn     20000
    log         /dev/log local0 info
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    log                     global
    mode                    http
    option                  httplog
    option                  dontlognull
    option http-server-close
    option redispatch
    option forwardfor       except 127.0.0.0/8
    retries                 3
    maxconn                 20000
    timeout http-request    10000ms
    timeout http-keep-alive 10000ms
    timeout check           10000ms
    timeout connect         40000ms
    timeout client          300000ms
    timeout server          300000ms
    timeout queue           50000ms

# Enable HAProxy stats
listen stats
    bind :9000
    mode http
    stats enable
    stats uri /
    monitor-uri /bastion-test
#    stats uri /stats
    stats refresh 5s

#---------------------------------------------------------------------
# static backend for serving up images, stylesheets and such
#---------------------------------------------------------------------
backend static
    balance     roundrobin
    server      static 127.0.0.1:4331 check

# Openshift  API Server
frontend openshift_api_frontend
    bind *:6443
    default_backend openshift_api_backend
    mode tcp
    option tcplog

backend openshift_api_backend
    mode tcp
    balance source
    server      bootstrap 192.168.1.127:6443 check
    server      okd-1 192.168.1.145:6443 check

# OCP Machine Config Server
frontend ocp_machine_config_server_frontend
    mode tcp
    bind *:22623
    default_backend ocp_machine_config_server_backend

backend ocp_machine_config_server_backend
    mode tcp
    balance source
    server      bootstrap 192.168.1.127:22623 check
    server      okd-1 192.168.1.145:22623 check

# OCP Ingress - layer 4 tcp mode for each. Ingress Controller will handle layer 7.
frontend ocp_http_ingress_frontend
    bind *:80
    default_backend ocp_http_ingress_backend
    mode tcp

backend ocp_http_ingress_backend
    balance source
    mode tcp
    server      okd-1 192.168.1.145:80 check

frontend ocp_https_ingress_frontend
    bind *:443
    default_backend ocp_https_ingress_backend
    mode tcp

backend ocp_https_ingress_backend
    mode tcp
    balance source
    server      okd-1 192.168.1.145:443 check
```


<br/>

/etc/named.conf

<br/>

```bash
[root@bastion shclub]# cat /etc/named.conf
options {
	listen-on port 53 { any; };
	listen-on-v6 port 53 { none; };
	directory 	"/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";
	secroots-file	"/var/named/data/named.secroots";
	recursing-file	"/var/named/data/named.recursing";
	allow-query     { any; };

	/*
	 - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
	 - If you are building a RECURSIVE (caching) DNS server, you need to enable
	   recursion.
	 - If your recursive DNS server has a public IP address, you MUST enable access
	   control to limit queries to your legitimate users. Failing to do so will
	   cause your server to become part of large scale DNS amplification
	   attacks. Implementing BCP38 within your network would greatly
	   reduce such attack surface
	*/
	recursion yes;

	dnssec-enable yes;
	dnssec-validation yes;

	managed-keys-directory "/var/named/dynamic";

	pid-file "/run/named/named.pid";
	session-keyfile "/run/named/session.key";

	/* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
	include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
	type hint;
	file "named.ca";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

<br/>


rfc1912 zone 파일 수정

<br/>

```bash
[root@bastion shclub]# cat /etc/named.rfc1912.zones
zone "localhost.localdomain" IN {
	type master;
	file "named.localhost";
	allow-update { none; };
};

zone "localhost" IN {
	type master;
	file "named.localhost";
	allow-update { none; };
};

zone "1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.ip6.arpa" IN {
	type master;
	file "named.loopback";
	allow-update { none; };
};

zone "1.0.0.127.in-addr.arpa" IN {
	type master;
	file "named.loopback";
	allow-update { none; };
};

zone "0.in-addr.arpa" IN {
	type master;
	file "named.empty";
	allow-update { none; };
};

zone "jakelee.com" IN {
        type master;
        file "/var/named/okd4.jakelee.com.zone";
        allow-update { none; };
};

zone "1.168.192.arpa" IN {
        type master;
        file "/var/named/1.168.192.in-addr.rev";
        allow-update { none; };
};
```

<br/>

/var/named 폴더로 이동한다.

<br/>

Zone 파일 설정 (DNS 정방향)

<br/>

```bash
[root@bastion shclub]# cd /var/named
[root@bastion named]# ls
1.168.192.in-addr.rev  data  dynamic  named.ca  named.empty  named.localhost  named.loopback  okd4.jakelee.com.zone  slaves
[root@bastion named]# cat okd4.jakelee.com.zone
$TTL 1D
@ IN SOA @ ns.jakelee.com. (
				0	; serial
				1D	; refresh
				1H	; retry
				1W	; expire
				3H )	; minimum
@ IN NS ns.jakelee.com.
@ IN A  192.168.1.154	;

; Ancillary services
lb.okd4 	IN	A       192.168.1.154

; Bastion or Jumphost
ns	IN	A	192.168.1.154	;

; OCP Cluster
bastion.okd4    IN      A       192.168.1.154
bootstrap.okd4	IN	A	192.168.1.127

okd-1.okd4	IN	A	192.168.1.145


api.okd4	IN	A	192.168.1.154
api-int.okd4	IN	A	192.168.1.154
*.apps.okd4	IN	A	192.168.1.154
```


<br/>

Zone 파일 설정 (DNS 역방향)

<br/>

```bash
[root@bastion named]# cat 1.168.192.in-addr.rev
$TTL 1D
@	IN	SOA	jakelee.com. ns.jakelee.com. (
						0	; serial
						1D	; refresh
						1H	; retry
						1W	; expire
						3H )	; minimum

@	IN	NS	ns.
154	IN	PTR	ns.
154	IN	PTR	bastion.okd4.jakelee.com.
127	IN	PTR	bootstrap.okd4.jakelee.com.
145	IN	PTR	okd-1.okd4.jakelee.com.

154	IN	PTR	api.okd4.jakelee.com.
154	IN	PTR	api-int.okd4.jakelee.com.
```

<br/>

zone 파일 권한 설정

<br/>

```bash
[root@bastion named]# chown root:named okd4.jakelee.com.zone
[root@bastion named]# chown root:named 1.168.192.in-addr.rev
[root@bastion named]# systemctl restart named
```  

<br/>

<br/>

배포준비 

1) SSK KEY 생성
bastion 루트로 이동한다.

<br/>

```bash
[root@bastion named]# cd ~/
[root@bastion ~]# ssh-keygen -t rsa -b 4096 -N ''
[root@bastion ~]# ls -l .ssh
total 12
-rw------- 1 root root 3401 Aug  6 05:26 id_rsa
-rw-r--r-- 1 root root  755 Aug  6 05:26 id_rsa.pub
```  

<br/>

2) oc 바이너리와 openshift-install 바이너리 다운로드

<br/>

```bash
[root@bastion ~]# wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.8.14/openshift-install-linux.tar.gz
--2023-08-07 02:41:34--  https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.8.14/openshift-install-linux.tar.gz
Resolving mirror.openshift.com (mirror.openshift.com)... 99.84.238.131, 99.84.238.92, 99.84.238.173, ...
Connecting to mirror.openshift.com (mirror.openshift.com)|99.84.238.131|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 92856643 (89M) [application/x-tar]
Saving to: 'openshift-install-linux.tar.gz.1'

openshift-install-linux.tar.gz. 100%[====================================================>]  88.55M  22.4MB/s    in 4.7s

2023-08-07 02:41:39 (18.8 MB/s) - 'openshift-install-linux.tar.gz.1' saved [92856643/92856643]

[root@bastion ~]# wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.8.14/openshift-client-linux.tar.gz
--2023-08-07 02:41:41--  https://mirror.openshift.com/pub/openshift-v4/clients/ocp/4.8.14/openshift-client-linux.tar.gz
Resolving mirror.openshift.com (mirror.openshift.com)... 99.84.238.131, 99.84.238.139, 99.84.238.173, ...
Connecting to mirror.openshift.com (mirror.openshift.com)|99.84.238.131|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 49725881 (47M) [application/x-tar]
Saving to: 'openshift-client-linux.tar.gz.1'

openshift-client-linux.tar.gz.1 100%[====================================================>]  47.42M  15.2MB/s    in 3.1s

2023-08-07 02:41:45 (15.2 MB/s) - 'openshift-client-linux.tar.gz.1' saved [49725881/49725881]

[root@bastion ~]# ls
anaconda-ks.cfg       ocp                            openshift-client-linux.tar.gz.1  openshift-install-linux.tar.gz.1
initial-setup-ks.cfg  openshift-client-linux.tar.gz  openshift-install-linux.tar.gz
[root@bastion ~]# rm -rf *.1
[root@bastion ~]# ls
anaconda-ks.cfg  initial-setup-ks.cfg  ocp  openshift-client-linux.tar.gz  openshift-install-linux.tar.gz
[root@bastion ~]# tar xvf openshift-client-linux.tar.gz -C /usr/local/bin/

[root@bastion ~]# tar xvf openshift-install-linux.tar.gz -C /usr/local/bin/
```

<br/>

3)  install-config.yaml 파일 생성 및 백업

<br/>
```bash
[root@bastion ~]# mkdir -p /root/ocp/config
[root@bastion ~]# cd /root/ocp/config
[root@bastion config]# vi install-config.yaml
apiVersion: v1
baseDomain: jakelee.com
compute:
- hyperthreading: Enabled
  name: worker
  replicas: 0
controlPlane:
  hyperthreading: Enabled
  name: master
  replicas: 1
metadata:
  name: okd4  #cluster name
networking:
  clusterNetworks:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  networkType: OpenShiftSDN
  serviceNetwork:
  - 172.30.0.0/16
platform:
  none: {}
fips: false
pullSecret: '{"auths":{"cloud.openshift.com":{"auth":"b3BlbnNoaWZ0LXJlbGVhc2UtZGV2K29jbV9hY2Nlc3NfODA3Yjc0MDgzODBmNDg4NmExYTE4YWVjMzZjZDc3ZTE6WEhHTVhYWlAzMjEyR0tJUFRaN0Y3MUNSWVRHUEVMM1BBRThQUExWSlEzSDdNSTFGRzFPN1hBODRSQjZONTFYSw==","email":"shclub@gmail.com"},"quay.io":{"auth":"b3BlbnNoaWZ0LXJlbGVhc2UtZGV2K29jbV9hY2Nlc3NfODA3Yjc0MDgzODBmNDg4NmExYTE4YWVjMzZjZDc3ZTE6WEhHTVhYWlAzMjEyR0tJUFRaN0Y3MUNSWVRHUEVMM1BBRThQUExWSlEzSDdNSTFGRzFPN1hBODRSQjZONTFYSw==","email":"shclub@gmail.com"},"registry.connect.redhat.com":{"auth":"fHVoYy1wb29sLTYxMTBkMjQyLTQ3MjgtNDBhYS05Zjc5LTdjZTMyNDUyNzJlYzpleUpoYkdjaU9pSlNVelV4TWlKOS5leUp6ZFdJaU9pSXdaamMyTkRRMU4yVXdNREUwT0dJek9EZGpNVGMyTW1GaE9ERTBORGcwTVNKOS5aNXZrTnNTb3NlQ1NfTDZOQ1NiN0I5NTVkUkR4NmVsUWpkZGMwRl9UcW4tbDhVNnh1YTRRVXJQTTd5eEZXY09jeGJjZ0J3QUdoWEhHVGtaNUJKbUxKMVRkUjAwRU8yV00tTGQySVYtU0w3ajIwLWNSS3Z2Q1JpMFVfZ1EwWDdrdDJacWEyOXZOTU1oQ0dHRlZldGl0UUNoSEFxQ3ZPQzhYejRLa2hoOFNQdi1SRWVjeW9OdWQwMVZRaHEtOUpiekxQSFNpTkZ2UGRCN2Njc1JaQmpXSFdqUUNPNzRaTURVdk5qZjlpQ2xlRno5endGVk1UM3p2S0M4M19GN1ZwU3hDamkzeEdGUmp2OVktRDhXUmtuamVkeHo0TXZNRnFhTUpYdFh4LUtpUXpfNFhjZ2ZnMWVnMlFqSEpyeUQzNkRPNG9lZ1V4X0tidnZvbUFJdFM5STJHM3F0Tm04WXdPSU9oVENuMFVPZWh5T1U5NE0xbzBiVHd2R2JFM0RCb1ZHdEZtdC1wdmh4SGN3cEtSYUo2akFKd1hxT1lZNmw4LTMwLUtFY3F2aDNTZnQzQnNuRENrYlVDb2hqRnJGdm9UNmZyc1MxNkJMbzNoLTNkckZ3V1FfcG51QkJ5NjJfWFVsNW1SbjVUY0thdHNnX216NWNiNDZnSmwtTG5pZ2V0QzdFbzVyQ2NkaEdWNDByUDZseHBHV1JoeUZzbnQ4ZThMbldUVDduVlRrRHNIMGRRWXVkRmZoNzRsTE1Da3JnS0xUMEJqYks5Y0FoR0JfRjBZMjZEa3lCOHF2SkdRSGE2VklOQ1Y3dnpRTU1GU3lHeWdZQ2VkWjFSWk9PRUQwSlRiX0hrUVVoUHE5dEthOVZDOWtsS2tCNVViSEF3OXByNTdnR25QSzFRNzJycDI4NA==","email":"shclub@gmail.com"},"registry.redhat.io":{"auth":"fHVoYy1wb29sLTYxMTBkMjQyLTQ3MjgtNDBhYS05Zjc5LTdjZTMyNDUyNzJlYzpleUpoYkdjaU9pSlNVelV4TWlKOS5leUp6ZFdJaU9pSXdaamMyTkRRMU4yVXdNREUwT0dJek9EZGpNVGMyTW1GaE9ERTBORGcwTVNKOS5aNXZrTnNTb3NlQ1NfTDZOQ1NiN0I5NTVkUkR4NmVsUWpkZGMwRl9UcW4tbDhVNnh1YTRRVXJQTTd5eEZXY09jeGJjZ0J3QUdoWEhHVGtaNUJKbUxKMVRkUjAwRU8yV00tTGQySVYtU0w3ajIwLWNSS3Z2Q1JpMFVfZ1EwWDdrdDJacWEyOXZOTU1oQ0dHRlZldGl0UUNoSEFxQ3ZPQzhYejRLa2hoOFNQdi1SRWVjeW9OdWQwMVZRaHEtOUpiekxQSFNpTkZ2UGRCN2Njc1JaQmpXSFdqUUNPNzRaTURVdk5qZjlpQ2xlRno5endGVk1UM3p2S0M4M19GN1ZwU3hDamkzeEdGUmp2OVktRDhXUmtuamVkeHo0TXZNRnFhTUpYdFh4LUtpUXpfNFhjZ2ZnMWVnMlFqSEpyeUQzNkRPNG9lZ1V4X0tidnZvbUFJdFM5STJHM3F0Tm04WXdPSU9oVENuMFVPZWh5T1U5NE0xbzBiVHd2R2JFM0RCb1ZHdEZtdC1wdmh4SGN3cEtSYUo2akFKd1hxT1lZNmw4LTMwLUtFY3F2aDNTZnQzQnNuRENrYlVDb2hqRnJGdm9UNmZyc1MxNkJMbzNoLTNkckZ3V1FfcG51QkJ5NjJfWFVsNW1SbjVUY0thdHNnX216NWNiNDZnSmwtTG5pZ2V0QzdFbzVyQ2NkaEdWNDByUDZseHBHV1JoeUZzbnQ4ZThMbldUVDduVlRrRHNIMGRRWXVkRmZoNzRsTE1Da3JnS0xUMEJqYks5Y0FoR0JfRjBZMjZEa3lCOHF2SkdRSGE2VklOQ1Y3dnpRTU1GU3lHeWdZQ2VkWjFSWk9PRUQwSlRiX0hrUVVoUHE5dEthOVZDOWtsS2tCNVViSEF3OXByNTdnR25QSzFRNzJycDI4NA==","email":"shclub@gmail.com"}}}
'
sshKey: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDI65c0zGm0FHvGvmoajOHset/grWS3gnOMZuYEa641mUMAfWBBIWn5cHymP49cHhcc4SIgJa6p5F7sCmmTP3aq+Ig4rMZ44zy6Aq4Wj1Q3f6/3+rpBkzzHVPjNwSdFLcBVbrL8le6EgZP0ZVoNs4rbXgI40fe3xytAnav30FHLbaFKENPFpQI3hRsSQCzIzCsBQdkyOEpVF6W5vKSmjxcGNqSkv4nSyKGM5i9KZX003OheoDd5pLvGaBSwyWa74ny9f/T7xKiUbnBd0bDAnZZIJpr6RYzMzfqQ4EAjjl5Jxq7oRzsaQZssk0VBsAUAgqrNHAFz5FRVPVUoiOmGWIag72KDHMcx9qrzvhOJwg8BNiVh0BPW7otBdR+ti+qfiQpVsBYrnWIwyxxIaFHaSm6Oxhf1HtR7SfWVtybuLvdzbZyU6l+oN2ZA4Ypsea2dSkaK1jvuCkgeIs7in3TGvGBJyWCmrRyZ2LC94cr/bpnp1GH+PgPKHZC6waJ51YKiAvVlwHbBTSNjIG7OIbIEAmr359i52MZBUqJ6V4lymA+wWAHXD0o2RWM4a87nCTz3EhfeXH+MSOnwR9DcLMP3QatbaXJb0/vpftEGZseSBOUoKTnnz2ArJ2heTbbFppfyfWN4pY9ytb1x5gFu/nfcHd6uodCSYoEfqgTFzEpao2cExQ== root@bastion.okd4.jakelee.com'
```

<br/>
install-config.yaml 파일은 manifest 와 ignition 생성후 삭제가 되기때문에 백업 폴더를 생성하여 저장한다. 


```bash
[root@bastion config]# mkdir backup
[root@bastion config]# cp install-config.yaml ./backup/install-config.yaml
```

<br/>
manifest 화일을 생성한다.  
openshift 폴더가 생성이되고 master/worker 노드 설정 화일들이 있어 여기 값을 수정하여 role 을 할당 할 수 있다.

<br/>

```bash
[root@bastion config]# /usr/local/bin/openshift-install create manifests --dir=/root/ocp/config/
INFO Consuming Install Config from target directory
WARNING Making control-plane schedulable by setting MastersSchedulable to true for Scheduler cluster settings
INFO Manifests created in: /root/ocp/config/manifests and /root/ocp/config/openshift
[root@bastion config]# cd openshift
[root@bastion openshift]# ls
99_kubeadmin-password-secret.yaml                      99_openshift-machineconfig_99-master-ssh.yaml
99_openshift-cluster-api_master-user-data-secret.yaml  99_openshift-machineconfig_99-worker-ssh.yaml
99_openshift-cluster-api_worker-user-data-secret.yaml  openshift-install-manifests.yaml
```

<br/>

coreos 설정을 위한 ignition 화일을 설정한다.  

<br/>

```bash
[root@bastion openshift]# cd ..
[root@bastion config]# /usr/local/bin/openshift-install create ignition-configs --dir=/root/ocp/config/
INFO Consuming Openshift Manifests from target directory
INFO Consuming OpenShift Install (Manifests) from target directory
INFO Consuming Master Machines from target directory
INFO Consuming Worker Machines from target directory
INFO Consuming Common Manifests from target directory
INFO Ignition-Configs created in: /root/ocp/config and /root/ocp/config/auth
[root@bastion config]# ls
auth  backup  bootstrap.ign  master.ign  metadata.json  worker.ign
```  

<br/>

bastion web 서버에 ign 화일을 복사하고 apache web server를 재 기동한다.

<br/>


```bash
[root@bastion config]# mkdir /var/www/html/ign
[root@bastion config]# cp *.ign /var/www/html/ign/
[root@bastion config]# chmod 777 /var/www/html/ign/*.ign
[root@bastion config]# systemctl restart httpd
```

<br/>

bootstrap 서버로 이동한다. 
먼저 네트웍 설정을 한다.  

<br/>

```bash
[root@localhost core]# nmtui
```  

<br/>

nmtui_bastion_new.png 추가

<br/>

```bash
[root@localhost core]# coreos-installer install  /dev/sda --insecure-ignition -I http://192.168.1.154:8080/ign/bootstrap.ign -n
Installing Red Hat Enterprise Linux CoreOS 48.84.202109241901-0 (Ootpa) x86_64 (512-byte sectors)
> Read disk 3.7 GiB/3.7 GiB (100%)
Writing Ignition config
Copying networking configuration from /etc/NetworkManager/system-connections/
Copying /etc/NetworkManager/system-connections/static.nmconnection to installed system
Install complete.
[root@localhost core]# reboot now
```

<br/>

다시 로그인 해보면

```bash
[root@bastion config]# ssh core@192.168.1.127
The authenticity of host '192.168.1.127 (192.168.1.127)' can't be established.
ECDSA key fingerprint is SHA256:7+MOJdsnC548GUrGZxYKTnvhG94F+2kyGa2bpSH6eA8.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.1.127' (ECDSA) to the list of known hosts.
Red Hat Enterprise Linux CoreOS 48.84.202109241901-0
  Part of OpenShift 4.8, RHCOS is a Kubernetes native operating system
  managed by the Machine Config Operator (`clusteroperator/machine-config`).

WARNING: Direct SSH access to machines is not recommended; instead,
make configuration changes via `machineconfig` objects:
  https://docs.openshift.com/container-platform/4.8/architecture/architecture-rhcos.html

---
This is the bootstrap node; it will be destroyed when the master is fully up.

The primary services are release-image.service followed by bootkube.service. To watch their status, run e.g.

  journalctl -b -f -u release-image.service -u bootkube.service
```

```bash
  [core@localhost ~]$   journalctl -b -f -u release-image.service -u bootkube.service
-- Logs begin at Mon 2023-08-07 05:48:45 UTC. --
Aug 07 05:50:23 localhost bootkube.sh[2232]: wrote /assets/ingress-operator-manifests/cluster-ingress-00-namespace.yaml
Aug 07 05:50:24 localhost bootkube.sh[2232]: Rendering MCO manifests...
Aug 07 05:50:32 localhost bootkube.sh[2232]: I0807 05:50:32.203425       1 bootstrap.go:86] Version: v4.8.0-202110020139.p0.git.6cf1670.assembly.stream-dirty (6cf167014583c41e80407eea5a4eda644f420d26)
Aug 07 05:50:32 localhost bootkube.sh[2232]: I0807 05:50:32.206504       1 bootstrap.go:188] manifests/machineconfigcontroller/controllerconfig.yaml
Aug 07 05:50:32 localhost bootkube.sh[2232]: I0807 05:50:32.208403       1 bootstrap.go:188] manifests/master.machineconfigpool.yaml
Aug 07 05:50:32 localhost bootkube.sh[2232]: I0807 05:50:32.208662       1 bootstrap.go:188] manifests/worker.machineconfigpool.yaml
Aug 07 05:50:32 localhost bootkube.sh[2232]: I0807 05:50:32.208867       1 bootstrap.go:188] manifests/bootstrap-pod-v2.yaml
Aug 07 05:50:32 localhost bootkube.sh[2232]: I0807 05:50:32.209096       1 bootstrap.go:188] manifests/machineconfigserver/csr-bootstrap-role-binding.yaml
Aug 07 05:50:32 localhost bootkube.sh[2232]: I0807 05:50:32.209326       1 bootstrap.go:188] manifests/machineconfigserver/kube-apiserver-serving-ca-configmap.yaml
Aug 07 05:50:32 localhost bootkube.sh[2232]: Rendering CCO manifests...
Aug 07 05:50:39 localhost bootkube.sh[2232]: time="2023-08-07T05:50:39Z" level=info msg="Rendering files to /assets/cco-bootstrap"
Aug 07 05:50:39 localhost bootkube.sh[2232]: time="2023-08-07T05:50:39Z" level=info msg="Writing file: /assets/cco-bootstrap/manifests/cco-cloudcredential_v1_operator_config_custresdef.yaml"
Aug 07 05:50:39 localhost bootkube.sh[2232]: time="2023-08-07T05:50:39Z" level=info msg="Writing file: /assets/cco-bootstrap/manifests/cco-cloudcredential_v1_credentialsrequest_crd.yaml"
Aug 07 05:50:39 localhost bootkube.sh[2232]: time="2023-08-07T05:50:39Z" level=info msg="Writing file: /assets/cco-bootstrap/manifests/cco-namespace.yaml"
Aug 07 05:50:39 localhost bootkube.sh[2232]: time="2023-08-07T05:50:39Z" level=info msg="Writing file: /assets/cco-bootstrap/manifests/cco-operator-config.yaml"
Aug 07 05:50:39 localhost bootkube.sh[2232]: time="2023-08-07T05:50:39Z" level=info msg="Rendering static pod"
Aug 07 05:50:39 localhost bootkube.sh[2232]: time="2023-08-07T05:50:39Z" level=info msg="writing file: /assets/cco-bootstrap/bootstrap-manifests/cloud-credential-operator-pod.yaml"
Aug 07 05:50:40 localhost bootkube.sh[2232]: https://localhost:2379 is healthy: successfully committed proposal: took = 8.685028ms
Aug 07 05:50:40 localhost bootkube.sh[2232]: Starting cluster-bootstrap...
Aug 07 05:50:46 localhost bootkube.sh[2232]: Starting temporary bootstrap control plane...
Aug 07 05:50:46 localhost bootkube.sh[2232]: Waiting up to 20m0s for the Kubernetes API
Aug 07 05:50:47 localhost bootkube.sh[2232]: Still waiting for the Kubernetes API: Get "https://localhost:6443/readyz": dial tcp [::1]:6443: connect: connection refused
```

<br/>

bastion으로 이동해서

```bash
[root@bastion config]# /usr/local/bin/openshift-install --dir=/root/ocp/config wait-for bootstrap-complete --log-level=debug
DEBUG OpenShift Installer 4.8.14
DEBUG Built from commit 0c5cd91b7f5b9c4497ce294e6af74afe30bd3fb1
INFO Waiting up to 20m0s for the Kubernetes API at https://api.okd4.jakelee.com:6443...
INFO API v1.21.1+a620f50 up
INFO Waiting up to 30m0s for bootstrapping to complete...
```


new  

```bash
[root@bastion named]# /usr/local/bin/openshift-install --dir=/root/ocp/config wait-for bootstrap-complete --log-level=debug
DEBUG OpenShift Installer 4.8.14
DEBUG Built from commit 0c5cd91b7f5b9c4497ce294e6af74afe30bd3fb1
INFO Waiting up to 20m0s for the Kubernetes API at https://api.okd4.shclub.duckdns.org:6443...
INFO API v1.21.1+a620f50 up
INFO Waiting up to 30m0s for bootstrapping to complete...
```  


<br/>


```bash
[root@localhost core]# nmtui
[root@localhost core]# coreos-installer install  /dev/sda --insecure-ignition -I http://192.168.1.154:8080/ign/master.ign -n
Installing Red Hat Enterprise Linux CoreOS 48.84.202109241901-0 (Ootpa) x86_64 (512-byte sectors)
> Read disk 3.7 GiB/3.7 GiB (100%)
Writing Ignition config
Copying networking configuration from /etc/NetworkManager/system-connections/
Copying /etc/NetworkManager/system-connections/static.nmconnection to installed system
Install complete.
```  

```bash
[root@localhost core]# coreos-installer install  /dev/sda --insecure-ignition -I http://192.168.1.69:8080/ign/master.ign -n
Installing Red Hat Enterprise Linux CoreOS 48.84.202109241901-0 (Ootpa) x86_64 (512-byte sectors)
> Read disk 3.7 GiB/3.7 GiB (100%)
Writing Ignition config
Copying networking configuration from /etc/NetworkManager/system-connections/
Copying /etc/NetworkManager/system-connections/static.nmconnection to installed system
Install complete.
[root@localhost core]#  reboot now
```  

master login

<br/>

```bash
[root@bastion shclub]# ssh core@192.168.1.145
The authenticity of host '192.168.1.145 (192.168.1.145)' can't be established.
ECDSA key fingerprint is SHA256:5DUuHQgrkA30P+93tSQ/1P1xKcvhS5yZQGMzuJiDYpc.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.1.145' (ECDSA) to the list of known hosts.
Red Hat Enterprise Linux CoreOS 48.84.202109241901-0
  Part of OpenShift 4.8, RHCOS is a Kubernetes native operating system
  managed by the Machine Config Operator (`clusteroperator/machine-config`).

WARNING: Direct SSH access to machines is not recommended; instead,
make configuration changes via `machineconfig` objects:
  https://docs.openshift.com/container-platform/4.8/architecture/architecture-rhcos.html
``` 

<br/>
아래 메시지가 나오면 master 서버 설치 종료
<br/>

```bash
[root@bastion config]# /usr/local/bin/openshift-install --dir=/root/ocp/config wait-for bootstrap-complete --log-level=debug
DEBUG OpenShift Installer 4.8.14
DEBUG Built from commit 0c5cd91b7f5b9c4497ce294e6af74afe30bd3fb1
INFO Waiting up to 20m0s for the Kubernetes API at https://api.okd4.jakelee.com:6443...
INFO API v1.21.1+a620f50 up
INFO Waiting up to 30m0s for bootstrapping to complete...
W0807 02:33:40.940487   17900 reflector.go:436] k8s.io/client-go/tools/watch/informerwatcher.go:146: watch of *v1.ConfigMap ended with: very short watch: k8s.io/client-go/tools/watch/informerwatcher.go:146: Unexpected watch close - watch lasted less than a second and no items received
DEBUG Bootstrap status: complete
INFO It is now safe to remove the bootstrap resources
DEBUG Time elapsed per stage:
DEBUG Bootstrap Complete: 21m46s
INFO Time elapsed: 21m46s
```

new

[root@bastion named]# /usr/local/bin/openshift-install --dir=/root/ocp/config wait-for bootstrap-complete --log-level=debug
DEBUG OpenShift Installer 4.8.14
DEBUG Built from commit 0c5cd91b7f5b9c4497ce294e6af74afe30bd3fb1
INFO Waiting up to 20m0s for the Kubernetes API at https://api.okd4.shclub.duckdns.org:6443...
INFO API v1.21.1+a620f50 up
INFO Waiting up to 30m0s for bootstrapping to complete...
W0807 10:13:19.869986    4715 reflector.go:436] k8s.io/client-go/tools/watch/informerwatcher.go:146: watch of *v1.ConfigMap ended with: very short watch: k8s.io/client-go/tools/watch/informerwatcher.go:146: Unexpected watch close - watch lasted less than a second and no items received
W0807 10:27:34.621111    4715 reflector.go:436] k8s.io/client-go/tools/watch/informerwatcher.go:146: watch of *v1.ConfigMap ended with: very short watch: k8s.io/client-go/tools/watch/informerwatcher.go:146: Unexpected watch close - watch lasted less than a second and no items received
I0807 10:27:45.681072    4715 trace.go:205] Trace[629431445]: "Reflector ListAndWatch" name:k8s.io/client-go/tools/watch/informerwatcher.go:146 (07-Aug-2023 10:27:35.647) (total time: 10033ms):
Trace[629431445]: ---"Objects listed" 10033ms (10:27:00.681)
Trace[629431445]: [10.033256522s] [10.033256522s] END
DEBUG Bootstrap status: complete
INFO It is now safe to remove the bootstrap resources
DEBUG Time elapsed per stage:
DEBUG Bootstrap Complete: 24m25s
INFO Time elapsed: 24m25s


<br/>

HAProxy 설정 변경

RHOCP 클러스터가 정상적으로 구성되었기 때문에, HAProxy에서 bootstrap으로 LB되지 않도록 수정 후 서비스를 재시작합니다.


<br/>

```bash
[root@bastion config]# vi /etc/haproxy/haproxy.cfg
```

<br/>

```bash
backend openshift_api_backend
    mode tcp
    balance source
    #server      bootstrap 192.168.1.127:6443 check
    server      okd-1 192.168.1.145:6443 check

backend ocp_machine_config_server_backend
    mode tcp
    balance source
    #server      bootstrap 192.168.1.127:22623 check
    server      okd-1 192.168.1.145:22623 check
```

<br/>

haproxy를  재기동 한다.

<br/>

```bash
[root@bastion config]# systemctl restart haproxy
```

<br/>

```bash
[root@bastion ~]# vi ~/.bash_profile
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
	. ~/.bashrc
fi

# User specific environment and startup programs
# /usr/local/bin 추가 
PATH=$PATH:$HOME/bin:/usr/local/bin

export PATH

# Added for okd4 : 아래 구문 추가
export KUBECONFIG=/root/ocp/config/auth/kubeconfig
```

<br/>

```bash
[root@bastion ~]# source ~/.bash_profile
[root@bastion ~]# oc get nodes
NAME        STATUS   ROLES           AGE   VERSION
localhost   Ready    master,worker   94m   v1.21.1+a620f50
```

<br/>

cluster componet 조회 
모든 Cluster Operator가 True / False / False 여야 정상입니다.

<br/>

```bash
[root@bastion ~]# oc get co
NAME                                       VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE
authentication                             4.8.14    True        False         False      70m
baremetal                                  4.8.14    True        False         False      82m
cloud-credential                           4.8.14    True        False         False      129m
cluster-autoscaler                         4.8.14    True        False         False      86m
config-operator                            4.8.14    True        False         False      87m
console                                    4.8.14    True        False         False      70m
csi-snapshot-controller                    4.8.14    True        False         False      64m
dns                                        4.8.14    True        False         False      82m
etcd                                       4.8.14    True        False         False      85m
image-registry                             4.8.14    True        False         False      77m
ingress                                    4.8.14    True        False         False      76m
insights                                   4.8.14    True        False         False      73m
kube-apiserver                             4.8.14    True        False         False      77m
kube-controller-manager                    4.8.14    True        False         False      82m
kube-scheduler                             4.8.14    True        False         False      84m
kube-storage-version-migrator              4.8.14    True        False         False      86m
machine-api                                4.8.14    True        False         False      85m
machine-approver                           4.8.14    True        False         False      86m
machine-config                             4.8.14    True        False         False      84m
marketplace                                4.8.14    True        False         False      86m
monitoring                                 4.8.14    True        False         False      70m
network                                    4.8.14    True        False         False      87m
node-tuning                                4.8.14    True        False         False      85m
openshift-apiserver                        4.8.14    True        False         False      77m
openshift-controller-manager               4.8.14    True        False         False      82m
openshift-samples                          4.8.14    True        False         False      77m
operator-lifecycle-manager                 4.8.14    True        False         False      85m
operator-lifecycle-manager-catalog         4.8.14    True        False         False      85m
operator-lifecycle-manager-packageserver   4.8.14    True        False         False      78m
service-ca                                 4.8.14    True        False         False      87m
storage                                    4.8.14    True        False         False      87m
```

<br/>

설치 확인을 한다.

```bash
[root@bastion ~]# openshift-install --dir=/root/ocp/config wait-for install-complete
INFO Waiting up to 40m0s for the cluster at https://api.okd4.jakelee.com:6443 to initialize...
INFO Waiting up to 10m0s for the openshift-console route to be created...
INFO Install complete!
INFO To access the cluster as the system:admin user when using 'oc', run 'export KUBECONFIG=/root/ocp/config/auth/kubeconfig'
INFO Access the OpenShift web-console here: https://console-openshift-console.apps.okd4.jakelee.com
INFO Login to the console with user: "kubeadmin", and password: "7jRt8-pN9vk-RCwBj-5Cd68"
INFO Time elapsed: 0s
```

new

```bash
[root@bastion named]# openshift-install --dir=/root/ocp/config wait-for install-complete
INFO Waiting up to 40m0s for the cluster at https://api.okd4.shclub.duckdns.org:6443 to initialize...
I0807 10:31:50.163900    5237 trace.go:205] Trace[1298498081]: "Reflector ListAndWatch" name:k8s.io/client-go/tools/watch/informerwatcher.go:146 (07-Aug-2023 10:31:40.149) (total time: 10014ms):
Trace[1298498081]: [10.014017046s] [10.014017046s] END
E0807 10:31:50.163931    5237 reflector.go:138] k8s.io/client-go/tools/watch/informerwatcher.go:146: Failed to watch *v1.ClusterVersion: failed to list *v1.ClusterVersion: an error on the server ("") has prevented the request from succeeding (get clusterversions.config.openshift.io)
I0807 10:32:01.730136    5237 trace.go:205] Trace[1427131847]: "Reflector ListAndWatch" name:k8s.io/client-go/tools/watch/informerwatcher.go:146 (07-Aug-2023 10:31:51.717) (total time: 10012ms):
Trace[1427131847]: [10.012876095s] [10.012876095s] END
E0807 10:32:01.730162    5237 reflector.go:138] k8s.io/client-go/tools/watch/informerwatcher.go:146: Failed to watch *v1.ClusterVersion: failed to list *v1.ClusterVersion: an error on the server ("") has prevented the request from succeeding (get clusterversions.config.openshift.io)
INFO Waiting up to 10m0s for the openshift-console route to be created...
INFO Install complete!
INFO To access the cluster as the system:admin user when using 'oc', run 'export KUBECONFIG=/root/ocp/config/auth/kubeconfig'
INFO Access the OpenShift web-console here: https://console-openshift-console.apps.okd4.shclub.duckdns.org
INFO Login to the console with user: "kubeadmin", and password: "MVckd-tGxbG-3DBhC-wBitK"
```

접속을 해봅니다

```bash
[root@bastion named]# oc login -u system:admin
Logged into "https://api.okd4.shclub.duckdns.org:6443" as "system:admin" using existing credentials.

You have access to 63 projects, the list has been suppressed. You can list all projects with 'oc projects'

Using project "default".
```

<br/> 
계정 생성

</br>
참고 : 
http://www.linuxdata.org/bbs/board.php?bo_table=OpenShift&wr_id=32&page=1&device=pc


https://gruuuuu.github.io/ocp/ocp4-authentication/


```bash
[root@bastion ~]# dnf install -y httpd-tools
Last metadata expiration check: 1:51:05 ago on Mon 07 Aug 2023 02:28:18 AM EDT.
Package httpd-tools-2.4.37-54.module_el8.8.0+1256+e1598b50.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
```


<br/>

```bash
[root@bastion ~]# touch htpasswd
[root@bastion ~]# htpasswd -Bb htpasswd shclub 'new1234!'
Adding password for user shclub
[root@bastion ~]# cat htpasswd
shclub:$2y$05$xBGKG6jt3sdL.Xl6cmxpROrRgWBxDhwR7KVtW6YgXX34robyHzVke
```
<br/>

secret 을 생성한다.

<br/>

```bash
[root@bastion ~]# oc --user=admin create secret generic htpasswd \
>     --from-file=htpasswd -n openshift-config
secret/htpasswd created
[root@bastion ~]# oc get secret  -n openshift-config
NAME                                      TYPE                                  DATA   AGE
builder-dockercfg-78tfr                   kubernetes.io/dockercfg               1      123m
builder-token-gg8d9                       kubernetes.io/service-account-token   4      123m
builder-token-hxgr2                       kubernetes.io/service-account-token   4      123m
default-dockercfg-lqmxs                   kubernetes.io/dockercfg               1      123m
default-token-d6pw2                       kubernetes.io/service-account-token   4      171m
default-token-qvtsl                       kubernetes.io/service-account-token   4      123m
deployer-dockercfg-bc67j                  kubernetes.io/dockercfg               1      123m
deployer-token-94dcq                      kubernetes.io/service-account-token   4      123m
deployer-token-r6vxg                      kubernetes.io/service-account-token   4      123m
etcd-client                               kubernetes.io/tls                     2      171m
etcd-metric-client                        kubernetes.io/tls                     2      171m
etcd-metric-signer                        kubernetes.io/tls                     2      171m
etcd-signer                               kubernetes.io/tls                     2      171m
htpasswd                                  Opaque                                1      20s
initial-service-account-private-key       Opaque                                1      171m
pull-secret                               kubernetes.io/dockerconfigjson        1      171m
webhook-authentication-integrated-oauth   Opaque                                1      126m
```  

<br/>

Cluster 에 적용

```bash
[root@bastion ~]# vi oauth-config.yaml
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: Local Password
    mappingMethod: claim
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpasswd
[root@bastion ~]# oc replace -f oauth-config.yaml
oauth.config.openshift.io/cluster replaced
```

<br/>

web console url을 확인한다.  

<br/>


```bash
[root@bastion ~]# oc whoami --show-console
https://console-openshift-console.apps.okd4.jakelee.com
```

<br/>

해당 유저로 로그인 합니다.

```bash
[root@bastion ~]# oc login https://api.okd4.jakelee.com:6443 -u shclub -p new1234! --insecure-skip-tls-verify
Login successful.
```

<br/>

test 라는 namespace를 생성합니다.  

```bash
[root@bastion ~]# oc new-project test
```  

해당 namespace 에 pod를 생성하면 시작이된 후 권한이 없어서 곧 에러가 발생합니다.

<br/>

admin 으로 로그인 한 후 anyuid 권한을 할당합니다.  

```bash
[root@bastion ~]# oc login -u system:admin
[root@bastion ~]# oc adm policy add-scc-to-user anyuid system:serviceaccount:test:default
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:anyuid added: "default"
```  


<br/>

pod를 하나 생성해 봅니다.

```bash
[root@bastion ~]# kubectl run nginx --image=nginx
pod/nginx created
[root@bastion ~]# kubectl get po
NAME    READY   STATUS              RESTARTS   AGE
nginx   0/1     ContainerCreating   0          4s
[root@bastion ~]# kubectl get po
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          8s
```    

정상적으로 Running 하는 것을 확인 할 수 있습니다.  

<br/>

참고  
- 소개 : https://velog.io/@_gyullbb/series/OKD
- 설치 (Main) : https://hkjeon2.tistory.com/104
- 설치 (Sub) : https://www.okd.io/guides/upi-sno/
#post-install
- 설치 (Baremetal) : https://gruuuuu.github.io/ocp/ocp4.7-restricted/
- 인증 : https://gruuuuu.github.io/ocp/ocp4-authentication/
- 미러 사이트 : https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.8/latest/
- HA Proxy : https://hoing.io/archives/2196
- DNS 서버 구축 : https://it-serial.tistory.com/entry/Linux-CentOS-7-DNS-%EC%84%9C%EB%B2%84-%EA%B5%AC%EC%B6%95-%EB%8F%84%EB%A9%94%EC%9D%B8-%EC%84%A4%EC%A0%95
- Proxmox 내부에 가상 사설망 구축: https://hwanstory.kr/@kim-hwan/posts/Proxmox-Virtual-Private-Network-Configuration
- pull secret 설정 : https://www.ibm.com/docs/ko/mas-cd/continuous-delivery?topic=platform-setting-up-bastion-host


<br/>

coreos ssh 패스워드로 연결 설정

```bash
[core@localhost ~]$ sudo su
[root@localhost core]# passwd core
Changing password for user core.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
[root@localhost core]# cd /etc/ssh
[root@localhost ssh]# ls
moduli      ssh_config.d        ssh_host_ecdsa_key.pub  ssh_host_ed25519_key.pub  ssh_host_rsa_key.pub
ssh_config  ssh_host_ecdsa_key  ssh_host_ed25519_key    ssh_host_rsa_key          sshd_config
```
<br/>
sshd_config 를 수정한다.  

```bash
[root@localhost ssh]# vi sshd_config
```  

<br/>

아래와 같이 no를 yes 로 변경하고 저장한다.  

```bash
     70 PasswordAuthentication yes
```  
<br/>

sshd 데몬을 재기동한다.  


```bash
[root@localhost ssh]# systemctl restart sshd
```

이제 어느 곳에서든 id/password로 접속 가능하다.  

<br/>


Using a pull secret from the Red Hat OpenShift Cluster Manager is not required. You can use a pull secret for another private registry. Or, if you do not need the cluster to pull images from a private registry, you can use {"auths":{"fake":{"auth":"aWQ6cGFzcwo="}}} as the pull secret when prompted during the installation.

If you do not use the pull secret from the Red Hat OpenShift Cluster Manager:

Red Hat Operators are not available.

The Telemetry and Insights operators do not send data to Red Hat.

Content from the Red Hat Ecosystem Catalog Container images registry, such as image streams and Operators, are not available.

<br/>

여기로 접속해 보면  https://console.redhat.com/openshift