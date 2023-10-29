

https://cwal.tistory.com/18

https://coffeewhale.com/apiserver


https://sphong0417.tistory.com/53

https://happycloud-lee.tistory.com/259

https://www.samsungsds.com/kr/insights/kubernetes_security_automation.html


https://docs.openshift.com/container-platform/4.8/post_installation_configuration/machine-configuration-tasks.html


https://stackoverflow.com/questions/51027036/what-is-the-difference-between-apiserver-kubelet-client-apiserver-and-kubelet-c


인증서 기반 그룹 유저 생성   

https://passwd.tistory.com/entry/k8s-%EC%9D%B8%EC%A6%9D%EC%84%9C-%EA%B8%B0%EB%B0%98-Group-User-%EC%83%9D%EC%84%B1?category=1213453


role , rolebinding 정리 좋음
https://jangcenter.tistory.com/123

openssl  x509 -in kubelet-client.pem -text

```bash
[root@okd-4 pki]# pwd
/var/lib/kubelet/pki
[root@okd-4 pki]# openssl  x509 -in kubelet-client-current.pem -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            14:73:e6:28:19:d3:10:56:13:0f:ab:8c:ca:57:20:70
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = kube-csr-signer_@1695003522
        Validity
            Not Before: Sep 18 05:28:57 2023 GMT
            Not After : Oct 18 02:18:43 2023 GMT
        Subject: O = system:nodes, CN = system:node:okd-4.okd4.ktdemo.duckdns.org
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    04:c7:f6:44:40:96:df:aa:8d:cc:56:98:59:80:aa:
                    3e:8d:25:ea:b5:d1:d7:69:f9:33:55:94:8d:03:df:
                    72:61:c4:f8:11:8a:e9:46:9a:74:3b:49:ea:ab:80:
                    58:f3:07:e5:2c:f2:3a:53:b8:e2:29:29:ab:06:16:
                    cb:e8:14:f0:31
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage:
                TLS Web Client Authentication
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Authority Key Identifier:
                18:EA:C7:DE:10:D0:65:B1:9A:37:A9:E7:F9:91:C5:94:19:1B:1F:5E
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        55:b1:21:01:18:5a:31:67:bc:f8:7e:e7:e1:dd:1e:ae:83:82:
        97:2b:a9:b4:c5:5c:0a:cf:20:55:a8:90:26:1d:c1:84:f4:1e:
        0f:b0:aa:7d:35:4d:79:6b:77:f5:18:9a:b4:1a:dc:4f:03:ee:
        c7:6b:62:ef:51:a1:ab:45:84:02:04:bf:59:91:5f:d8:20:42:
        5e:05:d3:40:89:a3:f2:9f:b0:a0:4b:51:b2:60:59:39:ad:34:
        1d:d2:48:38:5d:6b:cb:cd:18:bd:20:24:bc:08:5f:04:f2:9d:
        76:df:6c:ae:47:fb:72:a6:31:81:70:c4:cb:c3:fe:73:2b:d5:
        7e:6b:f4:aa:d1:87:a5:df:96:32:16:7f:bb:e6:fa:3f:03:40:
        60:02:d8:e7:33:ef:ba:bb:59:06:51:c8:41:79:45:ef:97:3f:
        f8:56:20:79:98:8f:26:04:03:45:46:55:b9:c2:03:72:7f:6b:
        c8:c5:d7:5e:24:e2:c0:e1:6a:87:71:dd:55:e7:d4:e0:0f:41:
        99:9a:83:c5:36:60:b0:d1:83:57:9e:12:6a:90:5b:fa:35:34:
        4a:90:54:09:93:4e:be:3d:a7:e7:5b:a3:d1:98:a5:ab:fb:ee:
        25:81:4e:8e:d6:7a:94:c6:01:17:56:b9:6b:7d:1e:40:83:a0:
        38:78:75:2d
-----BEGIN CERTIFICATE-----
MIIChjCCAW6gAwIBAgIQFHPmKBnTEFYTD6uMylcgcDANBgkqhkiG9w0BAQsFADAm
MSQwIgYDVQQDDBtrdWJlLWNzci1zaWduZXJfQDE2OTUwMDM1MjIwHhcNMjMwOTE4
MDUyODU3WhcNMjMxMDE4MDIxODQzWjBLMRUwEwYDVQQKEwxzeXN0ZW06bm9kZXMx
MjAwBgNVBAMTKXN5c3RlbTpub2RlOm9rZC00Lm9rZDQua3RkZW1vLmR1Y2tkbnMu
b3JnMFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEx/ZEQJbfqo3MVphZgKo+jSXq
tdHXafkzVZSNA99yYcT4EYrpRpp0O0nqq4BY8wflLPI6U7jiKSmrBhbL6BTwMaNW
MFQwDgYDVR0PAQH/BAQDAgWgMBMGA1UdJQQMMAoGCCsGAQUFBwMCMAwGA1UdEwEB
/wQCMAAwHwYDVR0jBBgwFoAUGOrH3hDQZbGaN6nn+ZHFlBkbH14wDQYJKoZIhvcN
AQELBQADggEBAFWxIQEYWjFnvPh+5+HdHq6DgpcrqbTFXArPIFWokCYdwYT0Hg+w
qn01TXlrd/UYmrQa3E8D7sdrYu9RoatFhAIEv1mRX9ggQl4F00CJo/KfsKBLUbJg
WTmtNB3SSDhda8vNGL0gJLwIXwTynXbfbK5H+3KmMYFwxMvD/nMr1X5r9KrRh6Xf
ljIWf7vm+j8DQGAC2Ocz77q7WQZRyEF5Re+XP/hWIHmYjyYEA0VGVbnCA3J/a8jF
114k4sDhaodx3VXn1OAPQZmag8U2YLDRg1eeEmqQW/o1NEqQVAmTTr49p+dbo9GY
pav77iWBTo7WepTGARdWuWt9HkCDoDh4dS0=
-----END CERTIFICATE-----
[root@okd-4 pki]# ls
kubelet-client-2023-09-18-05-33-57.pem  kubelet-client-current.pem  kubelet-server-2023-09-18-05-34-21.pem  kubelet-server-current.pem
[root@okd-4 pki]# openssl  x509 -in kubelet-server-current.pem -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            a2:24:1e:38:57:aa:c3:2f:25:06:5d:6f:1a:9d:c2:b0
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: CN = kube-csr-signer_@1695003522
        Validity
            Not Before: Sep 18 05:29:21 2023 GMT
            Not After : Oct 18 02:18:43 2023 GMT
        Subject: O = system:nodes, CN = system:node:okd-4.okd4.ktdemo.duckdns.org
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    04:6d:9b:1b:cb:1f:79:0b:59:85:e1:d7:a2:06:48:
                    8f:fb:1e:50:dd:07:80:80:a1:b1:fa:1e:89:ef:9b:
                    d1:73:3e:28:a5:f8:e6:f6:55:fd:db:18:ed:a0:f3:
                    46:5c:a1:db:13:aa:fc:36:d5:4b:7a:8e:d6:05:72:
                    39:9e:94:5c:53
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Key Encipherment
            X509v3 Extended Key Usage:
                TLS Web Server Authentication
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Authority Key Identifier:
                18:EA:C7:DE:10:D0:65:B1:9A:37:A9:E7:F9:91:C5:94:19:1B:1F:5E
            X509v3 Subject Alternative Name:
                DNS:okd-4.okd4.ktdemo.duckdns.org, IP Address:192.168.1.150
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        70:a4:b6:13:2c:b2:06:0f:eb:69:1f:71:ad:0a:3e:49:00:2a:
        86:4a:38:69:fa:97:8a:52:72:93:22:37:3d:24:0a:ec:94:38:
        35:ab:70:65:33:48:bf:ca:76:b3:76:84:c9:cc:e9:af:bd:23:
        ce:93:a0:73:dd:f2:22:54:69:2f:02:10:29:a2:66:99:a5:35:
        73:8f:dc:9c:bb:46:a5:f8:38:b8:2b:95:1a:90:42:f6:86:9a:
        2c:f7:36:14:0d:66:ba:e6:20:b2:cd:34:45:0f:bb:f2:46:34:
        b9:a9:00:5a:95:3d:1e:6f:33:c7:b9:69:be:6b:10:5f:f6:6d:
        53:f1:b4:ca:f9:bf:50:11:52:c2:a5:f0:46:f5:a9:f5:25:41:
        26:bd:ae:41:61:cb:f7:da:07:d2:04:30:f2:b6:c1:da:b9:f9:
        46:da:ea:22:16:fb:66:e7:d5:1e:99:ca:98:bf:3c:d4:95:42:
        ff:eb:1c:6b:74:ff:98:bf:2f:76:13:24:e6:42:ce:99:68:37:
        76:43:56:86:80:2b:0d:26:d8:e6:9a:c3:3a:c8:b4:13:96:a4:
        86:12:da:13:be:ab:63:ff:d1:95:93:2d:96:63:14:6e:16:36:
        6f:73:f6:69:04:74:92:19:a0:21:45:56:cd:1d:93:89:ea:bc:
        4a:ee:44:30
-----BEGIN CERTIFICATE-----
MIICuTCCAaGgAwIBAgIRAKIkHjhXqsMvJQZdbxqdwrAwDQYJKoZIhvcNAQELBQAw
JjEkMCIGA1UEAwwba3ViZS1jc3Itc2lnbmVyX0AxNjk1MDAzNTIyMB4XDTIzMDkx
ODA1MjkyMVoXDTIzMTAxODAyMTg0M1owSzEVMBMGA1UEChMMc3lzdGVtOm5vZGVz
MTIwMAYDVQQDEylzeXN0ZW06bm9kZTpva2QtNC5va2Q0Lmt0ZGVtby5kdWNrZG5z
Lm9yZzBZMBMGByqGSM49AgEGCCqGSM49AwEHA0IABG2bG8sfeQtZheHXogZIj/se
UN0HgIChsfoeie+b0XM+KKX45vZV/dsY7aDzRlyh2xOq/DbVS3qO1gVyOZ6UXFOj
gYcwgYQwDgYDVR0PAQH/BAQDAgWgMBMGA1UdJQQMMAoGCCsGAQUFBwMBMAwGA1Ud
EwEB/wQCMAAwHwYDVR0jBBgwFoAUGOrH3hDQZbGaN6nn+ZHFlBkbH14wLgYDVR0R
BCcwJYIdb2tkLTQub2tkNC5rdGRlbW8uZHVja2Rucy5vcmeHBMCoAZYwDQYJKoZI
hvcNAQELBQADggEBAHCkthMssgYP62kfca0KPkkAKoZKOGn6l4pScpMiNz0kCuyU
ODWrcGUzSL/KdrN2hMnM6a+9I86ToHPd8iJUaS8CECmiZpmlNXOP3Jy7RqX4OLgr
lRqQQvaGmiz3NhQNZrrmILLNNEUPu/JGNLmpAFqVPR5vM8e5ab5rEF/2bVPxtMr5
v1ARUsKl8Eb1qfUlQSa9rkFhy/faB9IEMPK2wdq5+Uba6iIW+2bn1R6Zypi/PNSV
Qv/rHGt0/5i/L3YTJOZCzploN3ZDVoaAKw0m2OaawzrItBOWpIYS2hO+q2P/0ZWT
LZZjFG4WNm9z9mkEdJIZoCFFVs0dk4nqvEruRDA=
-----END CERTIFICATE-----
```


