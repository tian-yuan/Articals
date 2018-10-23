##  Interrupt and Process Binding

Realtime environments need to minimize or eliminate latency when responding to various events. Ideally, interrupts (IRQs) and user processes can be isolated from one another on different dedicated CPUs.

Interrupts are generally shared evenly between CPUs. This can delay interrupt processing through having to write new data and instruction caches, and often creates conflicts with other processing occurring on the CPU. In order to overcome this problem, time-critical interrupts and processes can be dedicated to a CPU (or a range of CPUs). In this way, the code and data structures needed to process this interrupt will have the highest possible likelihood to be in the processor data and instruction caches. The dedicated process can then run as quickly as possible, while all other non-time-critical processes run on the remainder of the CPUs. This can be particularly important in cases where the speeds involved are in the limits of memory and peripheral bus bandwidth available. Here, any wait for memory to be fetched into processor caches will have a noticeable impact in overall processing time and determinism.

In practice, optimal performance is entirely application specific. For example, in tuning applications for different companies which perform similar functions, the optimal performance tunings were completely different. For one firm, isolating 2 out of 4 CPUs for operating system functions and interrupt handling and dedicating the remaining 2 CPUs purely for application handling was optimal. For another firm, binding the network related application processes onto a CPU which was handling the network device driver interrupt yielded optimal determinism. Ultimately, tuning is often accomplished by trying a variety of settings to discover what works best for your organization.

**Important**

For many of the processes described here, you will need to know the CPU mask for a given CPU or range of CPUs. The CPU mask is typically represented as a 32-bit bitmask (on 32-bit machines). It can also be expressed as a decimal or hexadecimal number, depending on the command you are using. For example: The CPU mask for CPU 0 only is `00000000000000000000000000000001` as a bitmask, `1` as a decimal, and ` 0x00000001` as a hexadecimal. The CPU mask for both CPU 0 and 1 is `00000000000000000000000000000011` as a bitmask, `3` as a decimal, and `0x00000003` as a hexadecimal.

**Disabling the irqbalance daemon**

This daemon is enabled by default and periodically forces interrupts to be handled by CPUs in an even, fair manner. However in realtime deployments, applications are typically dedicated and bound to specific CPUs, so the `irqbalance` daemon is not required.

1. Check the status of the `irqbalance` daemon

   ```
   # service irqbalance status
   irqbalance (pid PID) is running...

   ```

2. If the `irqbalance` daemon is running, stop it using the `service` command.

   ```
   # service irqbalance stop
   Stopping irqbalance:             [  OK  ]

   ```

3. Use `chkconfig` to ensure that `irqbalance` does not restart on boot.

   ```
   # chkconfig irqbalance off

   ```

**Partially Disabling the irqbalance daemon**

An alternative approach to is to disable `irqbalance` only on those CPUs that have dedicated functions, and enable it on all other CPUs. This can be done by editing the `/etc/sysconfig/irqbalance` file.

1. Open `/etc/sysconfig/irqbalance` in your preferred text editor and find the section of the file titled `FOLLOW_ISOLCPUS`.

   ```
   ...[output truncated]...
   # FOLLOW_ISOLCPUS
   #       Boolean value.  When set to yes, any setting of IRQ_AFFINITY_MASK above
   #       is overridden, and instead computed to be the same mask that is defined
   #       by the isolcpu kernel command line option.
   #
   #FOLLOW_ISOLCPUS=no

   ```

2. Enable FOLLOW_ISOLCPUS by removing the `#` character from the beginning of the line and changing the value to *yes*.

   ```
   ...[output truncated]...
   # FOLLOW_ISOLCPUS
   #       Boolean value.  When set to yes, any setting of IRQ_AFFINITY_MASK above
   #       is overridden, and instead computed to be the same mask that is defined
   #       by the isolcpu kernel command line option.
   #
   FOLLOW_ISOLCPUS=yes

   ```

3. This will make `irqbalance` operate only on the CPUs not specifically isolated. This has no effect on machines with only two processors, but will run effectively on a dual-core machine.

**Manually Assigning CPU Affinity to Individual IRQs**

1. Check which IRQ is in use by each device by viewing the `/proc/interrupts` file:

   ```
   # cat /proc/interrupts

   ```

   This file contains a list of IRQs. Each line shows the IRQ number, the number of interrupts that happened in each CPU, followed by the IRQ type and a description:

   ```
   CPU0             CPU1
   0:   26575949         11         IO-APIC-edge  timer
   1:         14          7         IO-APIC-edge  i8042
   ...[output truncated]...

   ```

2. To instruct an IRQ to run on only one processor, `echo` the CPU mask (as an hexadecimal number) to `/proc/interrupts`. In this example, we are instructing the interrupt with IRQ number 142 to run on CPU 0 only:

   ```
   # echo 1 > /proc/irq/142/smp_affinity

   ```

3. This change will only take effect once an interrupt has occurred. To test the settings, generate some disk activity, then check the `/proc/interrupts` file for changes. Assuming that you have caused an interrupt to occur, you should see that the number of interrupts on the chosen CPU have risen, while the numbers on the other CPUs have not changed.

**Binding Processes to CPUs using the taskset utility**

The `taskset` utility uses the process ID (PID) of a task to view or set the affinity, or can be used to launch a command with a chosen CPU affinity. In order to set the affinity, `taskset` requires the CPU mask expressed as a decimal or hexadecimal number.

1. To set the affinity of a process that is not currently running, use `taskset` and specify the CPU mask and the process. In this example, `my_embedded_process` is being instructed to use only CPU 3 (using the decimal version of the CPU mask).

   ```
   # taskset 8 /usr/local/bin/my_embedded_process

   ```

2. It is also possible to set the CPU affinity for processes that are already running by using the `-p` (`--pid`) option with the CPU mask and the PID of the process you wish to change. In this example, the process with a PID of 7013 is being instructed to run only on CPU 0.

   ```
   # taskset -p 1 7013
   ```