# swarm开发笔记

### Server端TLS链接设置

1. auto-tls-cert.sh

   ```sh
   #!/bin/bash
   # 
   # -------------------------------------------------------------
   # 自动创建 Docker TLS 证书
   # -------------------------------------------------------------
   
   # 以下是配置信息
   # --[BEGIN]------------------------------
   
   CODE="fnfn"
   IP="192.168.56.101"
   PASSWORD="fnfn9527"
   COUNTRY="CN"
   STATE="YUNNAN"
   CITY="KUNMING"
   ORGANIZATION="FnFn"
   ORGANIZATIONAL_UNIT="Dev"
   COMMON_NAME="$IP"
   EMAIL="sfme@qq.com"
   
   # --[END]--
   
   # Generate CA key
   openssl genrsa -aes256 -passout "pass:$PASSWORD" -out "ca-key-$CODE.pem" 4096
   # Generate CA
   openssl req -new -x509 -days 365 -key "ca-key-$CODE.pem" -sha256 -out "ca-$CODE.pem" -passin "pass:$PASSWORD" -subj "/C=$COUNTRY/ST=$STATE/L=$CITY/O=$ORGANIZATION/OU=$ORGANIZATIONAL_UNIT/CN=$COMMON_NAME/emailAddress=$EMAIL"
   # Generate Server key
   openssl genrsa -out "server-key-$CODE.pem" 4096
   
   # Generate Server Certs.
   openssl req -subj "/CN=$COMMON_NAME" -sha256 -new -key "server-key-$CODE.pem" -out server.csr
   
   echo "subjectAltName = IP:$IP,IP:127.0.0.1" >> extfile.cnf
   echo "extendedKeyUsage = serverAuth" >> extfile.cnf
   
   openssl x509 -req -days 365 -sha256 -in server.csr -passin "pass:$PASSWORD" -CA "ca-$CODE.pem" -CAkey "ca-key-$CODE.pem" -CAcreateserial -out "server-cert-$CODE.pem" -extfile extfile.cnf
   
   
   # Generate Client Certs.
   rm -f extfile.cnf
   
   openssl genrsa -out "key-$CODE.pem" 4096
   openssl req -subj '/CN=client' -new -key "key-$CODE.pem" -out client.csr
   echo extendedKeyUsage = clientAuth >> extfile.cnf
   openssl x509 -req -days 365 -sha256 -in client.csr -passin "pass:$PASSWORD" -CA "ca-$CODE.pem" -CAkey "ca-key-$CODE.pem" -CAcreateserial -out "cert-$CODE.pem" -extfile extfile.cnf
   
   rm -vf client.csr server.csr
   
   chmod -v 0400 "ca-key-$CODE.pem" "key-$CODE.pem" "server-key-$CODE.pem"
   chmod -v 0444 "ca-$CODE.pem" "server-cert-$CODE.pem" "cert-$CODE.pem"
   
   # 打包客户端证书
   mkdir -p "tls-client-certs-$CODE"
   cp -f "ca-$CODE.pem" "cert-$CODE.pem" "key-$CODE.pem" "tls-client-certs-$CODE/"
   cd "tls-client-certs-$CODE"
   tar zcf "tls-client-certs-$CODE.tar.gz" *
   mv "tls-client-certs-$CODE.tar.gz" ../
   cd ..
   rm -rf "tls-client-certs-$CODE"
   
   # 拷贝服务端证书
   mkdir -p /etc/docker/certs.d
   cp "ca-$CODE.pem" "server-cert-$CODE.pem" "server-key-$CODE.pem" /etc/docker/certs.d/
   ```
   
2. 源码：[client/options.go#L26](https://github.com/moby/moby/blob/3042254a87274ff5e9561f2da1a986a703dfc60f/client/options.go#L26)

3. 修改服务

   ```sh
   systemctl start docker.service
   ```

   ```sh
   vim /lib/systemd/system/docker.service
   ```

   **ExecStar**t属性中加入：`--tlsverify --tlscacert=/etc/docker/certs.d/ca-fnfn.pem --tlscert=/etc/docker/certs.d/server-cert-fnfn.pem --tlskey=/etc/docker/certs.d/server-key-fnfn.pem -H tcp://0.0.0.0:2376`

   ```sh
   sudo systemctl daemon-reload
   sudo systemctl restart docker.service
   ```

### Client端TLS连接设置

1. 环境变量
  
   ```sh
   $ env | grep DOCKER
   $ eval "$(docker-machine env dev)"
   $ env | grep DOCKER
   # 客户端访问需要host
   DOCKER_HOST=tcp://192.168.99.101:2376
   # 客户端反问需要cert
   DOCKER_CERT_PATH=/Users/nathanleclaire/.docker/machines/.client
   DOCKER_TLS_VERIFY=1
   DOCKER_MACHINE_NAME=dev
   $ # If you run a docker command, now it runs against that host.
   $ eval "$(docker-machine env -u)"
   $ env | grep DOCKER
   $ # The environment variables have been unset.
   ```

2. 客户端主机测试

   ```sh
   docker --tlsverify --tlscacert=ca-fnfn.pem --tlscert=cert-fnfn.pem --tlskey=key-fnfn.pem -H 192.168.56.101:2376 version
   ```

3. baas
   ```sh
   baas-service swarm --conf=swarm.toml
   ```

4. swarm task状态：PREPARING、RUNNING、FAILED、COMPLETE、REJECTED

5. 部署超级节点：
   ```sh
   docker --tlsverify --tlscacert=/Users/gaochun/.docker/machines/.client/ca.pem --tlscert=/Users/gaochun/.docker/machines/.client/cert.pem --tlskey=/Users/gaochun/.docker/machines/.client/key.pem -H 192.168.56.101:2376 stack deploy -c docker-supernode.yml fnfn
   ```
6. Swarm中Container(Task)->Service->Stack  
   * docker swarm：集群管理，子命令有init, join, leave, update。（docker swarm --help查看帮助）
   * docker service：服务创建，子命令有create, inspect, update, remove, tasks。（docker service--help查看帮助）
   * docker node：节点管理，子命令有accept, promote, demote, inspect, update, tasks, ls, rm。（docker node --help查看帮助）

   * node是加入到swarm集群中的一个docker引擎实体，可以在一台物理机上运行多个node，node分为：
     - manager node管理节点：执行集群的管理功能，维护集群的状态，选举一个leader节点去执行调度任务。
     - worker node工作节点：接收和执行任务。参与容器集群负载调度，仅用于承载task。


   * service服务：一个服务是工作节点上执行任务的定义。创建一个服务，指定了容器所使用的镜像和容器运行的命令。service是运行在worker nodes上的task的描述，service的描述包括使用哪个docker 镜像，以及在使用该镜像的容器中执行什么命令。
   * task任务：一个任务包含了一个容器及其运行的命令。task是service的执行实体，task启动docker容器并在容器中执行任务。