<br/>

```bash
[root@okd-4 pki]# cd /etc/kubernetes
[root@okd-4 kubernetes]# ls
ca.crt  cloud.conf  cni  kubeconfig  kubelet-ca.crt  kubelet-plugins  kubelet.conf  manifests  static-pod-resources
[root@okd-4 kubernetes]# cat kubelet.conf
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  x509:
    clientCAFile: /etc/kubernetes/kubelet-ca.crt
  anonymous:
    enabled: false
cgroupDriver: systemd
cgroupRoot: /
clusterDNS:
  - 172.30.0.10
clusterDomain: cluster.local
containerLogMaxSize: 50Mi
maxPods: 250
kubeAPIQPS: 50
kubeAPIBurst: 100
podPidsLimit: 4096
rotateCertificates: true
serializeImagePulls: false
staticPodPath: /etc/kubernetes/manifests
systemCgroups: /system.slice
featureGates:
  APIPriorityAndFairness: true
  RotateKubeletServerCertificate: true
  DownwardAPIHugePages: true
  CSIMigrationAzureFile: false
  CSIMigrationvSphere: false
serverTLSBootstrap: true
tlsMinVersion: VersionTLS12
tlsCipherSuites:
  - TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
  - TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
  - TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
  - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
  - TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256
  - TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256
```  

<br/>

