---
title: Linux's New Superpowers - Introducting eBPF
revealOptions:
  transition: 'none'
backgroundTransition: 'fade'
verticalSeparator: -v-
---

<!-- .slide: data-background="img/r2_color_rotate.jpeg" data-background-opacity="0.3" -->
##  Linux's New Superpower - Introducting eBPF

<small>rfc2119</small>

---

<!-- .slide: data-background="img/r2_color_rotate.jpeg" data-background-opacity="0.3" -->
## Packet Filtering

- Packet: a formatted unit of data carried by a packet-switched network
<!-- .element: class="fragment" -->
- Packet strictly refers to a protocol data unit (PDU) at layer 3, the network layer.
<!-- .element: class="fragment" -->
- Filters are implemented at both software and hardware (e.g dedicated firewall box) levels
<!-- .element: class="fragment" -->
- Software packet filters allows general-purpose machines for network monitoring
<!-- .element: class="fragment" -->

notes: so how does properiatery software packet filters / firewalls operate ? their OS should be finely tuned to their hardware, but ¯\\_(ツ)_/¯

---

<!-- .slide:  data-background-image="img/bpf_overview.png" data-background-size="90%" data-background-transition="none" -->
### BPF

note: When a packet arrives at a network interface the link level device driver normally sends it up the system protocol stack. But when BPF is listening on this interface, the driver first calls BPF. BPF feeds the packet to each participating process’ filter. This **user-defined** filter decides whether a packet is to be accepted and how many bytes of each packet should be saved. For each filter that accepts the packet, BPF copies the requested amount of data to the buffer associated with that filter. The device driver then regains control. If the packet was not addressed to the local host, the driver returns from the interrupt. Otherwise, normal protocol processing proceeds

---

<!-- .slide: data-background="img/r2_color_rotate.jpeg" data-background-opacity="0.3" -->

## BPF
* Berkely Packet Filter
<!-- .element: class="fragment" -->

* 1993, made for network packet filtering
<!-- .element: class="fragment" -->

* provides a **raw** interface to data link layer**s**
<!-- .element: class="fragment" -->

* goals:
<!-- .element: class="fragment" -->
  * minimize transition to user-space
    <!-- .element: class="fragment" -->
  * filtering packet as efficient as possible
    <!-- .element: class="fragment" -->
<!-- .element: class="fragment" -->

---

<!-- .slide: data-background="img/r2_color_rotate.jpeg" data-background-opacity="0.3" -->

### BPF
An **in-kernel** **sandboxed** VM:
```
# tcpdump host 127.0.0.1 -d
(000) ldh      [12]
(001) jeq      #0x800           jt 2    jf 6
(002) ld       [26]
(003) jeq      #0x7f000001      jt 12   jf 4
(004) ld       [30]
(005) jeq      #0x7f000001      jt 12   jf 13
(006) jeq      #0x806           jt 8    jf 7
(007) jeq      #0x8035          jt 8    jf 13
(008) ld       [28]
(009) jeq      #0x7f000001      jt 12   jf 10
(010) ld       [38]
(011) jeq      #0x7f000001      jt 12   jf 13
(012) ret      #262144
(013) ret      #0
```

---

<!-- .slide: data-background="img/r2_color_rotate.jpeg" data-background-opacity="0.3" -->
### eBPF

* **extended** BPF (**e**BPF). Can be used for non-networking purposes!
<!-- .element: class="fragment" -->

* Available in the Linux kernel >= 3.15 (see [patch](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=bd4cf0ed331a275e9bf5a49e6d0fd55dffc551b8))
<!-- .element: class="fragment" -->

* Running <span class="fragment highlight-red"> user-space code  inside kernel</span> is a powerful tool for kernel developers and production engineers
<!-- .element: class="fragment" -->


-v-
<!-- .slide: data-background="img/x86-rings-2.png" data-background-opacity="0.3" -->

#### CPU Privilege Levels

-v-

<!-- .slide: data-background-transition="none" data-background-color="#ffffff" data-background="img/x86-rings-2.png" -->

-v-
<!-- .slide:  data-background-image="img/x86rings.png" data-background-color="#ffffff"  data-background-size="60%" data-background-transition="none" -->

note: data-background-position="bottom"

-v-
<!-- .slide: data-background-color="#ffffff" data-background-transition="none" -->

#### CPU Privilege Levels
* kernel code:
  * direct access to hardware (peripherals, memory, hard disks, network cards, ... etc.) <!-- .element: class="fragment" -->

  * schedule jobs <!-- .element: class="fragment" -->

  * crash = BSOD <!-- .element: class="fragment" -->

  * is included in your operating system <!-- .element: class="fragment" -->

* user code:
  * limited access (can it read files from your hard disk ?) <!-- .element: class="fragment" -->

  * crash = throw an exception <!-- .element: class="fragment" -->

-v-

<!-- .slide: data-background-color="#ffffff"  data-background-image="img/system-call-overview-1.png" data-background-size="100%" data-background-opacity="0.3"-->
#### Transitioning To Kernel Space

-v-

<!-- .slide: data-background-color="#ffffff" data-background-transition="none" data-background-image="img/system-call-overview-1.png" data-background-size="100%"-->

-v-

<!-- .slide: data-background-color="#ffffff" data-background-transition="none"-->
#### Transitioning To Kernel Space
1. system calls

2. CPU exceptions  (running out of memory, division by zero, ...) <!-- .element: class="fragment" -->

3. hardware Interrupts <!-- .element: class="fragment" -->

note: see img/ARM-shell-0.png.pagespeed.ce.6wC1FtlsVk.png

---

<!-- .slide: data-background="img/r2_color_rotate.jpeg" data-background-opacity="0.3" -->

#### How it works
![Compile_bpf_Syscall.PNG](img/Compile_bpf_Syscall_org.PNG)

