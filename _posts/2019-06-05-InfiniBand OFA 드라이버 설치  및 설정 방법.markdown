---
layout: post
title:  "InfiniBand OFA 드라이버 설치  및 설정 방법"
date:   2019-06-05 02:28:00 +0900
categories: Linux InfiniBand
---
이번에 InfiniBand 장비를 가지고 프로젝트를 진행해야 하는 일이 있어서 Open Fabric Allience에서 제공하는 드라이버 설치와 설정하는 과정을 남겨두려고 한다. 본 글에서는 InfiniBand OFA 드라이버 설치, InfiniBand 장비 설정 그리고 벤치마킹을 통해서 정상적으로 동작하는지 알아볼 예정이다. 

본 글은 <a href="https://community.mellanox.com/s/article/howto-install-mlnx-ofed-driver">https://community.mellanox.com/s/article/howto-install-mlnx-ofed-driver</a> 에서 제공하는 문서를 기반으로 작성되었으며, 본인이 겪은 문제와 이를 해결하기 위한 내용을 추가로 담았다.

<h2>시스템 사양</h2>

구축한 시스템 사양은 다음과 같다. <br>CPU : i7 8700<br>RAM: 32G <br>InfiniBand: Mellanox ConnectX-3 FDR<br>OS: CentOS 7.6<br><br>동일한 사양의 컴퓨터 두 대를 가지고 InfiniBand를 1:1 직렬(Direct)로 연결하였다. 

<h2>OFA 드라이버 설치</h2>


설치한 InfiniBand 장비가 시스템에서 제대로 인식하고 있는지 다음과 같은 명령어로 확인할 수 있다.  LnkCap와 LnkSta 항목에서 Speed와 Width가 동일한 수치를 가지고 있다면, 정상적으로 시스템에서 올바른 속도로 인식하고 있음을 알 수 있다.