```bash
[root@okd-4 kubernetes]# cat kubelet-ca.crt
-----BEGIN CERTIFICATE-----
MIIDMDCCAhigAwIBAgIITw6Zmi8uHKYwDQYJKoZIhvcNAQELBQAwNjESMBAGA1UE
CxMJb3BlbnNoaWZ0MSAwHgYDVQQDExdhZG1pbi1rdWJlY29uZmlnLXNpZ25lcjAe
Fw0yMzA5MDEwMDM2NTNaFw0zMzA4MjkwMDM2NTNaMDYxEjAQBgNVBAsTCW9wZW5z
aGlmdDEgMB4GA1UEAxMXYWRtaW4ta3ViZWNvbmZpZy1zaWduZXIwggEiMA0GCSqG
SIb3DQEBAQUAA4IBDwAwggEKAoIBAQC1CVY2AqzQFwl4ZY+yR2xYRBQSHC1dnGbj
XNtu4jS5DGUHsSwvRMPVRT3JGMtu0gNhiEixtJ9Wq1L6ukZ15ZMVxyc0G+2V8BH8
daaUDe56iqqjrDxA70MskmZ33HEQeur/51ZsfgJyISVCId+S+7FuzHtEjx9xkXlC
a1okdVgiTiDb3EqYAVTFX1ETg6GD/8RLwiVbhUMaW42yCO0ZB8G1u8nRA7/nbYBC
VG4M8tHxW0eHrgJRF7vZQy9yRLt0TPqFMl9NFr1BNczt2AGH1JkTTaUIlYQf8Wyn
uRXG5JGobruAlCoS1nnDmf/Us1ZRLCtzjvNBmzGY0L/CmlPjyoyRAgMBAAGjQjBA
MA4GA1UdDwEB/wQEAwICpDAPBgNVHRMBAf8EBTADAQH/MB0GA1UdDgQWBBQ9P+d+
4ZMq6da9ljSd6KiNV3EB9TANBgkqhkiG9w0BAQsFAAOCAQEASZIMHYGSLzyzKgdQ
LhAc1m151vbNEiWWpu5JbhrtP+5s/EcapqQ+Zx0QkjfbTpQl4KM1PuWmff07BQhI
cb4Hw+8B08bHnBEOYsHdxaaKrToWpT37qYiboeIdpwFgHqKgKdDy0e9g3oThBW4q
DLTmrjL6c93z+58QDUPcvZ0CUPcViwQvl+vB3YBbjj4eBrfET1troHXP6gt7D5Os
ZlkSMjGVY0pTZvDKdKaKwvUe/3iLNgjuaZKCLUnTwSHSqAMSRcvYmtmzGXsTXpxR
H1FbI+UusN/4jnHKg3/M1+WBTfGwVWcN54V21dwYSkYBerWJtTkhOR+e5uyIHsgM
J42j9A==
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
MIIDHjCCAgagAwIBAgIIRjs0nVNCwr4wDQYJKoZIhvcNAQELBQAwLTESMBAGA1UE
CxMJb3BlbnNoaWZ0MRcwFQYDVQQDEw5rdWJlbGV0LXNpZ25lcjAeFw0yMzA5MDEw
MDM2NTdaFw0yMzA5MDIwMDM2NTdaMC0xEjAQBgNVBAsTCW9wZW5zaGlmdDEXMBUG
A1UEAxMOa3ViZWxldC1zaWduZXIwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEK
AoIBAQC1cglY0B4OFeHQHqsQBl2U6FFscb3SRkFlO4N4f83u+yMUNBuiB1+CFQZX
17iQRZtyYaOjOy2TWljid9iWSz2ewdwtEqW7dXbzNKcs+yNYr8TIDIF6uezBuWXN
vLuZ5XBu/jigfjByVV7lsEU2gbZA8UjbvlGFp0w1J0dT1tSsDuuD96oQSGMN2bq+
Cl19dLkv+xUvc9qNl1yFE83KSdat+irBWTiKoQo7IT17stH7ycz8C+IRUAbitNAi
Q4A4EBN83DPfW6aEv9D8KBEYGHgttNpYC+xOFqtY3+wXnSpt9z9Lw0j4lc4sElGN
2TVU5MCH1z5L0ZP7AHk1d5PtYzIbAgMBAAGjQjBAMA4GA1UdDwEB/wQEAwICpDAP
BgNVHRMBAf8EBTADAQH/MB0GA1UdDgQWBBRKE7v1ljYBXLr+JgaWger1S2fSlTAN
BgkqhkiG9w0BAQsFAAOCAQEARYXh3QzxJt13l2cj5PVlvIv3afUwRfmE2ZzvEoBW
qw9KE/OikIatXdY57zpKVSJAhwmU2URvKTeeU79G2xTGCpw0liyn0EWVOsXsuMuR
ht/2BaKecGs4CsQMC6qinRGsAXGC07UuxRbpUEhiEnoIG9z9WeOAL9uILL4JdeFO
BnJSrlAe+GgLZIXdgGsdqEipyu+d9ao0Z9z6d/QR7WuiPP2QUp0dTgzHMTJ5gX0E
Aqx6AxS5c+0fsNu8jjuAhwNqFXilULjPIQwV2AK6UK+9uLNXL4csYjfabLStYqgz
SxtjbTP5Hzzpil8OVW6D20a0r8Uv3VBRNSWP0areLC58XA==
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
MIIDNDCCAhygAwIBAgIIU19MHiiUBdMwDQYJKoZIhvcNAQELBQAwODESMBAGA1UE
CxMJb3BlbnNoaWZ0MSIwIAYDVQQDExlrdWJlLWNvbnRyb2wtcGxhbmUtc2lnbmVy
MB4XDTIzMDkwMTAwMzY1OFoXDTI0MDgzMTAwMzY1OFowODESMBAGA1UECxMJb3Bl
bnNoaWZ0MSIwIAYDVQQDExlrdWJlLWNvbnRyb2wtcGxhbmUtc2lnbmVyMIIBIjAN
BgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAy8YbqqiqJf5oqj2qrswLJxPfSef6
ThW3N4zudoP6hXiEJkqulk4KJtj7v37GBMfTW73vH5pow2DbGK4qu3B/twTNu4fs
BkYnnembittbG3aqnJB/uGCWF5IThuAiN5SnFcsN6LeSSDvhIbJ6+zP14bsEBmIr
C7RPB1RPJm7wSmewxy5QIc7IkRsO2WzaQseSAP++9ZY42qJYZmGJqsEz3/MaJzOH
So5olvoKopkwu6m4eybqX8NV6bmmmjiYLaWZEvWI4hVeZAWsE+kixUUMR4zkTXYa
Q2s0fyqrz1CnSJIbm+CAd4DkjCkeP5H/1SxuuuVbsr4eWUq3W9pVRHPK/QIDAQAB
o0IwQDAOBgNVHQ8BAf8EBAMCAqQwDwYDVR0TAQH/BAUwAwEB/zAdBgNVHQ4EFgQU
P3cYXSfUNfqQWg3JAr5d568VTAkwDQYJKoZIhvcNAQELBQADggEBAFQ4rFSjQTIK
g1XffgqSrp5l5OlW6uXpXMd0OXif6YQtF0e7raQcCEIKDTLwr8MzaY4yDWCK6d0d
LEblw6F6ZJkIukWjOvcS3kKM5j+KHf+WPwDIMU4SBRnUvdueBfzvuBHkkNhn51zv
8HC/yk1X0IHaGsCcuExETDtmJJxI+5U2sEVnFAZRNoUCPTgvsUpQp46mimp6ntIj
Pnl36qh+JcQqLxg3zmq7tXnlde/pd7sL/N0x/BOdKFxGQyTdmTITRgSnEZ6lnPqV
Ya3ptQ1loXO4FrfFws+Yba7IA7kYgk46l54pMh/wif315VZ34DNn9UrQV/lBMfv3
42kinbMHdY4=
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
MIIDQjCCAiqgAwIBAgIIbl1Z6CJogZAwDQYJKoZIhvcNAQELBQAwPzESMBAGA1UE
CxMJb3BlbnNoaWZ0MSkwJwYDVQQDEyBrdWJlLWFwaXNlcnZlci10by1rdWJlbGV0
LXNpZ25lcjAeFw0yMzA5MDEwMDM2NThaFw0yNDA4MzEwMDM2NThaMD8xEjAQBgNV
BAsTCW9wZW5zaGlmdDEpMCcGA1UEAxMga3ViZS1hcGlzZXJ2ZXItdG8ta3ViZWxl
dC1zaWduZXIwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDS6ER5sqq6
UNan24BdNCsa9RVaShvNplp+X5sx0RoPFxxkCN94AFIZCeAtXMj/bNp9nT68liUq
yg+e7AZuk3Ll3Whqi92B88gzQsOyQzDmwMfDU77OPvquLFttGj17j9S7dBbE8oaQ
Bg4ecrs9Rv9+FWp2gu9bEiuYrt4XLrVSsHAwIFZW5xQNTB9orCbmZmI6DsJeeoxr
GXmrTsj2KmFMx9sPJBx8l0qz46btTxQv/ter+vOiGAPr7gUFG2rdfaYdQTl5kyja
X4peb7f5kU5cqeWAwv8nZ/C8WXe9OLf97SprwpdJnVVA4R+M+fdKw45dYQzqDoer
7Kom5rOE2+XtAgMBAAGjQjBAMA4GA1UdDwEB/wQEAwICpDAPBgNVHRMBAf8EBTAD
AQH/MB0GA1UdDgQWBBREAvdgCwRLZu3L4xl7/gLvfehSazANBgkqhkiG9w0BAQsF
AAOCAQEAiKuEIxB93g56uo3ildjarYgs8lsUr0Ex6AEjpePmIWcC+xXz9X6oLFhn
Spo/5qXKHIvUCEZR8lJL4m6Ie9+WLiRmnBm/TbTVODzHuO8dmIbe4ePkycNEpcYr
t2/PpkD1gksc5SXvDS6Kzgoz/MSbiCixiD0cEWN/k21QWv6+fi2Aaz1Ur6LgsaHF
i9cRk3GOdvjaMYsLB8mZ5qYgfgdyYuA0vJgLHUzvPU8uo1Iw2OIt6qkFP1qOjWZQ
0LlAH2KyWvgyiq3h+f2tVk/VQYmr/CKeld8n4T3xp9ivSi4GOeYG6R8h/knYGizn
yLKQ3aV4MyfxwRs7ucj+JbvVcORdMw==
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
MIIDSDCCAjCgAwIBAgIIMONkbRPJIKUwDQYJKoZIhvcNAQELBQAwQjESMBAGA1UE
CxMJb3BlbnNoaWZ0MSwwKgYDVQQDEyNrdWJlbGV0LWJvb3RzdHJhcC1rdWJlY29u
ZmlnLXNpZ25lcjAeFw0yMzA5MDEwMDM2NTRaFw0zMzA4MjkwMDM2NTRaMEIxEjAQ
BgNVBAsTCW9wZW5zaGlmdDEsMCoGA1UEAxMja3ViZWxldC1ib290c3RyYXAta3Vi
ZWNvbmZpZy1zaWduZXIwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDn
zAUU/mGy3D9WkSkb6iY42aFMqWd1YWvemR0cvNWmsEs2mEJP6c1iwLUvI8241VsH
ICPWt+utesFFXJFotxULIY2opi1xRbRDCmx3EVlUYk8ZpDKrK1R8QYfLUauQd+Qe
agF6Uc1bIfP0RHE0/6h5kX//Azc3kv4wB3I65fcO3zdct2Luu85RUTIbdeZXp7UY
fLH7VmHao56ftYXKMdB8N+2C+tM0nhRZ4jSh3Oq3bPISwnucTNAHtt1m+wYTotqY
LlrphZsLRPMsQF7wlpAYW/yo6q0HoJIgYZrtcvhouKci0FwJJ6EMRcjnF1IHsHoY
ExdQS15l5wYoBMQ5uXPtAgMBAAGjQjBAMA4GA1UdDwEB/wQEAwICpDAPBgNVHRMB
Af8EBTADAQH/MB0GA1UdDgQWBBSTalsKLPIUFJLxbecw8TvajMW3bzANBgkqhkiG
9w0BAQsFAAOCAQEAZfEGzNc6i1WgDtZSTBCcdHipn+1kSUwm+QDU0GiXEESNGBLE
DRB0UWX5GQJWdBIkVnLFK5oljaABKG0b3XVjQZYtChnKzEIUbFNlOv0w4vidExQc
Qx82AtPlzlWtGa+HztLq9vOgwVoCJXt6g4s3PD+Lun++j4s3ycEA9Qp4NCOmh2Z7
LdMu4mYGIfN1FZugTo80eQ07gx1zypT8pklhyTbtc1HG7NGE+vW6J1spzjHbRqa2
qMuqZ9wtw3J+oVWAKkffUq88lbmjSWzGziyxRyv+QLYt+yZI10nxm5wdx4iz1oZ6
DIf0T3dENKlSgpz4ODvlxlNEKCk8uO0F3a/STA==
-----END CERTIFICATE-----
```  

