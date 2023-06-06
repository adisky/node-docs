# Troubleshooting Kubernetes nodes

## Issues related to node (node unschedulable/hanged/unreachable)

- List kuberentes nodes, It will give node status, kubelet, containerd, OS Image and Kernel version. Check if any known issues related with the particular version.
```
>> kubectl get nodes -o wide
```

- Determine node status, check under conditions if node is under any disk/memory/pid pressure
```
>> kubectl describe node <node-name>
```

- check containerd and kubelet service status
```
>> sudo systemctl status kubelet
>> sudo systemctl status containerd 
```

- collect & check kubernetes and containerd logs for any errors
```
>> journalctl -xeu containerd
>> journalctl -xeu kubelet
```

- check node memory, avg cpu load and disk usage, 
```
>> sudo free -m
>> sudo df -ah --total
>> uptime

```

- monitor the node for sometime with `top` command and check for processes having unusual usage.
>> top -o %MEM
>> top -o %CPU (%CPU usage given for 1 cpu, e.g. if cpu usage is 200% then 2 cores are used fully. average usage would be cpu usage/no. of cores)
>> top -o RES (press E to get memory unit in MB)

- check node process status with `ps -elf`. 
    - Specially focus on containerd-shim processes. If there are too many processes(more than the number of pods on the node) or any process consuming too much cpu/memory. Note the id associated with the containerd-shim process and, its the pod id at containerd level. With `sudo crictl pods` you can know the pod name.

    - If any `runc` process is stuck, ideally runc processes are not long running and shouldn't be consistently visible in `ps -elf` output.

- Dump kubelet and containerd config
```
>> containerd config dump or cat /etc/containerd/config.toml
>> cat /var/lib/kubelet/config.yaml > kubelet-component-config.out
>> cat /var/lib/kubelet/kubeadm-flags.env > kubeadm-flags.out
>> cat /var/lib/kubelet/cpu_manager_state > kubelet-cpu-manager.out
```
## Issues related with Pods (pod stuck, crashloopbackoff, unschedulable)


- Get pod status, check events related 
```
>> kubectl describe pod <pod name>
```
- Get pod status with containerd, crictl inspect gives lots of information applied by container runtime like capabilties, cgroup path, seccomp, apparmor profiles etc.
```
>> sudo crictl pods
>> sudo crictl inspectp <pod-id>
>> sudo crictl ps -a 
>> sudo crictl inspect <container-id>
```

- Pod QoS, memory, limit
- pod capabilites
