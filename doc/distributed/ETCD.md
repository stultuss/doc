# ETCD

> etcd v2 升级到 etcd v3 并发性能提升巨大，在 v3 版本中使用了 grpc gateway 来替代了 v2 的 proxy， etcd proxy 则已经被弃用

## 安装

1. 普通安装

```shell
ETCD_VER=v3.4.7

# choose either URL
GOOGLE_URL=https://storage.googleapis.com/etcd
GITHUB_URL=https://github.com/etcd-io/etcd/releases/download
DOWNLOAD_URL=${GOOGLE_URL}
DOWNLOAD_DIR=/tmp/etcd

rm -f ${DOWNLOAD_DIR}/etcd-${ETCD_VER}-linux-amd64.tar.gz
rm -rf ${DOWNLOAD_DIR} && mkdir -p ${DOWNLOAD_DIR}

curl -L ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o ${DOWNLOAD_DIR}/etcd-${ETCD_VER}-linux-amd64.tar.gz
tar xzvf ${DOWNLOAD_DIR}/etcd-${ETCD_VER}-linux-amd64.tar.gz -C ${DOWNLOAD_DIR} --strip-components=1
rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz

${DOWNLOAD_DIR}/etcd --version
${DOWNLOAD_DIR}/etcdctl version
```

2. docker 安装

```shell
# 搭建单节点ETCD服务
docker run -d -p 4001:4001 \
              -p 2380:2380 \
              -p 2379:2379 \
              --name etcd \
              quay.io/coreos/etcd
```

## 创建集群

**服务发现 discovery**

```shell
# 使用公共服务
curl -s https://discovery.etcd.io/new?size=3
#https://discovery.etcd.io/37667965bb9cbc0201d6e7b6a2aab717
```

**创建集群**

```sh
THIS_IP=127.0.0.1
DIR=/tmp/etcd

# For each machine
NAME_1=etcd-node-1
NAME_2=etcd-node-2
NAME_3=etcd-node-3
DISCOVERY=https://discovery.etcd.io/37667965bb9cbc0201d6e7b6a2aab717

# For node 1
THIS_NAME=${NAME_1}
THIS_PORT=12379
PEER_PORT=12380
mkdir -p ${DIR}/${THIS_NAME}
nohup ${DIR}/etcd \
  --data-dir=${DIR}/${THIS_NAME} \
  --name ${THIS_NAME} \
  --advertise-client-urls http://${THIS_IP}:${THIS_PORT} \
  --initial-advertise-peer-urls http://${THIS_IP}:${PEER_PORT} \
  --listen-client-urls http://0.0.0.0:${THIS_PORT} \
  --listen-peer-urls http://0.0.0.0:${PEER_PORT} \
  --discovery ${DISCOVERY} >> ${DIR}/${THIS_NAME}/output.log 2>&1 & echo $! > run_${THIS_NAME}.pid

# For node 2
THIS_NAME=${NAME_2}
THIS_PORT=22379
PEER_PORT=22380
mkdir -p ${DIR}/${THIS_NAME}
nohup ${DIR}/etcd \
  --data-dir=${DIR}/${THIS_NAME} \
  --name ${THIS_NAME} \
  --advertise-client-urls http://${THIS_IP}:${THIS_PORT} \
  --initial-advertise-peer-urls http://${THIS_IP}:${PEER_PORT} \
  --listen-client-urls http://0.0.0.0:${THIS_PORT} \
  --listen-peer-urls http://0.0.0.0:${PEER_PORT} \
  --discovery ${DISCOVERY} >> ${DIR}/${THIS_NAME}/output.log 2>&1 & echo $! > run_${THIS_NAME}.pid

# For node 3
THIS_NAME=${NAME_3}
THIS_PORT=32379
PEER_PORT=32380
mkdir -p ${DIR}/${THIS_NAME}
nohup ${DIR}/etcd \
  --data-dir=${DIR}/${THIS_NAME} \
  --name ${THIS_NAME} \
  --advertise-client-urls http://${THIS_IP}:${THIS_PORT} \
  --initial-advertise-peer-urls http://${THIS_IP}:${PEER_PORT} \
  --listen-client-urls http://0.0.0.0:${THIS_PORT} \
  --listen-peer-urls http://0.0.0.0:${PEER_PORT} \
  --discovery ${DISCOVERY} >> ${DIR}/${THIS_NAME}/output.log 2>&1 & echo $! > run_${THIS_NAME}.pid
```

## Proxy 模式

discovery 方式
```sh
nohup /tmp/etcd/etcd  \
     -name proxy \
     -proxy on  \
     -listen-client-urls http://0.0.0.0:2370  \
     -discovery https://discovery.etcd.io/37667965bb9cbc0201d6e7b6a2aab717 >> /tmp/etcd.log 2>&1 & echo $! > /tmp/run.pid
```

非 discovery 方式
```sh
nohup /tmp/etcd/etcd  \
     -name proxy \
     -proxy on  \
     -listen-client-urls http://0.0.0.0:2370  \
     -initial-cluster 'infra1=http://127.0.0.1:12380' >> /tmp/etcd.log 2>&1 & echo $! > /tmp/run.pid
```

## Gateway 模式

```shell
etcd gateway start \
	--endpoints=http://127.0.0.1:12379,http://127.0.0.1:22379,http://127.0.0.1:32379 \
	--listen-addr=0.0.0.0:80 >> /tmp/etcd-gateway.log 2>&1 &
```

## 验证

```shell
export ETCDCTL_API=3
export ETCD_ENDPOINTS="http://127.0.0.1:80"
etcdctl --endpoints=${ETCD_ENDPOINTS} member list
```