<br/>

```bash
[root@okd-4 kubernetes]# cat ca.crt
-----BEGIN CERTIFICATE-----
MIIDEDCCAfigAwIBAgIIPSNBCrIP8f0wDQYJKoZIhvcNAQELBQAwJjESMBAGA1UE
CxMJb3BlbnNoaWZ0MRAwDgYDVQQDEwdyb290LWNhMB4XDTIzMDkwMTAwMzY0MVoX
DTMzMDgyOTAwMzY0MVowJjESMBAGA1UECxMJb3BlbnNoaWZ0MRAwDgYDVQQDEwdy
b290LWNhMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA0gT2Bwr3JASo
wkJtNBkv9bGfd3yF+25pNdhIl8ndbsB52xjoAMLdPRzDW9+AZCDx1JP9PGn9BzyN
8+e1bMT04pXLTZCrEsefGTpk0aYcmlKSx0i8VaE5pBmHvX45ZPMXjkQzblY+FcpA
hrvJ0KA8y9yp2RBYroY2eSKq0vRBeD+ZCGreTJuGkCqJU1XKy0Dv4M5F9a8CwMKW
qYTJB0i8hVxx+ljNpKQkIr4vesNBClDDUg4uYQcuyfEDKp9jH8NI0Wbx9hPzlfI8
i2/0KcUvv+UDSVv/pp4EXxAvgIow1XlBu7oJaUf0IBVDYRyCYR/Q6D+B4W0me16k
RPUePJVP1QIDAQABo0IwQDAOBgNVHQ8BAf8EBAMCAqQwDwYDVR0TAQH/BAUwAwEB
/zAdBgNVHQ4EFgQUyEEedyc5XS+KGPcJ/su0Sy7ckXkwDQYJKoZIhvcNAQELBQAD
ggEBAGyc7W8rKq8XEd5seAHIJJWtXwvOVsMQkUZ9v+DjniITr6DUSrilEGC0f3O1
OOnNdZ29ETm6h1l6NTmuYKh+byWN9xFELfqIqozQBxmS5FV0F+5pqDNjlJ7QlHfC
TfDMdNy49XFA8hp8kOvILqtxblQyTzsZ73vDVPykMNIs0S4hRir8PTrktkjFjCH2
TDAHLDmuOuM06WDH/GFEBl9ZGIlvm8Ta64co/4NRiMYjNZZsAwI1JR4t99589RjO
z4RaT9TYvahfW3uBnca7NBxwoYKSUbU8a+Tk87m6Z4zntgXVxTV9QuNdfCpvIMM1
58DDzBvs69PjLGNBdp7yV2peBQ8=
-----END CERTIFICATE-----
```  

<br/>

