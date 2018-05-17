# 配置Oz

oz默认生成的镜像为raw格式，我们切换为qcow2。qcow2 格式的文件虽然在性能上比Raw 格式的有一些损失（主要体现在对于文件增量上，qcow2 格式的文件为了分配 cluster 多花费了一些时间），但是 qcow2 格式的镜像比 Raw 格式文件更小，只有在虚拟机实际占用了磁盘空间时，其文件才会增长，能方便的减少迁移花费的流量，更适用于云计算系统，同时，它还具有加密，压缩，以及快照等 raw 格式不具有的功能。

```
# vim /etc/oz/oz.cfg
[paths]
output_dir = /var/lib/libvirt/images
data_dir = /var/lib/oz
screenshot_dir = /var/lib/oz/screenshots
# sshprivkey = /etc/oz/id_rsa-icicle-gen

[libvirt]
uri = qemu:///system
image_type = qcow2
#image_type = raw
# type = kvm
# bridge_name = virbr0
# cpus = 1
# memory = 1024

[cache]
original_media = yes
modified_media = no
jeos = no

[icicle]
safe_generation = no
```

Oz 配置介绍：<https://github.com/clalancette/oz/wiki/oz-install>