---
<!-- .slide: data-background="img/r2_color_rotate.jpeg" data-background-opacity="0.3" -->

### SDN with XDP
<!-- .element text-align="left" -->

* an e**X**press **D**ata **P**ath (XDP)  in kernel-space
* determines *data paths* for a received packet
<!-- .element: class="fragment" -->

* works by adding eBPF hooks in the [NIC](https://www.wikiwand.com/en/Network_interface_controller) driver
<!-- .element: class="fragment" -->

* eBPF hooks can change **on the fly**!
<!-- .element: class="fragment" -->

* can drop a whooping **26 millions of packets per second** per core with commodity hardware!
 <!-- .element: class="fragment" -->

* [Netronome](https://www.wikiwand.com/en/Netronome) NICs has native support for XDP
<!-- .element: class="fragment" -->

note: from wiki: The idea behind XDP is to add an early hook in the RX path of the kernel, and let a user supplied eBPF program decide the fate of the packet. The hook is placed in the [NIC](https://en.wikipedia.org/wiki/Network_interface_controller) driver just after the [interrupt](https://en.wikipedia.org/wiki/Interrupt) processing, and before any memory allocation needed by the [network stack](https://en.wikipedia.org/wiki/Protocol_stack) itself, because memory allocation can be an expensive operation
also, there's some company out there that made a commercial SDN solution using eBPF

---
<!-- .slide: data-background="img/r2_color_rotate.jpeg" data-background-opacity="0.3" -->
### outro

* could do anything  with high performance and minimal overhead (filter/classify traffic, reactive defensive networking, ... )
<!-- .element: class="fragment" -->

* take eBPF seriously, it's game changing!
<!-- .element: class="fragment" -->

* katran (L2-L3 load balancing), goBPF, <span class="fragment highlight-red" data-fragment-index="2">redBPF</span>,  bpfd, <span class="fragment highlight-red" data-fragment-index="2">bpf-seccomp</span>, <span class="fragment highlight-red" data-fragment-index="2">cilium</span>.
<!-- .element: class="fragment" -->

note: bpf-cilium-turning-linux-into-a-microservicesaware-operating-system-26-638.jpg

-v-

<!-- .slide: data-background-image="img/cilium.png" -->

-v-
<!-- .slide: data-background="img/r2_color_rotate.jpeg" data-background-opacity="0.3" -->

### Usages

* eBPF is especially suited to writing network programs and it's possible to write programs that attach to a network socket to filter traffic, to classify traffic, and to run network classifier actions
* Another type of filtering performed by the kernel is restricting which system calls a process can use. This is done with [seccomp BPF](https://lwn.net/Articles/656307/). (see `pledge` syscall and `sysctl security.bsd` in bsd)
* eBPF is also useful for debugging the kernel and carrying out performance analysis (and also user). It's even possible to use eBPF to debug user-space programs by using [Userland Statically Defined Tracepoints](http://blog.memsql.com/bpf-linux-performance/)

---
<!-- .slide: data-background="img/r2_color_rotate.jpeg" data-background-opacity="0.3" -->

### Resources

* [A thorough introduction to eBPF](https://lwn.net/Articles/740157/)<!-- .element: class="fragment highlight-red" data-fragment-index="1"-->  - LWN

* [BPF - Tracing and More](https://www.youtube.com/watch?v=JRFNIKUROPE)<!-- .element: class="fragment highlight-red" data-fragment-index="1"-->

* [The BSD Packet Filter: A New Architecture for User-level Packet Capture](http://www.tcpdump.org/papers/bpf-usenix93.pdf) (original BPF paper)

* [BPF Performance Tools: Linux System and Application Observability](https://www.oreilly.com/library/view/bpf-performance-tools/9780136588870/)- Brendan Gregg's Blog (see [example chapter](https://www.oreilly.com/library/view/linux-observability-with/9781492050193/ch07.html) on observability)

* [Understanding User and Kernel Mode](https://blog.codinghorror.com/understanding-user-and-kernel-mode/) - Jeff Atwood

---
<!-- .slide: data-background="img/r2_color_rotate.jpeg" data-background-opacity="0.3" -->

* [User- and Kernel Mode, System Calls, I/O, Exceptions](https://minnie.tuhs.org/CompArch/Lectures/week05.html)

* [strace Wow Much Syscall](http://www.brendangregg.com/blog/2014-05-11/strace-wow-much-syscall.html) - Brendan Gregg's Bloga

* [XDP](https://www.iovisor.org/technology/xdp)<!-- .element: class="fragment highlight-red" data-fragment-index="1"-->  - IO Visor Project

* [Locking down a FreeBSD system with sysctl](https://www.techrepublic.com/blog/it-security/use-sysctl-security-settings-to-lock-down-a-freebsd-system/)

* eBPF working [diagram](https://ssup2.github.io/theory_analysis/Linux_BPF/)

* [BPF, eBPF, XDP and Bpfilter… What are These Things and What do They Mean for the Enterprise?](https://www.netronome.com/blog/bpf-ebpf-xdp-and-bpfilter-what-are-these-things-and-what-do-they-mean-enterprise/)<!-- .element: class="fragment highlight-red" data-fragment-index="1"-->  - Netronome Blog

* [eBPF Vulnerability](https://blog.aquasec.com/ebpf-vulnerability-cve-2017-16995-when-the-doorman-becomes-the-backdoor) (CVE-2017-16995)
<!-- .element: class="fragment highlight-red" data-fragment-index="1"-->

---
<!-- .slide: data-background="img/r2_color_rotate.jpeg" data-background-opacity="0.3" -->
#### \</eBPF>

slides @ [rfc2119.github.io/ebpf-talk](https://rfc2119.github.io/ebpf-talk)
