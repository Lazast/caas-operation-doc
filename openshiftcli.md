# OpenShiftCLI

## Node节点docker服务灾难恢复

集群运行过程中有时候会出现机器负载过高docker daemon挂掉，或者Node主机节点发生重启，此时再次启动docker服务时往往会发生错误无法重启，我们需要手动重置docker服务使得node节点加入到openshift集群重新提供服务。

```bash
#首先停止docker服务
systemctl stop docker
#卸载docker挂载的目录
umount /var/lib/docker/devicemapper
umount /var/lib/docker/containers
#使用docker提供的工具重置配置文件和docker存储池
docker-storage-setup --reset
#检查/etc/sysconfig/docker-storage-setup配置，确保VG=docker-vg写入
ls /etc/sysconfig/docker-storage-setup
#重建docker存储池自动更新配置文件
docker-storage-setup
```

