# 在Kubernetes(K8s)上搭建基于Zookeeper的Flink高可用集群(HA)
***

### 版本

- Flink-1.7.2
- Hadoop-2.6.0
- Docker	18.03.1-ce
- K8s	v1.10.4
- 系统-CentOS 7

### 非HA搭建
该过程Flink官网有提供，也比较简单，可以[点击查看](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/deployment/kubernetes.html)，也可以参考[我这里的配置文件](https://github.com/linweijiang/Flink-K8s/tree/master/flink-standalone)

### Flink HA搭建
- 在搭建之前，强烈建议先熟悉一下[非K8s时正常的Flink HA搭建的过程](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/jobmanager_high_availability.html)，主要知道要修改/添加哪些配置文件。也可以参考[中文版](http://flink-cn.shinonomelab.com/ops/jobmanager_high_availability.html)的

- 说明一下：

官方没有提供在K8s上搭建基于ZK的HA的镜像的脚本文件，所以我们需要自己修改构建Flink镜像的相关配置。我们可以 参考/借用 官方提供Flink非HA的构建脚本。根据需要搭建的版本，我们以这个[目录](https://github.com/docker-flink/docker-flink/tree/master/1.7/hadoop26-scala_2.12-alpine)作为我们添加HA配置的基础。

- 开始操作：

修改该目录下的`docker-entrypoint.sh`，添加如下配置（可以做一下非空判断，下面就直接echo了）：

```
echo "high-availability: ${HIGH_AVAILABILITY}" >> "${CONF_FILE}"
echo "high-availability.zookeeper.quorum: ${HIGH_AVAILABILITY_ZOOKEEPER_QUORUM}" >> "${CONF_FILE}"
echo "high-availability.zookeeper.path.root: ${HIGH_AVAILABILITY_ZOOKEEPER_PATH_ROOT}" >> "${CONF_FILE}"
echo "high-availability.cluster-id: ${HIGH_AVAILABILITY_CLUSTER_ID}" >> "${CONF_FILE}"
echo "high-availability.storageDir: ${HIGH_AVAILABILITY_STORAGEDIR}" >> "${CONF_FILE}"
```

修改后的`docker-entrypoint.sh`可以参考[我这里的](https://github.com/linweijiang/Flink-K8s/blob/master/docker/docker-entrypoint.sh)

然后通过该目录下的Dockerfile文件构建属于自己的镜像（这里就过多介绍构建过程啦），我构建出来的镜像名：leen/flink-ha:v1，在hub.docker上可以拉取到

然后部署用这[两个配置文件](https://github.com/linweijiang/Flink-K8s/tree/master/flink-standalone-ha)执行下面命令就可以搭建Flink HA集群了

```
kubectl apply -f jobmanager-statefulset.yaml -f taskmanager-deployment.yaml
```
