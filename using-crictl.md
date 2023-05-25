
## Troubleshooting containers/pods using crictl

- List all containers on node
```
>> crictl ps -a
CONTAINER           IMAGE               CREATED             STATE               NAME                      ATTEMPT             POD ID              POD
7894eae7762a0       8c811b4aec35f       29 hours ago        Running             shell                     0                   238bdc6ab73bb       nginx
e0dbb1169cfd0       a7be6198544f0       29 hours ago        Running             nginx                     0                   238bdc6ab73bb       nginx
5990eb6968004       8c811b4aec35f       4 weeks ago         Running             debugger-qjn5z            0                   ea63f34e34fd2       ephemeral-demo
```

- List all pods on node
```
>> crictl pods
POD ID              CREATED             STATE               NAME                                         NAMESPACE            ATTEMPT             RUNTIME
238bdc6ab73bb       29 hours ago        Ready               nginx                                        default              0                   (default)
ea63f34e34fd2       4 weeks ago         Ready               ephemeral-demo                               default              0                   (default)
4bd323bad37f0       4 weeks ago         Ready               coredns-787d4945fb-dc94p                     kube-system          0                   (default)
```

- Inspect container/pod, it gives information about cgroups path, linux capabilities, seccomp, Apparmor
```
# Inspect container
>> crictl inspect <container-id>
{
  "status": {
    "id": "7894eae7762a0dcb6fd78f198ab0ac7fc606adb875f91f0d3851c3ef0e37b7ab",
    "metadata": {
      "attempt": 0,
      "name": "shell"
    },
.......(truncated output)

# Inspect pod
>> crictl inspectp <pod-id>
{
  "status": {
    "id": "238bdc6ab73bb24b6a81e66191a60c9c9f202eb6116c6079921d3391976de499",
    "metadata": {
      "attempt": 0,
      "name": "nginx",
      "namespace": "default",
      "uid": "3113e3bc-9f79-483a-9a23-c5968d2f840d"
    },
.......(truncated output)
   
```
Note `crictl inspect` pod/container both gives different output and sometimes both are required.
 
-  Display Status & Configuration Information about Containerd & The CRI Plugin, gives information about CNI status, runtime status, CNI config, containerd state and root dir.
```
>> crictl info
{
  "status": {
    "conditions": [
      {
        "type": "RuntimeReady",
        "status": true,
        "reason": "",
        "message": ""
      },
      {
        "type": "NetworkReady",
 ......(truncated output)

```
- Displays information about containerd image file system
```
>> crictl imagefsinfo
{
  "status": {
    "timestamp": "1685013799612950482",
    "fsId": {
      "mountpoint": "/var/lib/containerd/io.containerd.snapshotter.v1.overlayfs"
    },
    "usedBytes": {
      "value": "1039409152"
    },
    "inodesUsed": {
      "value": "24370"
    }
  }
}

```
- Exec into a running container using crictl
```
>> crictl exec -it <container-id> <command>
e.g.
>> crictl exec -it 7894eae7762a0 /bin/sh
bin           home          product_uuid  tmp
dev           proc          root          usr
etc           product_name  sys           var

```
- Display Stats for the Containers
```
>> crictl stats
CONTAINER           NAME                      CPU %               MEM                 DISK                INODES
0290ce519931c       coredns                   0.27                16.74MB             53.25kB             16
0f71a0c254765       kindnet-cni               0.17                10.66MB             73.73kB             26
1dd4139b16040       kube-scheduler            0.33                22.22MB             20.48kB             9
57df0d2dd46e4       kube-apiserver            4.14                262.7MB             57.34kB             16
5990eb6968004       debugger-qjn5z            0.00                1.663MB             24.58kB             9
```