```
$ sudo lspci -vvv
02:00.0 Network controller: Mellanox Technologies MT27500 Family [ConnectX-3]
	Subsystem: Mellanox Technologies Device 0051
	Control: I/O- Mem+ BusMaster+ SpecCycle- MemWINV- VGASnoop- ParErr- Stepping- SERR- FastB2B- DisINTx+
	Status: Cap+ 66MHz- UDF- FastB2B- ParErr- DEVSEL=fast >TAbort- &lt;TAbort- &lt;MAbort- >SERR- &lt;PERR- INTx-
	Latency: 0, Cache Line Size: 64 bytes
	Interrupt: pin A routed to IRQ 17
	Region 0: Memory at df200000 (64-bit, non-prefetchable) [size=1M]
	Region 2: Memory at d2800000 (64-bit, prefetchable) [size=8M]
	Expansion ROM at df100000 [disabled] [size=1M]
	Capabilities: [40] Power Management version 3
		Flags: PMEClk- DSI- D1- D2- AuxCurrent=0mA PME(D0-,D1-,D2-,D3hot-,D3cold-)
		Status: D0 NoSoftRst+ PME-Enable- DSel=0 DScale=0 PME-
	Capabilities: [48] Vital Product Data
		Product Name: CX353A - ConnectX-3 QSFP
		Read-only fields:
			[PN] Part number: MCX353A-FCBT         
			[EC] Engineering changes: A3
			[SN] Serial number: MT##########            
			[V0] Vendor specific: PCIe Gen3 x8    
			[RV] Reserved: checksum good, 0 byte(s) reserved
		Read/write fields:
			[V1] Vendor specific: N/A   
			[YA] Asset tag: N/A                     
			[RW] Read-write area: 105 byte(s) free
			[RW] Read-write area: 253 byte(s) free
			[RW] Read-write area: 253 byte(s) free
			[RW] Read-write area: 253 byte(s) free
			[RW] Read-write area: 253 byte(s) free
			[RW] Read-write area: 253 byte(s) free
			[RW] Read-write area: 253 byte(s) free
			[RW] Read-write area: 253 byte(s) free
			[RW] Read-write area: 253 byte(s) free
			[RW] Read-write area: 253 byte(s) free
			[RW] Read-write area: 253 byte(s) free
			[RW] Read-write area: 253 byte(s) free
			[RW] Read-write area: 253 byte(s) free
			[RW] Read-write area: 253 byte(s) free
			[RW] Read-write area: 253 byte(s) free
			[RW] Read-write area: 252 byte(s) free
		End
	Capabilities: [9c] MSI-X: Enable+ Count=128 Masked-
		Vector table: BAR=0 offset=0007c000
		PBA: BAR=0 offset=0007d000
	Capabilities: [60] Express (v2) Endpoint, MSI 00
		DevCap:	MaxPayload 512 bytes, PhantFunc 0, Latency L0s &lt;64ns, L1 unlimited
			ExtTag- AttnBtn- AttnInd- PwrInd- RBE+ FLReset- SlotPowerLimit 116.000W
		DevCtl:	Report errors: Correctable- Non-Fatal- Fatal- Unsupported-
			RlxdOrd- ExtTag- PhantFunc- AuxPwr- NoSnoop-
			MaxPayload 256 bytes, MaxReadReq 512 bytes
		DevSta:	CorrErr+ UncorrErr- FatalErr- UnsuppReq+ AuxPwr- TransPend-
		LnkCap:	Port #8, Speed 8GT/s, Width x8, ASPM L0s, Exit Latency L0s unlimited, L1 unlimited
			ClockPM- Surprise- LLActRep- BwNot- ASPMOptComp+
		LnkCtl:	ASPM Disabled; RCB 64 bytes Disabled- CommClk+
			ExtSynch- ClockPM- AutWidDis- BWInt- AutBWInt-
		LnkSta:	Speed 8GT/s, Width x8, TrErr- Train- SlotClk+ DLActive- BWMgmt- ABWMgmt-
		DevCap2: Completion Timeout: Range ABCD, TimeoutDis+, LTR-, OBFF Not Supported
		DevCtl2: Completion Timeout: 50us to 50ms, TimeoutDis-, LTR-, OBFF Disabled
		LnkCtl2: Target Link Speed: 8GT/s, EnterCompliance- SpeedDis-
			 Transmit Margin: Normal Operating Range, EnterModifiedCompliance- ComplianceSOS-
			 Compliance De-emphasis: -6dB
		LnkSta2: Current De-emphasis Level: -6dB, EqualizationComplete+, EqualizationPhase1+
			 EqualizationPhase2+, EqualizationPhase3+, LinkEqualizationRequest-
	Capabilities: [c0] Vendor Specific Information: Len=18 &lt;?>
	Capabilities: [100 v1] Alternative Routing-ID Interpretation (ARI)
		ARICap:	MFVC- ACS-, Next Function: 0
		ARICtl:	MFVC- ACS-, Function Group: 0
	Capabilities: [148 v1] Device Serial Number 00-02-c9-03-00-3d-56-f0
	Capabilities: [154 v2] Advanced Error Reporting
		UESta:	DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
		UEMsk:	DLP- SDES- TLP- FCP- CmpltTO- CmpltAbrt- UnxCmplt- RxOF- MalfTLP- ECRC- UnsupReq- ACSViol-
		UESvrt:	DLP+ SDES- TLP- FCP+ CmpltTO- CmpltAbrt- UnxCmplt- RxOF+ MalfTLP+ ECRC- UnsupReq- ACSViol-
		CESta:	RxErr- BadTLP- BadDLLP- Rollover- Timeout- NonFatalErr-
		CEMsk:	RxErr- BadTLP- BadDLLP- Rollover- Timeout- NonFatalErr+
		AERCap:	First Error Pointer: 00, GenCap+ CGenEn- ChkCap+ ChkEn-
	Capabilities: [18c v1] #19
	Kernel driver in use: mlx4_core
	Kernel modules: mlx4_core
```