```bash
[root@okd-4 kubernetes]# cat kubeconfig
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURNakNDQWhxZ0F3SUJBZ0lJTlJZalpSZG0rQTR3RFFZSktvWklodmNOQVFFTEJRQXdOekVTTUJBR0ExVUUKQ3hNSmIzQmxibk5vYVdaME1TRXdId1lEVlFRREV4aHJkV0psTFdGd2FYTmxjblpsY2kxc1lpMXphV2R1WlhJdwpIaGNOTWpNd09UQXhNREF6TmpVMFdoY05Nek13T0RJNU1EQXpOalUwV2pBM01SSXdFQVlEVlFRTEV3bHZjR1Z1CmMyaHBablF4SVRBZkJnTlZCQU1UR0d0MVltVXRZWEJwYzJWeWRtVnlMV3hpTFhOcFoyNWxjakNDQVNJd0RRWUoKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBT2NIK0lwSU55c0w3S1NmUG1NaFNlUnYxayt1YUxXSApYL0E3em5DSlgzdWZiaWRXYVJiUjVUbHRVcVRDUUh2akZDWnJlT2txNjlUZGIyL1NsV04rQ3dCY1VZS2d3WGRoCkV6NjJGcTBDTHJPSHZPV3ZLeW04MGNnaVlHZ2lkR1BOdVBvbUdSTHVMZW04UXQyZGdONjU4V1hRUWJSWEtsK2cKOVUvaFpwYUs2RmphZWsvOW10WUJNQTNOMlV5b1BGa3dBSFExOFYzSFBKNmc4b3lJRXZjTjBwcmxCbHlrWUNZTApWSXBBRG9SQWE2RDgvZ0hCTk9jT2JPeFVvRVY4VERlR1p4d1VrREhIdDRXMnhqSHEvVjZDcXRmT05YL3BSc0pjCmV0NktMT2YyditlVXBxWVdIUzhKbDRCRHFOeWFVVTdXdzQxNW1SZW11M3ZGV3RKeEJSNGMzejBDQXdFQUFhTkMKTUVBd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCL3dRRk1BTUJBZjh3SFFZRFZSME9CQllFRk1nVApxckNtYnMyTW44MCtzZGgvVDFjdFl5UzhNQTBHQ1NxR1NJYjNEUUVCQ3dVQUE0SUJBUUNnSHduUXVzTU11S3hrCitsdWtBZEsrUmNZS3BWT2ZtZXU1N3lrc2Z2Q2ZvVGV0dUg4UWUybkF6Qm1BOEtscFR2cEMrWndUVm5ZUlA5SzIKWnR3SDAxenNzSW4zU1ZDK1gvQlVOZlNGNFZyMUZGbi9iL28xNjE4eVdtcXZsZ1ROVzdIaG9vaDlta1R5WlQvZwo1YVpzaEhGUXRKeno5eDFHVFU5STI4U0Yxei8wbitiWHRvSm8wZ3BJUU9FM3lhMGhPUmk5Ull1ZVBDMDlhK281Ci9PVnNoKzI1YzF6TTU2Q04xRGxBMm42WDVyVEFDaU9ueHYwUFkwMmplOERleG9IWkJMZDVEN2dtWmJTRWNsRFoKTUFvTm40cWhRYmVzdDVFWWtHS3I5VUpISi96aWQreEFNcUJMYkNhdGlUZEVOc2s2ZEJXZHhDMk9nczdJU2J3SgprVzdrcHFDbgotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCi0tLS0tQkVHSU4gQ0VSVElGSUNBVEUtLS0tLQpNSUlEUURDQ0FpaWdBd0lCQWdJSVplZ0k0SGNPa0I0d0RRWUpLb1pJaHZjTkFRRUxCUUF3UGpFU01CQUdBMVVFCkN4TUpiM0JsYm5Ob2FXWjBNU2d3SmdZRFZRUURFeDlyZFdKbExXRndhWE5sY25abGNpMXNiMk5oYkdodmMzUXQKYzJsbmJtVnlNQjRYRFRJek1Ea3dNVEF3TXpZMU5Gb1hEVE16TURneU9UQXdNelkxTkZvd1BqRVNNQkFHQTFVRQpDeE1KYjNCbGJuTm9hV1owTVNnd0pnWURWUVFERXg5cmRXSmxMV0Z3YVhObGNuWmxjaTFzYjJOaGJHaHZjM1F0CmMybG5ibVZ5TUlJQklqQU5CZ2txaGtpRzl3MEJBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUF5Y3A4OTBkYkNPM0EKOEVUVFFYaG5ZZjFmOUljVTU4aExGOFREQ29qbkdxNEk3d3dwaXpBeGVzdmRnTDFTcVVPbm1IbmE1dE9HQ1cyUQpPSS82cnZLM29JNXVUT0dNbFpKOUJ4bktpTGtBKzU4U0xicXBpbTNwREx5bjU4TjlJOVFldjZtWTZaNnRSazRwCkxWZk9uOWxLdE9kVEF4c3RTTVJZbHhwbWNiZ0FiOThMUzJwMERxQlhCRnFUZGRXaDVoeHBVV20zNnBNMWswS1gKVHJmamlkNXpIS1QzQ3BuSXRKa0c3VUdJT3ZQVGVzOWFLeStjV0VseVYvUE1GeVpVeHRxd0pXeU9zbCtMNXI5RQpMMGlMQjFqWDhBZHFibEV3SWFDMGZLY1NzUkpYU2JmS3FFSVJSQmNBVURoY2EzTGIvejR5WWxtN29lb2kvaWxTCklGOWFrNjJBK1FJREFRQUJvMEl3UURBT0JnTlZIUThCQWY4RUJBTUNBcVF3RHdZRFZSMFRBUUgvQkFVd0F3RUIKL3pBZEJnTlZIUTRFRmdRVS9ZMS9TeERDMFBhbnRmV3NPeWh4OHp0ZURVa3dEUVlKS29aSWh2Y05BUUVMQlFBRApnZ0VCQUVOMlgyZTFSQWIvV1hud0ZNVE9ScjZ2RTJibzIxYXBXQkdSV25kblRVbnl6RmFXOGRUSG0yQ3VSZnNGCmhMbUdyU0tRMWVWQmc4TmhJdVdNbEpVRFZhZFFEWnpMR1NoWlFwUERuRW9uaGtYeTFCa013bkVxdUhsMWJ0d20KdUJlaDIrWDZEQXJxRlFDTTVwVW5qYTBtQ1ByN0M1VVM1dXBCZGFud1VZV1IwOUR3ODBjU21KUnpiRW0wQkZ1Two2L2Y1S1J1SCtBbWpWRlJjSXRWQkhTUHdXWXBhenBDeVBDWWRUUWZCMXZ2TEs0TndFa0JKT0tvandJaS9xQXVLCk9TRmZocnIxNStiRmhTZW5XalV5NDhUMk9wVDZzdkI0bGRpRXQ4Y1M5ZW8vWFVRdUdMVEtFYUdjN2VGQXVUY3EKcy82YkYxTTArVTdhRzZ3blJsY0Y0WG1mVmw4PQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCi0tLS0tQkVHSU4gQ0VSVElGSUNBVEUtLS0tLQpNSUlEVERDQ0FqU2dBd0lCQWdJSVNyWWYxRDBjc09Bd0RRWUpLb1pJaHZjTkFRRUxCUUF3UkRFU01CQUdBMVVFCkN4TUpiM0JsYm5Ob2FXWjBNUzR3TEFZRFZRUURFeVZyZFdKbExXRndhWE5sY25abGNpMXpaWEoyYVdObExXNWwKZEhkdmNtc3RjMmxuYm1WeU1CNFhEVEl6TURrd01UQXdNelkxTkZvWERUTXpNRGd5T1RBd016WTFORm93UkRFUwpNQkFHQTFVRUN4TUpiM0JsYm5Ob2FXWjBNUzR3TEFZRFZRUURFeVZyZFdKbExXRndhWE5sY25abGNpMXpaWEoyCmFXTmxMVzVsZEhkdmNtc3RjMmxuYm1WeU1JSUJJakFOQmdrcWhraUc5dzBCQVFFRkFBT0NBUThBTUlJQkNnS0MKQVFFQXhtbFdWcEdsek90UU5RZFM5cWVXTks5Sk96UUIrb0hLekxVenkwMk1EdEdIcDE0VnRzVnc0aTdEdGJNSApQNGEzK3IrU3UwaWZieDBBUkdQeGE2dHVJbVpOcVZQbnduS2hjejhRSjE1Z1hqeWdEWnRqZmFhQjVYYWdGT2lvCjFRSFEzTmxUWWw3THg3TEdyZjZkRGI5Sm42dnMrQWt0Ymp2eDQ3QmtQTEN2bWZpcHEzOWcrVVlGZWhqVXEwSGgKcnV3MURQMWJhZStKZlUzNE14OXl6aEQ1RlhNZTQvS29FOWZkSDdoN1F2dEdWMU16Zk12UlcrRU54MHpHa0g3ZAp6eTd6VDJFOXYya3IvWTM4bGcyYTFrVTc2U3BDWm9yTzQ4SW04NHFTTHBnUmtwS2JSbmt5VGh3UjIyWnhLc0NsCnlocnQ5WU4yYzRQUzlCbmZkdkt0bkI2Rk9RSURBUUFCbzBJd1FEQU9CZ05WSFE4QkFmOEVCQU1DQXFRd0R3WUQKVlIwVEFRSC9CQVV3QXdFQi96QWRCZ05WSFE0RUZnUVVZUksyTkdIMS9qZHN6QlJZWE5YREtMWjU0d0V3RFFZSgpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFLVjdJeVo4SGk1SXFGNVRNL0lHN2pTUWk4YW14RG1UVDhVUmRpWlNYejNnCnV0RkJBemZyOFBEWmlYRnY1VkY1TG94MzB3QisxTVA5UE9tSGtLNjllYnV5cGZUdUxPNGJIQkNWYzlIZEVmVHcKVVhQdVVKUmx3cGpJNXJTQitUc3BCNVV5WWVZekJkNkVrSDVZaFV2WXdPT3hjUlEwd1U3QWZUczE4RTd6Y1pSNwp5RUpESVpoQVJTT2h2elJlTUZxTFdKNDUvNmgxYzJTU1BFR0dlTFhhbkpXVDl2VkdnbHNTbjV0aWE3UThoeHl6CkQwWGRJVFpwWDZTRWtQM3dPMzBQMG9GWWljVHNJcktrWndtdENNZk5CSEczK2dIS29Za3FoWThNMGRKY2syemoKQlF4T1QwSVhScm93djVQS2VNM2x5ZDJWMndkOWlPcXBVQjdOTTlteGs1RT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQotLS0tLUJFR0lOIENFUlRJRklDQVRFLS0tLS0KTUlJRGx6Q0NBbitnQXdJQkFnSUlEbnloNTBUQ3RFa3dEUVlKS29aSWh2Y05BUUVMQlFBd1dURlhNRlVHQTFVRQpBd3hPYjNCbGJuTm9hV1owTFd0MVltVXRZWEJwYzJWeWRtVnlMVzl3WlhKaGRHOXlYMnh2WTJGc2FHOXpkQzF5ClpXTnZkbVZ5ZVMxelpYSjJhVzVuTFhOcFoyNWxja0F4Tmprek5UTTRNemc1TUI0WERUSXpNRGt3TVRBek1UazAKT1ZvWERUTXpNRGd5T1RBek1UazFNRm93V1RGWE1GVUdBMVVFQXd4T2IzQmxibk5vYVdaMExXdDFZbVV0WVhCcApjMlZ5ZG1WeUxXOXdaWEpoZEc5eVgyeHZZMkZzYUc5emRDMXlaV052ZG1WeWVTMXpaWEoyYVc1bkxYTnBaMjVsCmNrQXhOamt6TlRNNE16ZzVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQXdxVGoKY2NLZllOY1Vvb2RsbitIWEpWMEwrdmk3VWQ4ZlkvQnhFanZDVDYzV2xDUnVCQkxSSmZVWktvQkg4RklYYnlKYQphbDBUUkJseDRQU2hVbzhnSUpIQy9pcnEvbW5aOS9BaGNCTGQzSUxkSElzNlVJcnpOQzlnOUNJZkYrSkxiMmVsCldtZm8yWHE5dWhrTzlMbHgyYlVOTm9KVURENHIrQUhRTkFkd2RhVmZJT1V1VlVUNE1HVHM2ZEhEc3hOOEZQaVoKUXlPU3cyZ21MSTNmZFV3RXRPTGJ6Q2dwZXZlNlcrb0Fyblc0U1M5bDdXVWtMMGxaUDVNWjk3anZPWDVFZklJVApNb0ROcTZOLzd6OVdiZXVUY2hPZlhwMkp4RXExMEFmWEpkMXFjQU83cjVnY1FlTTNndmhOTGRBNWV0MDgxQkxlClRKT09CenhNVmRXUW5WV05qd0lEQVFBQm8yTXdZVEFPQmdOVkhROEJBZjhFQkFNQ0FxUXdEd1lEVlIwVEFRSC8KQkFVd0F3RUIvekFkQmdOVkhRNEVGZ1FVd2pVQ2FkSnFyT2w5VlVlYWkxUVE5Rkc4VUVNd0h3WURWUjBqQkJndwpGb0FVd2pVQ2FkSnFyT2w5VlVlYWkxUVE5Rkc4VUVNd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFCN2V2L0p5Clk3cWUzNHVwZHFKT3BKdkVMVEVXcGgwaWRnUE02Z3lOeVdkWHg3ME0raDhXUEgranNBamFGRXBsZEFBTDMxbVoKSU1HTTFMaXdmd3AyODB6aHd4MjVnNDZNMUxvNjhKd1NZdklkN2pIUVRqQXVPY2c3Q3pQTzdVQ0MxdDd0SlVwVQoyNHRoMzY5QmRiM3I2KzdTaEJIR0VMZnR0Njg1cFFPY2U4cWxpV2ZHUWVhNVlQclJ2Y2cyRXZ4YkpxRVVBYVlsCnlxZzd0dGVBMHhoZGVoaWJOTHRaNUZRRVIxbDNLU1FwZHlnc0J6UEorOWppbHdKRVRuY052cGg3NlR3T24zUmYKR0t5TjB4Z2lJZ3ZrLytrRU5wend5ekxWbWNSVlc2c20yQU4wL2VXaExadDFBeU9CTnlOdDEvL29VZVhXVmdvdwpob3h0WWF0WGNUVFg2Rjg9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0KLS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURjekNDQWx1Z0F3SUJBZ0lJT213SGtZREJ3b0l3RFFZSktvWklodmNOQVFFTEJRQXdKakVrTUNJR0ExVUUKQXd3YmFXNW5jbVZ6Y3kxdmNHVnlZWFJ2Y2tBeE5qa3pOVE00TlRNd01CNFhEVEl6TURrd01UQXpNakl4TUZvWApEVEkxTURnek1UQXpNakl4TVZvd0tURW5NQ1VHQTFVRUF3d2VLaTVoY0hCekxtOXJaRFF1YTNSa1pXMXZMbVIxClkydGtibk11YjNKbk1JSUJJakFOQmdrcWhraUc5dzBCQVFFRkFBT0NBUThBTUlJQkNnS0NBUUVBeHlOR3lUR0IKVCtaSnUrRGhoTU9wdWh5OGtETTJiRTlTaWRsK1VuU1lOMHVnSDErZDBub2J0OW5rMUc5QXhkMVZtOFRQUFBPago4TS9OUzYvOGxvQ0R4d1hmbUY0cmRkK1A0b3ArVXl1eHN0SEhpcmd3eUlSRHUvcjVjeXV5SVBkTFVDUDNGb3JnCnBWbnRoUW45alpKWWM1Z2JqdktxYTd5TmlYMEZnWmlOMXdrTExndS8xN0t5QzJYSVV3V2FsbEVQbERvcFh0MjUKYUMwUDRKMjRGeU5XdGtXMU8zS3MxSmJ3emVZOVBhSzZ5c212OElQdFlTbWJvTVZFVzRsVTVlamJaNnhWbFl5SwpMQjhBcmdMMGcwNXFsTUk2dDJiRkdUUWtHODFLNEhBeWZYNVh4NW9WdklPU05lWEhoSjN2UW9iYkpPY2FNUXZMCnRiRUFKUi9JTHNuMzN3SURBUUFCbzRHaE1JR2VNQTRHQTFVZER3RUIvd1FFQXdJRm9EQVRCZ05WSFNVRUREQUsKQmdnckJnRUZCUWNEQVRBTUJnTlZIUk1CQWY4RUFqQUFNQjBHQTFVZERnUVdCQlN2dWVESnNDUXBDdDFNYjFxbQozUk14bDcrUGFEQWZCZ05WSFNNRUdEQVdnQlROenZNSDdnTjM1ek10TGljOEJqM2VKTzZIRURBcEJnTlZIUkVFCklqQWdnaDRxTG1Gd2NITXViMnRrTkM1cmRHUmxiVzh1WkhWamEyUnVjeTV2Y21jd0RRWUpLb1pJaHZjTkFRRUwKQlFBRGdnRUJBR09qa2lNUjZQSFh6QncwSWJKSWd2L2Mva2lSNnlIYXFncS80VW5oRW5vNlNFd0I1OU8xR25qRQpuQkd2MUNCdDFKUStHRDFLMUcweXFkVEVNM1pRbGNaM2ZqYTdOc3JGR1YycWlJb1Q2TVFuTUdRR214Yk85NGtFCmoyU0d3d0JjRWErRExTZ0ZEYTRlYVlPSmNQR0R2bXRIVWtycXZLMGpFaUoxdVpiTi81M25jN2FWQ0pMbFpCaWEKRzdseDBwbTZNTytWOE9VZU9IY1A1KzlrQm1RYVlWTkFhaFJOUjlFdUtCcFVFZXJaZDEycDRMbkVpbDVDS0JCeApWSWtmbjkzTHl5cUFwVC9ESWhZaHUvU3NxZUxSazdZRVpQcFdpMVU4c1FOUEEwYzlMRVMzQkxsSFJPOTN0RmhYClp3TnBvOWF3bFFXcUdYdkUvcTJEcVliZW1JTHZiYlk9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0KLS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURERENDQWZTZ0F3SUJBZ0lCQVRBTkJna3Foa2lHOXcwQkFRc0ZBREFtTVNRd0lnWURWUVFEREJ0cGJtZHkKWlhOekxXOXdaWEpoZEc5eVFERTJPVE0xTXpnMU16QXdIaGNOTWpNd09UQXhNRE15TWpBNVdoY05NalV3T0RNeApNRE15TWpFd1dqQW1NU1F3SWdZRFZRUUREQnRwYm1keVpYTnpMVzl3WlhKaGRHOXlRREUyT1RNMU16ZzFNekF3CmdnRWlNQTBHQ1NxR1NJYjNEUUVCQVFVQUE0SUJEd0F3Z2dFS0FvSUJBUURDOGw2azdLZzNFQTVYL05KN0tVRmYKY0xLV0YyNTAza0RIRlRQOWVBMlJOcFFOWmt1TG1CdG5CK0hlS2ozZndTc2ZaZWRQUkt6Uk4wUDFFMmFqZFBsdgplMm5GU3U4SkIvMFpselFlMTFObWFEd0kyYmJ3Qk1yZFc3M1R1dGxTa2NlVllxV1IwZnZBT1FCbU9xc3RIdm84CmZUZWtmMHFqVndueVlwMUIxaWZCMEdEYzhUeUxiYlJUYWtkcVBoMG1nNFlmRDlNTk5lZWNUSDRVWUFhbDVOVWYKZ2lOTkxHdk55RHNyZE0wcXFYSGlWVEdGeWU0dXY3ZFFqdDg5c0RWd1FhNU85Mk9TQVZlZW42UFVTVE9RdnZqYQpaeVpuTzFoZUZMZGRCQmJWa3VtTUw1ZitZZlFhaGllVnVPOVB1WjU3Y0drdkFFUnZwc05IbUVrY05hOCtpZm5iCkFnTUJBQUdqUlRCRE1BNEdBMVVkRHdFQi93UUVBd0lDcERBU0JnTlZIUk1CQWY4RUNEQUdBUUgvQWdFQU1CMEcKQTFVZERnUVdCQlROenZNSDdnTjM1ek10TGljOEJqM2VKTzZIRURBTkJna3Foa2lHOXcwQkFRc0ZBQU9DQVFFQQpJOUlYckFzZTdkL01oZVV6U3BxYmNCOXl1ZE1xZzV2aDhNZ3VXS0sydEZvZDB0YVFCR2s3c3JWSElSOVY5OEFYClNyRXZiUSt6NEJCY3RXeFB4V3lJTGtEbU5RL3hlc2kvaFhyUVc1Q3d0ekZuWUIrNVdJRElJZFc3amt1M1dzUGwKZnRQcitPTTltS2VwYUFZUFpoUjhKUzBWWktzS3pmUVV0Q1hVUjFtakZ4RUV6VG1vQWlWZ1hrMEpyM01SeGFVSgpLRXZsamZ6WFpJSkdTVUdxVmtxMkNPUDV4ZXBNU0hpRmsxV05pdDVqTTVCNU1XckxDTzNidFl2dVBEdFEybXFWCmFsZlE4Vk9mazM4RDcyUnh2TllGNkNZdFJjQlBpZzJzUW5LYW9LMUFaYmVGWDB6cnFQNGR5SmRGZmhQRHU2OXMKR0hmWlZIeFN6cytlN0xBL0hPS3Ewdz09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
    server: https://api-int.okd4.ktdemo.duckdns.org:6443
  name: local
contexts:
- context:
    cluster: local
    user: kubelet
  name: kubelet
current-context: kubelet
preferences: {}
users:
- name: kubelet
  user:
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6IkJnWXp6Q1lqaW5qSjFvbDdaNWFkbFBYOHQ5WEtpQURNRFJYT3lQTkNJNzQifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJvcGVuc2hpZnQtbWFjaGluZS1jb25maWctb3BlcmF0b3IiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlY3JldC5uYW1lIjoibm9kZS1ib290c3RyYXBwZXItdG9rZW4iLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoibm9kZS1ib290c3RyYXBwZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI3ODg2NzEyMy00YTVlLTQ3MTctODgxOS0yNDFhMGE1ZjYyZTciLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6b3BlbnNoaWZ0LW1hY2hpbmUtY29uZmlnLW9wZXJhdG9yOm5vZGUtYm9vdHN0cmFwcGVyIn0.TSHUaDVCMQw6IBLBTjb6LIeGPDaovaVa2v4thV9S1NrddNYM6jjrGWO-9Gigh0Kz0Ra48DhLdMMXvv7tVDJLEI3RJ3UgqdVI967bzjX95U_azCHxJf1eiE5KiAcAirZI7LvWrtqCY5R85bTNryTifS_MZJyGvB_I-7IUVtXqg1Dk_Iid7GwWk1ayLqpOEsrVgOGMgejjgnXeEDWGRVsDaiqe9h4A76N5JmsJeqvJn3jjc3w5mupDDaXLn2lphBbz5UubJFALSGTZH9TvmaxjvFVb9wvuhTHcqv3gurJ6gVdjSuA0FV81IT43wp_FfCA3IK9irIHvs6jEsCu7lp9o9Q
```  


