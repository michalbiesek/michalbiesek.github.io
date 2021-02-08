---
title: Memkind support for heterogeneous memory attributes
author: michalbiesek
layout: post
identifier: hmat_memkind
---

### Introduction

Memkind is a library which provide support for Persistent Memory but it is not limited only for
it. In fact it is general solution designated for platform with heterogenous memory.

But before we go to heterogenous memory itself, let's start from short recap about [NUMA][kernel-numa].
Primary the NUMA concept was designated to handle the reality - the dynamic extension of CPU count-per-socket and system memory.
With initial solution with same level of latency access from all the CPU's to whole memory, platforms faced scalability problem.
Single shared bus between CPU and DRAM memory become a bottleneck, each CPU decrease the available bandwidth for Node and increasing length

Provide two ways 

Providing the simillar access of same level of latency access to With it platforms scalability problem, adding
extended so much, that uniform memory accessing time for all the CPU's become a bottleneck - memory bus got limits in terms of bandwidth

 shared bus between CPUs and RAM is not a trivial engineer task and definitely isn’t a cheap solutio

and each new CPU decrease tha banwdith avaialble, the physycial bus must also increate the legnth which impacts latency.
The solution for this problem was Non-Uniform Memory access, where isntead of having unfi

symetric

Multiple memory controllers
A NUMA optimized operating system such as ESXi allows workload to consume memory from both memory addresses spaces while optimizing for local memory access


Today's reality is dynamically developing in manner of different Processing Unit, but the memory
doesn't stay behind in this race. With both of these facts the NUMA Node concept is a little vague.
The NUMA Node itself now doesn’t have to be a couple between memory and CPU. For example, we could have a memory-only NUMA Node, the NUMA Node, which cannot generate memory requests and contains only memory. In the real world, this is a hot-added memory device.
On top of that, we could have a different type of memory initiators: today, CPU can perform memory allocation request and other processing units as Tensor Processing Unit or Graphics processing unit. The same applies to memory targets as this could be different memory physical mediums like DRAM, PMEM, and HBW.
Long story short: NUMA Node concept evolve, and it’s much different compared to the past. No 
No longer if some entity can make allocation request means that it has to be CPU. A similar rule applies to memory – if the processing unit can make a memory allocation request, it doesn’t mean that memory must come from DRAM.


However, bus-based systems have their own scalability problems. The main issue is the limited amount of bandwidth, this restrains the number of processors the bus can accommodate. Adding CPUs to the system introduces two major areas of concern:

The available bandwidth per node decreases as each CPU added.
The bus length increases when adding more processors, thereby increasing latency.
The performance growth of CPU and specifically the speed gap between the processor and the memory performance was, and actually still is, devastating for multiprocessors. Since the memory gap between processor and memory was expected to increase, a lot of effort went into developing effective strategies to manage the memory systems. 


 systems today.

Memory access time and effective memory bandwidth varies depending on how far away the cell containing the CPU or IO bus making the memory access is from the cell containing the target memory.

 Non-Uniform Memory access 

particular CPUs or sockets


nce central implies single, whereas most modern systems have more than one processing unit, or core. Physically, CPUs are contained in a package attached to a motherboard in a socket. Each socket on the motherboard has various connections: to other CPU sockets, memory controllers, interrupt controllers, and other peripheral devices. A socket to the operating system is a logical grouping of CPUs and associated resources. This concept is central to most of our discussions on CPU tuning.
Red Hat Enterprise Linux keeps a wealth of statistics about system CPU events; these statistics are useful in planning out a tuning strategy to improve CPU performance. Section 4.1.2, “Tuning CPU Performance” discusses some of the more useful statistics, where to find them, and how to analyze them for performance tuning.
Topology
Older computers had relatively few CPUs per system, which allowed an architecture known as Symmetric Multi-Processor (SMP). This meant that each CPU in the system had similar (or symmetric) access to available memory. In recent years, CPU count-per-socket has grown to the point that trying to give symmetric access to all RAM in the system has become very expensive. Most high CPU count systems these days have an architecture known as Non-Uniform Memory Access (NUMA) instead of SMP.



I/O buses, that comprise what is known as a “NUMA node”.
Processor accesses to memory or I/O resources within the local NUMA node is generally faster than
processor accesses to memory or I/O resources outside of the local NUMA node.




But it is not always the case, application expectation are often releated not to the hardware itself
but to specific memory property - in other words what memory offers.
To follow this belief we extend memkind library with different type of abstraction.

We could take a look into memory from hardware point of view, but we could change the perspective
to memory property overview.

The following table contains a comparision between hardware and :

| Physical medium      | Memory property |
| -------------------- | ----------------|
| DRAM                 | Latency         |
| [MCDRAM][MCDRAM]     | Bandwidth       |
| PMEM                 | Capacity        |

### Why should I care about memory property semantics ?

The answer is pretty straightforward, application provided requierments against task as
processing obligations or memory obligations increasing the memory bandwidth available. 

The library could support this in different type of abstraction providing more hardware-agnostic interface.

, expected behaviour
doesn't have to be related to which medium currently supports it.


uld expect. Their AI capabilities,
S

memory 


expects different behaviour 
what physical medium fullfill applcation requierments could be irrrelevant 

on the application layer the processing unit as GPU/CPU same th

The second one is today's limitation

- currently we utilize information about NUMA distance which doesn't cover all
the expected use cases. NUMA distance abstraction covers only latency property


