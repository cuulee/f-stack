# F-Stack
![](F-Stack.png)

## Introduction
With the rapid development of NIC, the poor performance of data packets processing with Linux kernel has become the bottleneck. However the rapid development of the Internet needs high performance of network processing, kernel bypass has caught more and more attention. There are various similar technologies appear, such as DPDK, NETMAP and PF_RING. The main idea of kernel bypass is that Linux is only used to deal with control flow, all data streams are processed in user space. Therefore kernel bypass can avoid performance bottlenecks caused by kernel packet copy, thread scheduling, system calls and interrupt. Further more, kernel bypass can achieve higher performance with multi optimizing methods.  Within various techniques, DPDK has been widely used because of its more thorough isolation from kernel scheduling and active community support.

[F-Stack](http://www.f-stack.org/?from=github) is an open source network framework with high performance based on DPDK. With follow characteristics

1. Ultra high network performance which can achieve network card under full load, 10 million concurrent connection, 5 million RPS, 1 million CPS.
2. Transplant FreeBSD 11.01 user space stack, provides a complete stack function, cut a great amount of irrelevant features. Therefore greatly enhance the performance.
3. Support Nginx, Redis and other mature applications, service can easily use F-Stack
4. With Multi-process architecture, easy to extend
5. Provide micro thread interface. Various applications with stateful app can easily use F-Stack to get high performance without processing complex asynchronous logic.
6. Provide Epoll/Kqueue interface that allow many kinds of applications easily use F-Stack

## History

 In order to deal with the increasingly severe DDoS attacks, authorized DNS server of Tencent Cloud DNSPod switched from Gigabit Ethernet to 10-Gigabit at the end of 2012. We faced several options, one is to continue to use the original model another is to use kernel bypass technology. After several rounds of investigation, we finally chose to develop our next generation of DNS server based on DPDK . The reason is DPDK providing  ultra high performance, and can be seamlessly extended to 40G, or even 100G NIC in the future. 

After several months of development and testing, DKDNS - DNS server based on DPDK officially released in October 2013. DNS processing performance of single 10GE port was up to 11 million QPS. The DNS processing performance of double 10GE net export reached 18.2 million QPS. Our newly independently developed TCP/IP protocol stack, F-Stack, can process 0.6 million RPS with single 10GE port.

 With the fast growth of Tencent Cloud, more and more services needs higher network access performance. At the meanwhile F-Stack is continuous improving driven by the business growth, and ultimately developed into a general network access framework. For the TCP/IP protocol stack part, we've tried several different plans and finally determine to transplant and simplify the FreeBSD protocol stack to the user space. Therefor we can reduce the cost of maintenance and quickly follow up the new application of the protocol stack.

With rapid development of all kinds of application, in order to help different APPs quick and easily use F-Stack, F-Stack has integrated Nginx, Redis and other commonly used APP, and a micro thread framework, and provides a standard Epoll/Kqueue interface.

Currently, besides authorized DNS server of DNSPod, there are various product in Tencent Cloud has used the F-Stack, such as HttpDNS (D+), COS access module, CDN access module, etc..

## Quick Start

    #clone F-Stack
    mkdir /data/f-stack
    git clone https://github.com/F-Stack/f-stack.git /data/f-stack
    
    cd f-stack
    # compile DPDK
    cd dpdk/tools
    ./dpdk-setup.sh # compile with x86_64-native-linuxapp-gcc
    
    # Set hugepage
    # single-node system
    echo 1024 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages

    # or NUMA
    echo 1024 > /sys/devices/system/node/node0/hugepages/hugepages-2048kB/nr_hugepages
    echo 1024 > /sys/devices/system/node/node1/hugepages/hugepages-2048kB/nr_hugepages
	
    # Using Hugepage with the DPDK
    mkdir /mnt/huge
    mount -t hugetlbfs nodev /mnt/huge
	
    # offload NIC
    modprobe uio
    insmod /data/f-stack/dpdk/x86_64-native-linuxapp-gcc/build/kmod/igb_uio.ko
    insmod /data/f-stack/dpdk/x86_64-native-linuxapp-gcc/build/kmod/rte_kni.ko
    python dpdk-devbind.py --status
    ifconfig eth0 down
    python dpdk-devbind.py --bind=igb_uio eth0 # assuming that use 10GE NIC and eth0
	
    # Compile F-Stack
    cd ../../lib/
    make
    export FF_PATH=/data/f-stack
    export FF_DPDK=/data/f-stack/dpdk/x86_64-native-linuxapp-gcc/lib

#### Nginx

    cd app/nginx-1.11.10
    ./configure --prefix=/usr/local/nginx_fstack --with-ff_module
    make
    make install
    cd ../..
    ./start.sh -b /usr/local/nginx_fstack/sbin/nginx -c config.ini

#### Redis

    cd app/redis-3.2.8/
    make
    make install


## Nginx Testing Result 

Test environment

    NIC:Intel Corporation Ethernet Controller XL710 for 40GbE QSFP+
    CPU:Intel(R) Xeon(R) CPU E5-2670 v3 @ 2.30GHz
    Memory：128G
    OS:CentOS Linux release 7.2 (Final)
    Kernel：3.10.104-1-tlinux2-0041.tl2

Nginx uses linux kernel's default config, all soft interrupts are working in the first CPU core.

Nginx si means modify the smp_affinity of every IRQ, so that the decision to service an interrupt with a particular CPU is made at the hardware level, with no intervention from the kernel. 


CPS (Connection:close, Small data packet)  test result
![](http://i.imgur.com/PvCRmXR.png)

RPS (Connection:Keep-Alive, Small data packet) test data
![](http://i.imgur.com/CTDPx3a.png)

Bandwidth (Connection:Keep-Alive, 3.7k bytes data packet) test data
![](http://i.imgur.com/1ZM6yT9.png)

## Licenses

Main code of F-Stack is BSD licensed.

### Other dependencies and licenses:

#### DPDK

BSD, Copyright(c) 2010-2017 Intel Corporation.

#### FreeBSD

BSD, Copyright 1992-2016 The FreeBSD Project.

#### libuinet
BSD, Copyright (c) 2015 Patrick Kelsey.

#### libplebnet
BSD, Copyright (c) 2011 Kip Macy.

#### Nginx

BSD, Copyright (C) 2002-2017 Igor Sysoev. Copyright (C) 2011-2017 Nginx, Inc.

#### Redis

BSD,  Copyright (c) 2006-2015, Salvatore Sanfilippo.

#### inih(a simple .INI file parser)
BSD, Copyright (c) 2009.

#### Microthread framework

GNU, Copyright (C) 2016 THL A29 Limited, a Tencent company.
