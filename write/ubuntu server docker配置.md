### 1604 `/etc/network/interfaces`

```
# ifupdown has been replaced by netplan(5) on this system.  See
# /etc/netplan for current configuration.
# To re-enable ifupdown on this system, you can run:
#    sudo apt install ifupdown

auto enp0s8
iface enp0s8 inet static
address 192.168.56.101
netmask 255.255.255.0
broadcast 192.168.56.255

auto enp0s9
iface enp0s9 inet static
address 192.168.199.58 
netmask 255.255.255.0
broadcast 192.168.199.255
```

### 1804 `/etc/netplan/50-cloud-init.yaml`

```yaml
# This file is generated from information provided by
# the datasource.  Changes to it will not persist across an instance.
# To disable cloud-init's network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        enp0s3:
            dhcp4: true
        enp0s8:
            dhcp4: no
            addresses: [192.168.56.101/24]
        enp0s9:
            dhcp4: no
            addresses: [192.168.199.62/24]
    version: 2
```

### docker用户权限

```sh
#docker用户设置
sudo usermod -aG docker node02
```

```sh
#初始化主节点
docker swarm init --advertise-addr 192.168.56.101
```
```sh
docker info
```
```sh
docker node ls
```
```sh
#获取集群加入命令
docker swarm join-token worker
```
```sh
#加入集群
docker swarm join --token SWMTKN-1-0xcmbjog617f6996pvpe9obklsfep31sjduahkv61ld1nrabt9-5u7ov7cwy1zwxnqzq06f1vpcu 192.168.56.101:2377
```

```sh
#创建服务并运行
docker service create --replicas 1 --name helloworld alpine ping docker.com
```
```sh
docker service ps helloworld
```
```sh
#扩容
docker service scale helloworld=5
```
```sh
#删除服务
docker service rm helloworld
```

```yaml
version: '3'

services:
  influx:
    image: influxdb
    volumes:
      - influx:/var/lib/influxdb
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == manager

  grafana:
    image: grafana/grafana
    ports:
      - 0.0.0.0:80:3000
    volumes:
      - grafana:/var/lib/grafana
    depends_on:
      - influx
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == manager

  cadvisor:
    image: google/cadvisor
    hostname: '{{.Node.Hostname}}'
    command: -logtostderr -docker_only -storage_driver=influxdb -storage_driver_db=cadvisor -storage_driver_host=influx:8086
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    depends_on:
      - influx
    deploy:
      mode: global

volumes:
  influx:
    driver: local
  grafana:
    driver: local
```

