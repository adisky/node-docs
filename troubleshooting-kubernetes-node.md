# Troubleshooting Kubernetes nodes

## Issues related to node (node unschedulable/hanged/unreachable)

- List kuberentes nodes, It will give node status, kubelet, containerd, OS Image and Kernel version. Check if any known issues related with the particular version.
```
>> kubectl get nodes -o wide
NAME                 STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
kind-control-plane   Ready    control-plane   42d   v1.26.3   172.18.0.2    <none>        Ubuntu 22.04.2 LTS   5.4.0-1103-gcp   containerd://1.6.19-46-g941215f49

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
(it gives cpu load avg for las 1min, 5min, 15min e.g. load average: 0.49, 0.30, 0.22
the output is in terms of 1 cpu, to get overall avg for multicore in load-avg*100/no. of cores )
```

- monitor the node for sometime with `top` command and check for processes having unusual usage.
```
>> top -o %MEM
>> top -o %CPU 
(%CPU usage given for 1 cpu, e.g. if cpu usage is 200% then 2 cores are used fully. average usage would be cpu usage/no. of cores)
>> top -o RES (press E to get memory unit in MB)
```

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


- Get pod status, check related events 
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
- Get pod logs
```
>> kubectl logs -c <container-name> <pod-name> 
>> kubectl logs --previous <pod-name>
(--previous is used if pod is crashing and not able to get any running containers log, it will fetch logs from failed pod)
```
- exec into running containers  
```
>> kubectl exec -it  <pod-name> -c <container-name> -- /bin/sh
```

