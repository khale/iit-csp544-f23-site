# Rootkits and Backdoors

The goal: establish a *persistent threat*. Not only have we gained control of a machine; we get the ability to maintain
that control. 

One way to do that: employ a rootkit - a piece of software that gives us a jumping off point for further attacks/analysis etc.

A rootkit can run as a privileged user program (e.g., a daemon), but that limits its power. 

What kinds of things do we want to do with a rootkit?
- Send telemetry back to command/control server or attacker
- Stage attacks to other systems
- Allow for in-bound connections by attackers (backdoor)
- Hide network connections
- Hide files
- Hide processes
- Make itself persistent
- Manipulate I/O traffic

Many of these are difficult to do in userspace, so we go to the kernel.

## The Kernel

- Software that runs in a separate privilege mode (ring 0) from user processes
- Isolated address space
- Has full (mostly) control of the machine: there are exceptions such as SGX, SMM, hypervisors, etc.
- Services requests from userspace via syscall API. Mostly runs in response to external events.


Good software architecture pretty much obviates a truly monolithic kernel. Most OSes in use today
have _loadable kernel modules_. They allow kernel to extend its functionality; separation of concerns.
Examples:
- filesystems
- device drivers
- experimental kernel functionality

They are compiled into object files (`.ko` on Linux), and linked at runtime against a running kernel. On Linux,
they conform to a very thin API. Windows is more restrictive. **The modules run at the same privilege level as the kernel!**
This makes them very dangerous. 

Typical Flow (Linux):
- Device designer writes a kernel module to drive device
- Drivers ship separate from the kernel.
- Privileged user (root) can insert the module at runtime to use the functionality (e.g., `insmod foo.ko`)
- Userspace programs and libraries interface with the device driver via special syscalls (`ioctl()`) or
virtual files created by the driver (e.g., in sysfs, or profs, or in `/dev`). 

How does kernel prevent misuse:
- Limit scope of access to kernel: self write-protected kernel regions, KASLR, `EXPORT` macros (compiler/linker techniques)
- Module signing: only modules from trusted sources can be inserted into the kernel! (You'll understand this later with PKI)

**Userspace is completely under control of the kernel!**:
- How do you view files? `ls`. How does this work?
- How do you view network connections? 
- Processes?

It's all via **kernel** interfaces. If these interfaces can be manipulated, we can **scrub** the information they 
return.

