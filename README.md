# Real-time Capabilities of Linux

Examining RT Linux and its Computing Benefits

Compared to a regular OS, RT Systems provide 
a tight timeframe for system tasks to be 
completed. When certain functions must be 
completed on a system, RTs provide a reliable 
way to ensure they output a response within
a time constraint. The speed at which these 
are completed is often determined by the 
priority of the tasks being completed, thus
distributing system resouces to the highest 
priorities first. In contrast, a general 
purpose OS simply coordinates incoming 
operations on the go giving equal performance
to all functions, which can become an issue 
if there is a more important function in 
the queue. 

RT is vital for systems that need to deliver 
an output predictably. This can include 
system controls, like motor vehicles or 
medical equipment. 

# Hard Real-time vs. Soft Real-time 

The difference between these types of systems
is in the event of a failed function, or 
what is considered a "missed deadline" (Failing
to complete a task in a specified deadline). 

Soft RT systems continue to function after 
failure. However, they will continue running
and move onto the next operation. Whilst the
output will be worse and some data may be lost, 
the system can still continue functioning 
without too much adverse effects. (Can be
used in calls, online data transfers). 

Hard RT systems on the other hand cannot 
fail. They require zero delay tolerance 
where not completing a task within the 
time frame can result in catastrophe failure
to the technology is it applied to. (Can 
be used in vehicle systems, medical 
equipment). 

# Investigation of RT Linux 

The Linux OS can be made into an RT system
with the associated patch named "PREEMPT_RT". 
This comes with patches that improve system
responsiveness and essentially upgrades Linux
into an RT system. The "Preempt" stands 
for preemption, which means
Linux will stall tasks considered to be 
lower priority in favor of higher priority just
as an RT system would. This turns Linux into a 
soft RT system. While it can still occassionaly 
miss deadlines, it is not too determential 
and expected for it to continueing running 
as an OS. 

Linux Preemption allows for system tasks to 
be interupted when a more important function
needs to be completed (Basically being able 
to stop a task to run a higher priority task
first then returning to the previous task). 

RT Linux does this by using interrupts in the
OS scheduler - which distinguishes which 
tasks are to be run at which time in an order. 
RT Linux tasks will always have higher priority
than the regular Linux Kernel. It should be noted
that in these instances RT Linux has its own kernel,
and the regular Linux OS uses its own as well. The
regular Linux kernel will only run when all RT Linux
tasks are finished. In summary, whenever RT Linux 
needs to run a task, it sends an interrupt to the 
scheduler and takes priority over Linux Kernel tasks.
The RTOS then distributes any necessary resources as
it sees fit to complete its priorities (CPU usage, 
mem, queues). 

# Market RT OS Solutions 

There are many RT OS systems in the market built for 
different applications, especially in the automotive, 
software, and medical environments. 

LynxOS from Lynx Software Technologies (Automotive, 
Industrial, IT). emdOS from Segger with use cases 
in Networking devices, battery powered handhelds. 
FreeRTOS by Amazon is a open source kernel to provide
an RT OS to a system. There are many other RT OS with
varying degrees of uses. Zephyr (Linux), ThreadX (Microsoft), 
Neutrino (BlackBerry). 

# Analysis of RT Patch for Kernel 6.12

Many scripts in the patch folder appear to be enablers for
preemption with modifications for the scheduler 
and memory allocators. 

At a glance, the VFP (Vector Floating Point) processors in 
ARM - which in general run floating point operations - disables
preemption and can't get interrupted. With disabled preemption,
this can harm processes that the system deems higher priority
in this instance, thus a quick modification seems to apply 
vfp_state_hold() to allow preemptive synchronization. (Seen
in patch 0031). 

There is a modification to Interrupt Requests (IRQs) that enables
it for exceptions in ARM architecture as "ARM disables irq when it
enters exception/interrupt mode". Since interrupts are an important
part of RT systems to be able to priortize certain tasks in the
scheduler, it is viable to ensure that different modes are 
interruptable. 

The RT Patch enablers often include simply configuring some 
architecture support options related to config files. 

Jump labels get disabled (patch 0028). These are used 
to switch between different instructions/codes in the
OS. However, whilst this modification processes, 
the stop_machine() function is called and stalls the
CPU. This can be for an unpredicable timeframe and
cause higher wait times for tasks to execute - a fact
that RT systems want to avoid entirely. Thus the support
for jump labels is disabled in the PREEMPT_RT system. 

Other modifications follow a similar mindset where certain
features (Spinlocks, softirqs, atomic updates) are changed 
so that RT Linux can supersede them when it needs to run
a priorited task, or they are simply optimized to be
more in line with an RT OS. 

# Transforming into RT Linux 

Get *patches-6.12.43-rt12.tar.xz* file from 
kernel.org/pub/linux/kernel/projects/rt/6.12/ to
view individual patch files. Get *patch-6.12.43-rt12.patch.gz*
from same location for single patch file for ease of 
implementation. 

gunzip patch-6.12.43-rt12.patch.gz to unzip this type of file

```sh
cd linux-6.12.43
patch -p1 < ../patch-6.12.43-rt12.patch
```

If you move it into the kernel folder, can just do: 
```sh
patch -p1 < patch-6.12.43-rt12.patch
```

Necessary Build Configurations: 

These can be found and configured in the menuconfig
of the linux kernel with

```sh
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig
```

CONFIG_PREEMPT_RT=y

This is the most important configuration to enable RT for 
the Linux Kernel, should be configured to say "=y":


CONFIG_NO_HZ_FULL=y
CONFIG_HZ_1000=y (Highest possible frequency to response)
CONFIG_CPUSETS=y (Isolating CPUs for RT Workloads)
CONFIG_BLK_CGROUP_IOLATENCY=y (Improved latencies)

Then compile the kernel

```sh
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j4
```

When running the kernel, using *uname -a* will show
that it is a RT Kernel. 

![](/RTPatch.PNG)

# Comparing Performance Between Standard OS and RT OS: 

By using the cyclictest package, each OS can be evaluated
based on their latency readings. Cyclictest will create, 
prioritize, and awaken threads (scheduled tasks). The threads
are expected to wake up within set intervals, but they will
usually be delayed by a bit. This delay is the measured 
latency for which to evaluate the OS performance. Below are
general system tests. 

Process for installing cyclictest on ARM/ARM64
https://zhuanlan.zhihu.com/p/512480007?share_code=12wltdj8oKZfo&utm_psn=1948717590580028944

Testing for an x86 OS and ARM64 (Using cyclictest executable): 

```sh
./cyclictest --smp -t2 -a 0-1 -d200 -p80
```



# Sources 

https://www.intel.com/content/www/us/en/learn/what-is-a-real-time-system.html

https://ubuntu.com/blog/real-time-linux-vs-rtos-2

https://www.cs.ru.nl/~hooman/DES/RealtimeLinuxBasics.pdf

https://www.linuxfoundation.org/blog/blog/intro-to-real-time-linux-for-embedded-developers

https://www.lynx.com/blog/most-popular-real-time-operating-systems-rtos#:~:text=MOST%20POPULAR%20RTOS%20(2025)%20*%20Deos%20(DDC%2DI),VxWorks%20(Wind%20River)%20*%20Zephyr%20(Linux%20Foundation)

https://tldp.org/HOWTO/pdf/RTLinux-HOWTO.pdf