MLNX_OFED를 wget으로 다음과 같이 다운로드 받는다. 본 글의 환경은 CentOS 7.6 이므로 동일한 환경일 경우 다음과 같이 커맨드를 입력하면 된다. 글을 쓰는 시점으로 MLNX_OFED-4.5-1.0.1.0 버전이다. (추후 업데이트가 될 수 있으므로 <a href="http://www.mellanox.com/page/products_dyn?product_family=26&amp;mtag=linux_sw_drivers">http://www.mellanox.com/page/products_dyn?product_family=26&amp;mtag=linux_sw_drivers</a> 에서 버전을 확인할 수 있다)<br>

```
$ wget http://www.mellanox.com/downloads/ofed/MLNX_OFED-4.5-1.0.1.0/MLNX_OFED_LINUX-4.5-1.0.1.0-rhel7.6-x86_64.tgz
$ tar xvf MLNX_OFED_LINUX-4.5-1.0.1.0-rhel7.6-x86_64.tgz 
$ cd MLNX_OFED_LINUX-4.5-1.0.1.0-rhel7.6-x86_64
$ sudo ./mlnxofedinstall 
$ sudo /etc/init.d/openibd restart</code></pre>
```


위와 같이 커맨드를 입력하게 되면, MLNX_OFED 설치가 완료 된다. 마지막줄은 Driver를 재시작하는 명령이다. 


모든 노드에 동일한 방식으로 드라이버를 설치하면 된다. 어렵지 않은 과정으로 설치를 끝낼 수 있다.


<h2>시스템 설정 및 트러블 슈팅</h2>



InfiniBand Interface에 IP Address를 할당하는 과정이다. 주의할 점은 각 IB 링크는 개별적인 서브넷 주소를 가지고 있어야 한다. ifconfig 명령어를 통해서 InfiniBand Interface 이름을 알아본다. 이후 /etc/sysconfig/network-scripts/ifcfg-ibx 파일을 수정한다. 다음 설정 파일은 본 시스템에서 설정한 ifcfg-ib0 파일이다. UUID는 #으로 숨김처리 하였다. 

```
CONNECTED_MODE=no
TYPE=InfiniBand
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ib0
UUID=########-####-####-####-############
DEVICE=ib0
ONBOOT=yes
BOOTPROTO=static
IPADDR=192.168.1.20
NETMASK=255.255.255.0
NETWORK-192.168.1.0
BROADCAST=192.168.1.255
NM_CONTROLLED=no
```


설정파일을 수정하고 네트워크 데몬을 다시 시작한다.

```
$ sudo systemctl restart network
```


제대로 설정이 되었는지 확인을 위하여 ibstat 명령어로 확인한다. 여기서 문제가 발생하는데, State가 Initializing 상태에서 멈춰있고 노드간 제대로 통신이 되고 있지 않다는 것을 알 수 있다. 

```
[jkpark@node1 ~]$ ibstat
CA 'mlx4_0'
	CA type: MT4099
	Number of ports: 1
	Firmware version: 2.42.5000
	Hardware version: 1
	Node GUID: 0x################
	System image GUID: 0x################
	Port 1:
		State: Initializing
		Physical state: LinkUp
		Rate: 56
		Base lid: 1
		LMC: 0
		SM lid: 2
		Capability mask: 0x02514868
		Port GUID: 0x###############
		Link layer: InfiniBand
```


이와 같은 문제를 해결하기 위하여 Open Subnet Manager를 실행하여야 한다. InfiniBand 의 기능을 사용하기 위해선 OpenSM을 설정해주어야 한다. (OpenSM에 대한 내용은 추후에 기억이 나거나, 제대로 설정할 수 있는 기회가 있다면 그때 새로운 글을 쓸 예정이다)


다음과 같은 명령어로 설정 파일을 생성한다. 그리고, 자신이 좋아하는 에디터로 생성한 opensm.conf 파일을 편집한다.

```
$ opensm -c opensm.conf
[jkpark@node1 ~]$ opensm -c opensm.conf
-------------------------------------------------
OpenSM 5.3.0.MLNX20181108.33944a2
Command Line Arguments:
 Creating config file template 'opensm.conf'.
 Log File: /var/log/opensm.log
------------------------------------------------
```


opensm.conf 파일을 편집하여 다음과 같은 부분을 수정한다. 이외의 부분은 당장 설정할 필요는 없다.

```
# Line: 34
# Subnet prefix used on this subnet
subnet_prefix 0xfe80000000000000 # 서브넷은 각 IBA Interface 마다 달라야한다
```


opensm.conf 파일을 수정하였다면, opensm 을 다음과 같이 실행하고 ibstat을 확인한다.

```
$ sudo opensm -F opensm.conf
[jkpark@node1 ~]$ ibstat
 CA 'mlx4_0'
	CA type: MT4099
	Number of ports: 1
	Firmware version: 2.42.5000
	Hardware version: 1
	Node GUID: 0x################
	System image GUID: 0x################
	Port 1:
		State: Activate
		Physical state: LinkUp
		Rate: 56
		Base lid: 1
		LMC: 0
		SM lid: 2
		Capability mask: 0x02514868
		Port GUID: 0x###############
		Link layer: InfiniBand
```


성공적으로 IBA 포트 상태가 Activate 된 것을 확인할 수 있다. 위와 같은 작업을 각 노드마다 설정한다.  주의할 점은 서로 다른 IP 주소와 subnet_prefix를 설정하는 것 뿐이다. 


<h2>벤치마킹<br></h2>



두 대의 노드 간 IBA 통신이 원활하게 되는지 확인하기 위하여 MLNX_OFED에 포함되어 있는 Bandwidth 측정 도구인 <em>ib_send_bw</em> 툴을 사용하였다. <br><br>서버(192.168.1.10)에서는 다음과 같이 실행한다. 

```
[jkpark@node1 ~]$ sudo ib_send_bw -d mlx4_0 -i 1 -F --report_gbits

************************************
* Waiting for client to connect... *
************************************
```


클라이언트(192.168.1.20)에서는 다음과 같이 실행한다. 

```
[jkpark@node2 ~]$ sudo ib_send_bw -d mlx4_0 -i 1 -F --report_gbits 192.168.1.10
---------------------------------------------------------------------------------------
                    Send BW Test
 Dual-port       : OFF		Device         : mlx4_0
 Number of qps   : 1		Transport type : IB
 Connection type : RC		Using SRQ      : OFF
 TX depth        : 128
 CQ Moderation   : 100
 Mtu             : 2048[B]
 Link type       : IB
 Max inline data : 0[B]
 rdma_cm QPs	 : OFF
 Data ex. method : Ethernet
---------------------------------------------------------------------------------------
 local address: LID 0x02 QPN 0x0600 PSN 0x225118
 remote address: LID 0x01 QPN 0x0218 PSN 0xd4a9fb
---------------------------------------------------------------------------------------
 #bytes     #iterations    BW peak[Gb/sec]    BW average[Gb/sec]   MsgRate[Mpps]
 65536      1000             49.73              49.57  		   0.094554
---------------------------------------------------------------------------------------
```


클라이언트에서 접속을 시도하고, Bandwidth 측정을 진행한다. 결과적으로 서버는 아래와 같고 클라이언트는 위와 같은 결과를 보여준다. 

```
[jkpark@node1 ~]$ sudo ib_send_bw -d mlx4_0 -i 1 -F --report_gbits

************************************
* Waiting for client to connect... *
************************************
---------------------------------------------------------------------------------------
                    Send BW Test
 Dual-port       : OFF		Device         : mlx4_0
 Number of qps   : 1		Transport type : IB
 Connection type : RC		Using SRQ      : OFF
 RX depth        : 512
 CQ Moderation   : 100
 Mtu             : 2048[B]
 Link type       : IB
 Max inline data : 0[B]
 rdma_cm QPs	 : OFF
 Data ex. method : Ethernet
---------------------------------------------------------------------------------------
 local address: LID 0x01 QPN 0x0218 PSN 0xd4a9fb
 remote address: LID 0x02 QPN 0x0600 PSN 0x225118
---------------------------------------------------------------------------------------
 #bytes     #iterations    BW peak[Gb/sec]    BW average[Gb/sec]   MsgRate[Mpps]
 65536      1000             0.00               49.81  		   0.095003
---------------------------------------------------------------------------------------
```


벤치마크 결과를 보았을 때 Bandwidth Average가 49.81Gb/sec 란 수치를 보여주고 있다. 이는 본 시스템에 설치되어 있는 InfiniBand 장비에서 최대로 낼 수 있는 속도(54Gb)에 상회하는 속도이며 정상적으로 동작하고 있음을 알 수 있다. 


<h2>결론</h2>



CentOS 7.6 환경에서 InfiniBand 장비 중 하나인 Mellanox ConnectX-3 FDR 장비를 인식시키기 위하여 MLNX_OFED를 설치하는 것을 다루었고, 발생하는 문제에 대해서 해결하는 방법과 성능측정을 하는 것을 알아보았다. 이제 InfiniBand를 접해서 프로젝트를 진행하는 것이기 때문에 아직 부족함이 많지만, 진행하면서 얻을 수 있는 정보를 정리해서 계속 포스팅을 해 볼 예정이다.