<br/>

```bash
[root@okd-4 kubernetes]# systemctl status kubelet
● kubelet.service - Kubernetes Kubelet
     Loaded: loaded (/etc/systemd/system/kubelet.service; enabled; preset: disabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─01-kubens.conf, 10-mco-default-env.conf, 10-mco-default-madv.conf, 20-logging.conf, 20-nodenet.conf
     Active: active (running) since Mon 2023-09-18 05:32:19 UTC; 37min ago
   Main PID: 1769 (kubelet)
      Tasks: 20 (limit: 19051)
     Memory: 57.6M
        CPU: 59.059s
     CGroup: /system.slice/kubelet.service
             └─1769 /usr/bin/kubelet --config=/etc/kubernetes/kubelet.conf --bootstrap-kubeconfig=/etc/kubernetes/kubeconfig --kubeconfig=/var/lib/kubelet/kubeconfig --container-runti>
```   


<br/>

```bash
[root@okd-4 kubernetes]# cd /etc/systemd/system/kubelet.service.d
[root@okd-4 kubelet.service.d]# ls
01-kubens.conf  10-mco-default-env.conf  10-mco-default-madv.conf  20-logging.conf  20-nodenet.conf
```  

<br/>

master node 에서  


참고 : https://kubernetes.io/ko/docs/tasks/administer-cluster/access-cluster-api/

