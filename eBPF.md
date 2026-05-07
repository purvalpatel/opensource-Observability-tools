Technology in linux that lets you run small programs inside the kernel safely.
like,
- Java script for linux kernel.
- Plugins for the operating system.

If you want to:
- Monitor network traffic
- Trace processes
- Observe containers
- inspect syscalls
- Enforce Security

You have to:
- Modify kernela code
- Load Risky kernel modules
- Reboot servers

**eBPF changed that.**


### What eBPF can observe?
It can hook into:
- Network packets
- file access
- Sys calls
- Process execution
- TCP connections
- Kubernetes networking
- container events

When process opens file:
```
Process
   |
Open () syscall
   |
Kernel
```
eBPF can intercept event and: `log, block, count, trace latency`.

## Popular tools using ePDF
| Tool                                                                    | Use                            |
| ----------------------------------------------------------------------- | ------------------------------ |
| [Cilium](https://cilium.io?utm_source=chatgpt.com)                      | Kubernetes networking/security |
| [Hubble](https://github.com/cilium/hubble?utm_source=chatgpt.com)       | Network observability          |
| [Pixie](https://px.dev?utm_source=chatgpt.com)                          | K8s observability              |
| [Falco](https://falco.org?utm_source=chatgpt.com)                       | Runtime security               |
| [bcc](https://github.com/iovisor/bcc?utm_source=chatgpt.com)            | eBPF tracing tools             |
| [bpftrace](https://github.com/bpftrace/bpftrace?utm_source=chatgpt.com) | Linux tracing                  |

# Use case:

### Security example:
eBPF can detect:
```
container trying to access sensitive files
```
or:
```
unexpected shell spawned inside container
```




# Easiest way to start

Install bpftrace
```Ubuntu/Debian
sudo apt update
sudo apt install bpftrace
```

1. Trace every process execution:
```
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_execve { printf("%s\n", comm); }'
```
Open another terminal and run any command, you will see the traces.

2. Monitor file opens
```
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_openat { printf("%s opened a file\n", comm); }'
```

3. Pixie
- API tracing
- HTTP monitoring
- SQL visibility
- Kubernetes observability

without changing application code.

4. Falco
Detects:
- shell inside container
- crypto miners
- suspicious syscalls
- privilege escalation

using eBPF.