To holding the principle that 1 picture is worth more than 1000 words.
Let's get back, what we [already][hazelcast-post] know from memkind semantics:

<figure class="image">
  <img src="/assets/hazelcast_3.png" alt="PMEM KMEM-DAX">
  <figcaption>Figure 1. Memory architecture overview based on physical medium.</figcaption>
</figure>

And presented same view in different form:

<figure class="image">
  <img src="/assets/memkind_memory_view.png" alt="CAPACITY LATENCY VIEW">
  <figcaption>Figure 2. Memory architecture overview based on memory property.</figcaption>
</figure>

With this approach memkind library become offers more hardware agnostic.

### Prerequisites

[ACPI-6.2][ACPI-doc] extends the ACPI standard about Heterogeneous Memory Attribute
Table (HMAT) table. HMAT provides a way for firmware to propagate to Operating System memory hardware
characteristics - in details it describes bandwidth and latency from the initiator of memory requests
(a processor or an accelerator) to any memory target.

Memkind utilize this information and take advantage of it.

Besides support for HMAT in hardware
Linux kernel version brings in support for HMAT:

{% highlight console %}
$ make nconfig
	Power management and ACPI options --->
		[*] ACPI (Advanced Configuration and Power Interface) Support --->
			-*-   NUMA support
			[*]     ACPI Heterogeneous Memory Attribute Table Support
{% endhighlight %}

To utilize information provided by OS memkind use [hwloc library][hwloc] therefore it is one of the dependency to
use new features offer by memkind.

### Memory kind summary

| New API                                   | Associated Memory property |
| ----------------------------------------- | -------------------------- |
| MEMKIND_HIGHEST_CAPACITY					        |   Capacity                 |
| MEMKIND_HIGHEST_CAPACITY_PREFERRED		    |   Capacity                 |		
| MEMKIND_HIGHEST_CAPACITY_LOCAL            |   Capacity                 |
| MEMKIND_HIGHEST_CAPACITY_LOCAL_PREFERRED  |   Capacity                 |
| MEMKIND_LOWEST_LATENCY_LOCAL              |   Latency                  |
| MEMKIND_LOWEST_LATENCY_LOCAL_PREFERRED    |   Latency                  |
| MEMKIND_HIGHEST_BANDWIDTH_LOCAL           |   Bandwidth                |
| MEMKIND_HIGHEST_BANDWIDTH_LOCAL_PREFERRED	|   Bandwidth                |

### What's the point to have local property kind summary

In memkind we 

But today’s things are more complicated, and the NUMA Node concept is a little vague. The NUMA Node itself now doesn’t have to be a couple between memory and CPU. For example, we could have a memory-only NUMA Node, the NUMA Node, which cannot generate memory requests and contains only memory. In the real world, this is a hot-added memory device.
On top of that, we could have a different type of memory initiators: today, CPU can perform memory allocation request and other processing units as Tensor Processing Unit or Graphics processing unit. The same applies to memory targets as this could be different memory physical mediums like DRAM, PMEM, and HBW.
Long story short: NUMA Node concept evolve, and it’s much different compared to the past. No 
No longer if some entity can make allocation request means that it has to be CPU. A similar rule applies to memory – if the processing unit can make a memory allocation request, it doesn’t mean that memory must come from DRAM.

s the relative distance (memory latency) between
all System Localities, which are also referred to as Proximity Domains. Systems employing a Non
Uniform Memory Access (NUMA) architecture contain collections of hardware resources including
for example, processors, memory, and I/O buses, that comprise what is known as a “NUMA node”.
Processor accesses to memory or I/O resources within the local NUMA node is generally faster than
processor accesses to memory or I/O resources outside of the local NUMA node. 

Locality – hwloc with the topology solution provides an easy way to support the correlation between NUMA Node as memory and processing unit. It allows us to determine the correlation to socket – in memkind, we depend mostly on Numa distance metrics in this manner

In case of non-symmetrical memory topology, where for example

5.2.17 System Locality Distance Information Table (SLIT)
Socket correlation / Sub NUMA cluster

### What’s Coming Next

NUMA Node concept is vague right compared to past. The NUMA Node itself now doesn’t have to be a
couple between memory and CPU. For example, we could have a memory-only NUMA Node, the NUMA Node,
which cannot generate memory requests and contains only memory(hot-added memory device).
So modern approach to None Unified Memory Architecture is concept “initiator of memory request” in the form of Processing Unit(s), e.g CPU, GPU, TPU and "target of memory request" in the form of memory(s), which is characterized by different memory properties
like latency/bandwidth depending on specific initiator of memory request.

We believe that more complex memory architectures will be widely available in the future.

With this perspective a one dimensional approach based on NUMA distance metrics would be out-of-date 
Hardware/Firmware architects already defining soultion with HMAT and provided this extension in 
Linux Kernel developers provided this support in recent kernel version.
On application layer libmemkind tries to measure those challenges as well with new features presented in this post.

[kernel-numa]: https://www.kernel.org/doc/html/latest/vm/numa.html
[memkind-release]: https://github.com/memkind/memkind/releases/tag/v1.11.0
[MCDRAM]: https://software.intel.com/content/www/us/en/develop/blogs/an-intro-to-mcdram-high-bandwidth-memory-on-knights-landing.html
[ACPI-doc]: https://uefi.org/sites/default/files/resources/ACPI_6_2.pdf
[hwloc]: https://www.open-mpi.org/software/hwloc/v2.3/
[hazelcast-post]: https://pmem.io/2021/02/11/hazelcast_memkind.html
