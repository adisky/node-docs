# Troubleshooting containerd

- Check the containerd service status
```
>> sudo systemctl status containerd
● containerd.service - containerd container runtime
     Loaded: loaded (/etc/systemd/system/containerd.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2023-04-24 11:05:09 UTC; 1 month 0 days ago
       Docs: https://containerd.io
   Main PID: 236 (containerd)
      Tasks: 167
     Memory: 676.4M

```

- Collect containerd logs and look for errors and failures.
```
>> journalctl -xeu containerd >> containerd.log

```
- Enable debug logs in containerd config, containerd config is usually located at `/etc/containerd/config.toml`. Edit the file and and make `level = debug`. Add debug section if not present
```
[debug]
  level = “debug”
```

-  Compare the default config with the config placed in `/etc/containerd/config.toml`, to know the specifice configuration changed. You can get default config with following command
```
>> containerd config default >> containerd-default-config.toml
```

- Check the number of `containerd-shim` processes on the node
```
>> ps -ef| grep "containerd-shim"
root        1354       1  0 Apr24 ?        00:11:52 /usr/local/bin/containerd-shim-runc-v2 -namespace k8s.io -id 4bd323bad37f09489f65b36a3d6f7819bfed15a09ab99627b734bbbfc4564eb0 -address /run/containerd/containerd.sock
root        2104       1  0 Apr24 ?        00:21:21 /usr/local/bin/containerd-shim-runc-v2 -namespace k8s.io -id ea63f34e34fd28951ac51576684f2c295bbd6b22157525ef5d8567ed30459c24 -address /run/containerd/containerd.sock
root      490758       1  0 May24 ?        00:00:53 /usr/local/bin/containerd-shim-runc-v2 -namespace k8s.io -id 238bdc6ab73bb24b6a81e66191a60c9c9f202eb6116c6079921d3391976de499 -address /run/containerd/containerd.sock
```
compare it with the number of pods running on the node, ideally it should be equal

```
>> crictl ps -a
POD ID              CREATED             STATE               NAME                                         NAMESPACE            ATTEMPT             RUNTIME
238bdc6ab73bb       29 hours ago        Ready               nginx                                        default              0                   (default)
ea63f34e34fd2       4 weeks ago         Ready               ephemeral-demo                               default              0                   (default)
4bd323bad37f0       4 weeks ago         Ready               coredns-787d4945fb-dc94p                     kube-system          0                   (default)
```
You can also see which shim process is connected to particular container using process tree.
```
>> ps aux --forest
root         375  0.0  0.0 720780 10108 ?        Sl   Apr24  11:45 /usr/local/bin/containerd-shim-runc-v2 -namespace k8s.io -id 10699e6860bdd461c560a42074897f5594e127f47202f49e8c487914feab958f -address /run/containerd/containerd.sock
65535        471  0.0  0.0    992     4 ?        Ss   Apr24   0:00  \_ /pause
root         709  2.4  0.3 11215932 64484 ?      Ssl  Apr24 1110:20  \_ etcd --advertise-client-urls=https://172.18.0.2:2379 --cert-file=/etc/kubernetes/pki/etcd/server.crt --client-cert-auth=true --data-dir=/var/lib/etcd --experimental-initial-corrupt-c
root         383  0.0  0.0 720780 10420 ?        Sl   Apr24  11:32 /usr/local/bin/containerd-shim-runc-v2 -namespace k8s.io -id 5130859ffbc0e2b775d56f5c5b129c153317b572a1d76b7c590fd964a5c02cfa -address /run/containerd/containerd.sock
65535        480  0.0  0.0    992     4 ?        Ss   Apr24   0:00  \_ /pause
root         637  4.4  1.8 1069700 303460 ?      Ssl  Apr24 1968:05  \_ kube-apiserver --advertise-address=172.18.0.2 --allow-privileged=true --authorization-mode=Node,RBAC --client-ca-file=/etc/kubernetes/pki/ca.crt --enable-admission-plugins=NodeRestri

- check disk usage of containerd state and root dir
```
>> du -sh /var/lib/containerd/
1.4G	/var/lib/containerd/
>> du -sh /run/containerd
930M	/run/containerd
```


