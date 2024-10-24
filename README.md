# BinaryInferno

* BinaryInferno is a tool designed to help automatically reverse engineer the formats of binary messages. 

* BinaryInferno is best described in this paper [BinaryInferno: A Semantic-Driven Approach to Field Inference for Binary Message Formats](https://github.com/binaryinferno/binaryinferno/blob/main/BinaryInferno2023Chandler.pdf).

* Just want to try it on some data?  Quick Start Google Colab Notebook [BinaryInferno.ipynb](https://github.com/binaryinferno/binaryinferno/blob/main/BinaryInferno.ipynb).

* More examples and better documentation will come as time permits. If you'd like to be notified by email [fill out this form](https://forms.gle/xH3rPyn7GvfSm2pL7).

* If you are interested in protocol reverse engineering, consider participating in a related user-study: [https://tsp.cs.tufts.edu/protocol-re/](https://tsp.cs.tufts.edu/protocol-re/)

# Requirements

* Python libraries `sklearn`, `scipy`.
* Command line util `parallel` for serialization pattern search.
```
conda create -n binaryinferno python=3.7
conda activate binaryinferno
conda install scipy scikit-learn
apt install parallel
```
协议：bgp,mavlink
DHCP (Dynamic Host Configuration Protocol)
DHCP 是一种网络协议，用于自动分配 IP 地址、子网掩码、默认网关以及其他配置参数给局域网内的设备。DHCP 服务器负责维护一个地址池，并向请求 IP 地址的客户端分配这些地址。DHCP 可以简化网络管理，尤其是在大型网络中。

DNP3 (Distributed Network Protocol)
DNP3 是一种专为电力系统设计的通信协议，用于实现远程设备之间的可靠通信。它主要用于监控和控制工业自动化设备，特别是在电力系统、水处理设施等领域。DNP3 提供了多种服务，包括数据采集、事件记录、遥测、遥控等。

SMB (Server Message Block) / SMB2
SMB 是一种应用层网络协议，用于提供共享访问文件、打印机和其他资源的功能。SMB 最初由 IBM 开发，后来微软对其进行了改进和发展。SMB2 是 SMB 协议的一个重大更新版本，提供了更好的性能、安全性和互操作性。

MAVLink
MAVLink 是一种轻量级的消息协议，用于无人机系统之间的通信。它支持多种平台间的数据交换，包括地面站、飞行控制器、传感器等。MAVLink 消息格式紧凑高效，广泛应用于无人驾驶航空器（UAVs）的研究和开发中。

Mirai
Mirai 不是一种协议，而是一种恶意软件，以其针对物联网设备的能力而臭名昭著。Mirai 能够感染大量连接互联网但缺乏安全措施的设备，形成僵尸网络，进而发动大规模分布式拒绝服务攻击（DDoS）。Mirai 通常利用默认用户名和密码来感染设备。

MODBUS
MODBUS 是一种串行通信协议，用于连接工业电子设备，使其能够相互通信。它最初由 Modicon（现在的施耐德电气）公司为可编程逻辑控制器（PLCs）设计，现在广泛用于工业控制系统中，支持多种传输模式，包括 RS-232、RS-485 和 TCP/IP。

NTP (Network Time Protocol)
NTP 用于在分布式网络环境中同步计算机时钟的时间。它是一个基于 UDP 协议的客户端/服务器模式的应用层协议。NTP 的设计目的是尽量减少时钟同步的网络延迟，并提供高精度的时间同步。

这些协议各自服务于不同的应用场景和技术领域，从网络时间同步到工业自动化控制都有涉及。了解这些协议的基本概念有助于在网络管理和安全方面做出正确的决策。

# Usage 

BinaryInferno takes messages to reverse engineer on stdin. One message per line in Hex format.  Here's the example we use in the paper.

```
01000D60A67AED054150504C45
01001160A67B0504504C554D0450454152
01000E60A67AF9064F52414E4745
```
Basic usage: `cat msgs.txt | python3 blackboard.by`

More complicated usage `cat msgs.txt  | python3 blackboard.py --detectors BE --tslow "2001-02-08 11:41:41" --tshigh "2028-02-08 11:41:41" 1> log.txt 2> errs.txt`


* The detectors flag `BE` means use only BIG ENDIAN detectors
* Use detectors flag `LE` for LITTLE ENDIAN detectors

If you want to just use the entropy boundary search: `--detectors boundBE` or `--detectors boundLE`

Timestamp search is performed when a search span is provided:
* `tslow` is lower bound for timestamps
* `tshigh` is upper bound for timestamps 

You can also limit the search to use a specific detector or combination of detectors:

* `boundBE`
* `boundLE`
* `floatLE`
* `floatBE`
* `seq8LE`
* `seq16LE`
* `seq24LE`
* `seq32LE`
* `seq8BE`
* `seq16BE`
* `seq24BE`
* `seq32BE`
* `length`
* `length2LE`
* `length2BE`
* `length3LE`
* `length3BE`
* `length4LE`
* `length4BE`
* `rep_par_BE` (Serialization Pattern Search using BIG ENDIAN multi-byte fields)
* `rep_par_LE` (Serialization Pattern Search using LITTLE ENDIAN multi-byte fields)
* `lvstar`
* `lvone`

Don't worry if timespan is years too wide, that's totally fine in practice.

`log.txt` contains BinaryInferno's exhaustive output

`errs.txt` contains anything which came out on stderr

We mainly care about the stuff at the very end of log.txt

`cat log.txt | awk '/INFERRED DESCRIPTION/,/SPECEND/'` gives us:

```
INFERRED DESCRIPTION
--------------------------------------------------------------------------------

	?? LLLL | TTTTTTTT RRRRRRRRRRRR
	--
	01 000D | 60A67AED 054150504C45
	01 0011 | 60A67B05 04504C554D0450454152
	01 000E | 60A67AF9 064F52414E4745
	--
	0 ? UNKNOWN TYPE 1 BYTE(S) 3.0
	1 L BE UINT16 LENGTH + 0 = TOTAL MESSAGE LENGTH 6.0
	2 T BE 32BIT SPAN SECONDS 2001-02-08 11:41:41.000000 TO 2028-02-08 11:41:41.000000 1.0 12.0
	3 R 0T_1L_V_BIG* 23.0

QTY SAMPLES
3
HEADER ONLY
?? LLLL | TTTTTTTT RRRRRRRRRRRR
SPECSTART
FieldFixed 1V (Unknown Type 1 Byte(s))
Length 2V_BE (BE uint16 Length + 0 = Total Message Length)
FieldFixed 4V_BE (BE 32BIT SPAN Seconds 2001-02-08 11:41:41.000000 to 2028-02-08 11:41:41.000000 1.0)
FieldRep *Q_0T_1L_1V_BE (0T_1L_V_big*)
SPECEND
```

Please see the paper for details on how to interpret the output. 

# Roadmap

Aspirationally, BinaryInferno will will be refactored with three main goals.

* The integration algorithm refactored as a stand-alone component.
* The serialization pattern search algorithm refactored such that it is easier for a novice to add new patterns.
* A refactored detector interface, allowing arbitrary programs to be called as detectors. 

These efforts will take some time.