<br/>

```bash
[root@bastion ~]# kubectl config view -o jsonpath='{"Cluster name\tServer\n"}{range .clusters[*]}{.name}{"\t"}{.cluster.server}{"\n"}{end}'
Cluster name	Server
api-okd4-ktdemo-duckdns-org:6443	https://api.okd4.ktdemo.duckdns.org:6443
okd4	https://api.okd4.ktdemo.duckdns.org:6443

[root@bastion ~]# export CLUSTER_NAME="okd4"

APISERVER=$(kubectl config view -o jsonpath="{.clusters[?(@.name==\"$CLUSTER_NAME\")].cluster.server}")
[root@bastion ~]# APISERVER=$(kubectl config view -o jsonpath="{.clusters[?(@.name==\"$CLUSTER_NAME\")].cluster.server}")
[root@bastion ~]# echo $APISERVER
https://api.okd4.ktdemo.duckdns.org:6443
```  

<br/>

# 기본 서비스 어카운트용 토큰을 보관할 시크릿을 생성한다.
kubectl -n default apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: default-token
  annotations:
    kubernetes.io/service-account.name: default
type: kubernetes.io/service-account-token
EOF


```bash
[root@bastion ~]# kubectl -n default apply -f  - <<EOF
 apiVersion: v1
 kind: Secret
 metadata:
   name: default-token
   annotations:
     kubernetes.io/service-account.name: default
 type: kubernetes.io/service-account-token
 EOF
secret/default-token created
```  

<br/>

```bash
[root@bastion ~]# TOKEN=$(kubectl get secret default-token -o jsonpath='{.data.token}' -n default | base64 --decode)
```  

<br/>

```bash
[root@bastion ~]# curl -X GET $APISERVER/api --header "Authorization: Bearer $TOKEN" --insecure
{
  "kind": "APIVersions",
  "versions": [
    "v1"
  ],
  "serverAddressByClientCIDRs": [
    {
      "clientCIDR": "0.0.0.0/0",
      "serverAddress": "192.168.1.146:6443"
    }
  ]
}
```

```bash
[root@bastion ~]# curl -X GET $APISERVER/api/v1/namespaces/default/pods --header "Authorization: Bearer $TOKEN" --insecure
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "Unauthorized",
  "reason": "Unauthorized",
  "code": 401
}
```  

<br/>

default-token의 값으로 다시 설정하고 테스트.  

