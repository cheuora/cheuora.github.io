---
layout: post
title: multipass에서 고정ip사용하기
tags: [multipass,VM]
---

#### 서론

예전에는 윈도우즈에서 우분투를 사용하려면 WSL을 사용했었는데 생각보다 좀 불편했다. 왜 그런가 하면..

* 네트웍 문제.. : 호스트 PC와 동일한 IP를 사용하기 떄문에 별도 서버를 상정한 테스트를 하기가 어렵다.
* 2대 이상의 우분투 설치가 어려움 

그래서 이의 훌륭한 대안이 [multipass](https://multipass.run/) 이다. OS가 우분투만 된다는 점만 빼면 복수개의 OS를
설치하고 복수개의 내부 IP를 쓸 수 있다는 점이 좋다.

그런데 기본적으로 multipass의 우분투는 DHCP를 사용한다. 다시 말해 서버를 stop했다가
start하면 처음 할당되었던 IP가 바뀌어 버린다. 이를 고정 IP로 쓸 수 있도록 설정하는 방법을 찾
아 여기에 기록한다.

---


multipass의 호스트PC는 기본적으로 각 우분투 OS내 eth0 를 통해 통신을 하는 것 같다(이 부분은 나도 확실치 않다. 그냥 나의 추측일 뿐...). 각우분투 내의 eth0네트워크를 건드리면 호스트에서 접속이 불가능해지는 현상이 발생한다.
그래서 우분투 내에 eth1이라는 새로운 네트워크 어뎁터를 설정하는데, 이 어뎁터의 게이트웨이를 호스트 PC의 가상 네트워크 스위치에 맞춘다. (여기에서는 Windowns의 HyperV를 사용한다) 

#### HyperV에서 가상 스위치 생성

* 가상 스위치를 만든다 여기서는 multipass 라는 이름으로 만들었다. 
* 이 때 스위치는 '외부 네트워크' 로 잡는다. 

![](https://raw.githubusercontent.com/cheuora/cheuora.github.io/master/_posts/2023/20230428.png)

#### 호스트PC multipass에서 우분투 설치 

호스트 PC에서 multipass로 우분투를 설치한다.

```
> multipass launch -n ubuntu_test -c 2 -m 2G -d 10G --network name=multipass,mode=manual
```

-n ubuntu_test: ubuntu_test을 이름으로 우분투 서버를 설치(여러분의 서버 이름 
-c 2 : 2CPU
-m 2G : 2G메모리
-d 10G: 10G 디스크
--network=multipass,mode=manual : multipass네트워크, 수동 모드로 설치

설치가 끝나면 ubuntu_test에 접속함 

```
> multipass shell ubuntu_test
```

#### 우분투 설정 파일 변경

ubuntu_test에 접속하여 /etc/netplan 디렉토리로 가면 어떤 yaml파일이 보일 것이다.. 이를 `sudo vi`로 열어보면 다음과 비슷할 것이다. 

```
network:
    ethernets:
        eth0:
            dhcp4: true
            match:
                macaddress: 52:54:00:f1:f0:e8
            set-name: eth0
    version: 2
```
여기에 eth1을 추가한다. 

```
network:
    ethernets:
        eth0:
            dhcp4: true
            match:
                macaddress: 52:54:00:f1:f0:e8
            set-name: eth0
        eth1:
            addresses: [192.168.0.33/24]
            routes:
              - to: default
                via: 192.168.0.1
            nameservers:
                addresses: [8.8.8.8, 1.1.1.1]
    version: 2
 ```
 
 * addresses : 사용하고자 하는 IP/NetMask. IP는 HOSTPC에서 접근가능해야 한다. 
 * route via : HOSTPC가 사용하는 Gateway
 * nameservers : 사용할 DNS 

수정 후에 `sudo netplan apply`를 실행하면 반영이 되며, 이후 ifconfig등으로 eth1이 추가 반영된 것을 볼 수 있다. 

```
ubuntu@:/etc/netplan$ ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.26.206.76  netmask 255.255.240.0  broadcast 172.26.207.255
        inet6 fe80::5054:ff:fef1:f0e8  prefixlen 64  scopeid 0x20<link>
        ether 52:54:00:f1:f0:e8  txqueuelen 1000  (Ethernet)
        RX packets 2741  bytes 459553 (459.5 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 897  bytes 138028 (138.0 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.33  netmask 255.255.255.0  broadcast 192.168.0.255
        inet6 fe80::5054:ff:fe87:5e23  prefixlen 64  scopeid 0x20<link>
        ether 52:54:00:87:5e:23  txqueuelen 1000  (Ethernet)
        RX packets 64965  bytes 90553308 (90.5 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 12142  bytes 934644 (934.6 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

고정 IP 테스트는  호스트 PC에서 설정한 IP로 ping등을 날려 보면 된다. 

```
PS C:\Users\S> ping 192.168.0.33

Ping 192.168.0.33 32바이트 데이터 사용:
192.168.0.33의 응답: 바이트=32 시간<1ms TTL=64
192.168.0.33의 응답: 바이트=32 시간<1ms TTL=64
192.168.0.33의 응답: 바이트=32 시간<1ms TTL=64
192.168.0.33의 응답: 바이트=32 시간<1ms TTL=64

192.168.0.33에 대한 Ping 통계:
    패킷: 보냄 = 4, 받음 = 4, 손실 = 0 (0% 손실),
왕복 시간(밀리초):
    최소 = 0ms, 최대 = 0ms, 평균 = 0ms
```