```bash  
[root@bastion ~]# curl -X GET $APISERVER/api/v1/namespaces/default/pods --header "Authorization: Bearer $TOKEN" --insecure
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "pods is forbidden: User \"system:serviceaccount:default:default\" cannot list resource \"pods\" in API group \"\" in the namespace \"default\"",
  "reason": "Forbidden",
  "details": {
    "kind": "pods"
  },
  "code": 403
}
```  


<br/>

```bash
[root@bastion security]# vi pod-list-role.yaml
[root@bastion security]# vi pod-list-rolebinding.yaml
[root@bastion security]# kubectl apply -f pod-list-role.yaml
role.rbac.authorization.k8s.io/pod-list-role created
[root@bastion security]# kubectl apply -f pod-list-rolebinding.yaml
rolebinding.rbac.authorization.k8s.io/pod-list-rolebinding created
[root@bastion security]# curl -X GET $APISERVER/api/v1/namespaces/default/pods --header "Authorization: Bearer $TOKEN" --insecure
{
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {
    "resourceVersion": "107346"
  },
  "items": []
```  


kubectl create clusterrolebinding default-cluster-admin --clusterrole cluster-admin --serviceaccount default:default

oc adm policy add-scc-to-user anyuid system:serviceaccount:default:default

<br/>

https://jangcenter.tistory.com/123
https://velog.io/@ehdrms2034/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-RBAC-%EA%B8%B0%EB%B0%98-ServiceAccount

[k8s] 사용자 전체 권한 확인 - 1

https://passwd.tistory.com/entry/k8s-%ED%98%84%EC%9E%AC-%EC%82%AC%EC%9A%A9%EC%9E%90-%EC%A0%84%EC%B2%B4-%EA%B6%8C%ED%95%9C-%ED%99%95%EC%9D%B8

pod-list-role.yaml
```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-list-role
  namespace: default
rules:
- apiGroups: [""]
  verbs: ["get", "list"]
  resources: ["pods"]
```  

<br/>

pod-list-rolebinding.yaml
```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-list-rolebinding
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-list-role
subjects:
- kind: ServiceAccount
  name: default
  namespace: default
```



<br/>


```bash
[root@bastion security]# kubectl auth can-i list pods  --as system:serviceaccount:default:default -n default
yes
[root@bastion security]# kubectl auth can-i get pods  --as system:serviceaccount:default:default -n default
yes
```  


<br/>




```bash
[root@okd-1 static-pod-resources]# pwd
/etc/kubernetes/static-pod-resources
[root@okd-1 static-pod-resources]# ls
configmaps  etcd-pod-4            kube-apiserver-last-known-good  kube-apiserver-pod-5  kube-controller-manager-certs  kube-controller-manager-pod-7  kube-scheduler-pod-6
etcd-certs  etcd-pod-5            kube-apiserver-pod-2            kube-apiserver-pod-6  kube-controller-manager-pod-5  kube-scheduler-certs           kube-scheduler-pod-7
etcd-pod-2  kube-apiserver-certs  kube-apiserver-pod-4            kube-apiserver-pod-7  kube-controller-manager-pod-6  kube-scheduler-pod-4           secrets
```  




helm repo add kubescape https://kubescape.github.io/helm-charts/ ; helm repo update ; helm upgrade --install kubescape kubescape/kubescape-operator -n kubescape --create-namespace --set clusterName=`kubectl config current-context` --set account=3054be31-a882-4c7e-a223-5b79d9e37157 --set server=api.armosec.io


```bash
[root@bastion /]# oc new-project kubescape
Now using project "kubescape" on server "https://api.okd4.ktdemo.duckdns.org:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=k8s.gcr.io/e2e-test-images/agnhost:2.33 -- /agnhost serve-hostname

[root@bastion /]# oc adm policy add-scc-to-user anyuid -z default -n kubescape
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:anyuid added: "default"
[root@bastion /]# oc adm policy add-scc-to-user privileged -z default -n kubescape
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:privileged added: "default"
```  

<br/>

```bash
[root@bastion /]# helm upgrade --install kubescape kubescape/kubescape-operator -n kubescape --set clusterName=`kubectl config current-context` --set account=3054be31-a882-4c7e-a223-5b79d9e37157 --set server=api.armosec.io
WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /root/okd4/auth/kubeconfig
Release "kubescape" has been upgraded. Happy Helming!
NAME: kubescape
LAST DEPLOYED: Tue Sep 19 12:37:32 2023
NAMESPACE: kubescape
STATUS: deployed
REVISION: 3
TEST SUITE: None
NOTES:
Thank you for installing kubescape-operator version 1.15.3.

You can see and change the values of your's recurring configurations daily scan in the following link:
https://cloud.armosec.io/settings/assets/clusters/scheduled-scans?cluster=kubescape-api-okd4-ktdemo-duckdns-org-6443-root
> kubectl -n kubescape get cj kubescape-scheduler -o=jsonpath='{.metadata.name}{"\t"}{.spec.schedule}{"\n"}'

You can see and change the values of your's recurring images daily scan in the following link:
https://cloud.armosec.io/settings/assets/images
> kubectl -n kubescape get cj kubevuln-scheduler -o=jsonpath='{.metadata.name}{"\t"}{.spec.schedule}{"\n"}'

See you!!!
[root@bastion /]# kubectl get po -n kubescrap
No resources found in kubescrap namespace.
[root@bastion /]# kubectl get po -n kubescape
NAME                             READY   STATUS    RESTARTS   AGE
gateway-8497d7696f-frmz8         1/1     Running   0          2m25s
nginx                            1/1     Running   0          3m31s
otel-collector-b98c5854f-tzpk4   1/1     Running   0          2m25s
```  


<br/>

```bash
[root@bastion dynamic_provisioning]# oc adm policy add-scc-to-user privileged -z storage -n kubescape
clusterrole.rbac.authorization.k8s.io/system:openshift:scc:privileged added: "storage"
```   

<br/>

```bash
[root@bastion ~]# kubectl run nginx --image=nginx -n default
Warning: would violate PodSecurity "restricted:latest": allowPrivilegeEscalation != false (container "nginx" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "nginx" must set securityContext.capabilities.drop=["ALL"]), runAsNonRoot != true (pod or container "nginx" must set securityContext.runAsNonRoot=true), seccompProfile (pod or container "nginx" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
pod/nginx created
```  

<br/>

```bash
[root@bastion ~]# kubectl edit namespace default -n default
apiVersion: v1
kind: Namespace
metadata:
  annotations:
    openshift.io/sa.scc.mcs: s0:c5,c0
    openshift.io/sa.scc.supplemental-groups: 1000020000/10000
    openshift.io/sa.scc.uid-range: 1000020000/10000
  creationTimestamp: "2023-09-01T03:07:17Z"
  labels:
    kubernetes.io/metadata.name: default
  name: default
  resourceVersion: "471"
  uid: 5df4c86f-47f9-497b-b172-cae76d79b31b
spec:
  finalizers:
  - kubernetes
status:
  phase: Active
[root@bastion ~]# kubectl edit namespace shclub -n shclub
apiVersion: v1
kind: Namespace
metadata:
  annotations:
    openshift.io/description: ""
    openshift.io/display-name: ""
    openshift.io/node-selector: devops=true
    openshift.io/requester: system:admin
    openshift.io/sa.scc.mcs: s0:c26,c15
    openshift.io/sa.scc.supplemental-groups: 1000680000/10000
    openshift.io/sa.scc.uid-range: 1000680000/10000
  creationTimestamp: "2023-09-01T04:03:58Z"
  labels:
    kubernetes.io/metadata.name: shclub
    pod-security.kubernetes.io/audit: baseline
    pod-security.kubernetes.io/audit-version: v1.24
    pod-security.kubernetes.io/warn: baseline
    pod-security.kubernetes.io/warn-version: v1.24
  name: shclub
  resourceVersion: "367807"
  uid: af28fd79-eb32-4aa6-bcc9-3a368b51271d
spec:
  finalizers:
  - kubernetes
status:
  phase: Active
```