# Установка и настройка кластера Redis в Linux (Centos8)

## Описание Nodes

В таблице ниже описаны узлы, для которых я буду поднимать кластер Redis      

| **Nodes name**          | **Nodes IP**  |
| ----------------------- | ------------- |
| **kaa_conf** (сервер1)  | 192.168.3.221 |
| **kaa_conf2** (сервер2) | 192.168.3.222 |
| **kaa_conf3** (сервер3) | 192.168.3.223 |



## Установка

Установка Redis:    

```
yum install redis -y
```



## Настройка

В данной инструкции каждый master будет подключен к одному slave.   

Официальная документация рекомендует использовать использовать  6 nodes (3 Masters и 3 Slaves).   

Схема  подлючения.

![image](https://user-images.githubusercontent.com/52449706/118400757-a7ecc500-b66b-11eb-8ac5-7efc12fb73b7.png)

Я в установке буду использовать 3 сервера, , на каждом из которых запущено по два экземпляра Redis (Master+Slave, не зависимых друг от друга).

###  На сервер1:   

Копируем конфиги    

```
cp /etc/redis.conf /etc/redis_shard1.conf 
cp /etc/redis.conf /etc/redis_replica2.conf 
mv /etc/redis.conf /etc/redis.bak
chown redis /etc/redis*
```

Cоздание сервиса

```
cp /usr/lib/systemd/system/redis.service  /usr/lib/systemd/system/redis-shard1.service
```

```
[Service]
ExecStart=/usr/bin/redis-server /etc/redis_shard1.conf  --supervised systemd
ExecStop=/usr/libexec/redis-shutdown
```

```
cp /usr/lib/systemd/system/redis.service  /usr/lib/systemd/system/redis-replica2.service
```

```
[Service]
ExecStart=/usr/bin/redis-server /etc/redis_replica2.conf   --supervised systemd
ExecStop=/usr/libexec/redis-shutdown
```

**mcedit /etc/redis_shard1.conf** 

```
# bind 127.0.0.1
protected-mode no
port 6379
pidfile /var/run/redis_6379.pid
logfile /var/log/redis/redis_shard1.log
dbfilename dump_shard1.rdb
cluster-enabled yes
cluster-config-file nodes-6379.conf
cluster-node-timeout 15000
```

**mcedit /etc/redis_replica2.conf** 

```
# bind 127.0.0.1
protected-mode no
port 6380
pidfile /var/run/redis_6380.pid
logfile /var/log/redis/redis_replica2.log
dbfilename dump_replica2.rdb
cluster-enabled yes
cluster-config-file nodes-6380.conf
cluster-node-timeout 15000
```

**Firewall**

```
firewall-cmd --permanent --add-port=6379/tcp
firewall-cmd --permanent --add-port=6380/tcp
firewall-cmd --permanent --add-port=16379/tcp
firewall-cmd --permanent --add-port=16380/tcp
firewall-cmd --reload
```



### На сервер2

```
cp /etc/redis.conf /etc/redis_shard2.conf 
cp /etc/redis.conf /etc/redis_replica3.conf 
mv /etc/redis.conf /etc/redis.bak
chown redis /etc/redis*
```

**Cоздание сервиса**

```
cp /usr/lib/systemd/system/redis.service  /usr/lib/systemd/system/redis-shard2.service
```

```
[Service]
ExecStart=/usr/bin/redis-server /etc/redis_shard2.conf  --supervised systemd
ExecStop=/usr/libexec/redis-shutdown
```

```
cp /usr/lib/systemd/system/redis.service  /usr/lib/systemd/system/redis-replica3.service
```

```
[Service]
ExecStart=/usr/bin/redis-server /etc/redis_replica3.conf   --supervised systemd
ExecStop=/usr/libexec/redis-shutdown
```

**mcedit /etc/redis_shard2.conf** 

```
# bind 127.0.0.1
protected-mode no
port 6380
pidfile /var/run/redis_6380.pid
logfile /var/log/redis/redis_shard2.log
dbfilename dump_shard2.rdb
cluster-enabled yes
cluster-config-file nodes-6380.conf
cluster-node-timeout 15000
```

**mcedit /etc/redis_replica3.conf** 

```
# bind 127.0.0.1
protected-mode no
port 6381
pidfile /var/run/redis_6381.pid
logfile /var/log/redis/redis_replica3.log
dbfilename dump_replica3.rdb
cluster-enabled yes
cluster-config-file nodes-6381.conf
cluster-node-timeout 15000
```

**Firewall**

```
firewall-cmd --permanent --add-port=6380/tcp
firewall-cmd --permanent --add-port=6381/tcp
firewall-cmd --permanent --add-port=16380/tcp
firewall-cmd --permanent --add-port=16381/tcp
firewall-cmd --reload
```

### На сервер3

```
cp /etc/redis.conf /etc/redis_shard3.conf 
cp /etc/redis.conf /etc/redis_replica1.conf 
mv /etc/redis.conf /etc/redis.bak
chown redis /etc/redis*
```

**Cоздание сервиса**

```
cp /usr/lib/systemd/system/redis.service  /usr/lib/systemd/system/redis-shard3.service
```

```
[Service]
ExecStart=/usr/bin/redis-server /etc/redis_shard3.conf  --supervised systemd
ExecStop=/usr/libexec/redis-shutdown
```

```
cp /usr/lib/systemd/system/redis.service  /usr/lib/systemd/system/redis-replica1.service
```

```
[Service]
ExecStart=/usr/bin/redis-server /etc/redis_replica1.conf   --supervised systemd
ExecStop=/usr/libexec/redis-shutdown
```

**mcedit /etc/redis_shard3.conf** 

```
# bind 127.0.0.1
protected-mode no
port 6381
pidfile /var/run/redis_6381.pid
logfile /var/log/redis/redis_shard3.log
dbfilename dump_shard3.rdb
cluster-enabled yes
cluster-config-file nodes-6381.conf
cluster-node-timeout 15000
```

**mcedit /etc/redis_replica1.conf** 

```
# bind 127.0.0.1
protected-mode no
port 6379
pidfile /var/run/redis_6379.pid
logfile /var/log/redis/redis_replica1.log
dbfilename dump_replica1.rdb
cluster-enabled yes
cluster-config-file nodes-6379.conf
cluster-node-timeout 15000
```

**Firewall**

```
firewall-cmd --permanent --add-port=6381/tcp
firewall-cmd --permanent --add-port=6379/tcp
firewall-cmd --permanent --add-port=16381/tcp
firewall-cmd --permanent --add-port=16379/tcp
firewall-cmd --reload
```

### Таблица сервер-порт

| Server                  | Shard              | Replica              |
| ----------------------- | ------------------ | -------------------- |
| Server1 - 192.168.3.221 | Shard1 (порт 6379) | Replica2 (порт 6380) |
| Server2 - 192.168.3.222 | Shard2 (порт 6380) | Replica3 (порт 6381) |
| Server3 - 192.168.3.223 | Shard3 (порт 6381) | Replica1 (порт 6379) |





### Запуск узлов Master и Slave:

**systemctl  daemon-reload**

|                                        |                                                              |
| -------------------------------------- | ------------------------------------------------------------ |
| **На сервер1**                         |                                                              |
| redis-server /etc/redis_shard1.conf    | systemctl start redis-shard1.service                         |
| redis-server  /etc/redis_replica2.conf | systemctl start redis-replica2.service                       |
|                                        | systemctl enable  redis-shard1.service     systemctl enable redis-replica2.service |
| **На сервер2**                         |                                                              |
| redis-server /etc/redis_shard2.conf    | systemctl start redis-shard2.service                         |
| redis-server  /etc/redis_replica3.conf | systemctl start redis-replica3.service                       |
|                                        | systemctl enable  redis-shard2.service      systemctl enable  redis-replica3.service |
| **На сервер3**                         |                                                              |
| redis-server /etc/redis_shard3.conf    | systemctl start redis-shard3.service                         |
| redis-server  /etc/redis_replica1.conf | systemctl start redis-replica1.service                       |
|                                        | systemctl enable  redis-shard3.service     systemctl enable redis-replica1.service |



### Создание кластера с использованием встроенного скрипта Ruby  



|                                 |                   |
| ------------------------------- | ----------------- |
| Установка Ruby                  | yum install ruby  |
| Установите пакет Redis для Ruby | gem install redis |



### Добавление узлов Master   

 Чтобы запустить скрипт, перейдите в каталог, где находится исходный код Redis и выполните настройку серверов кластера, передав список пар ip:port серверов, которые будут играть роль master:

```
redis-cli --cluster create 192.168.3.221:6379 192.168.3.222:6380 192.168.3.223:6381
```

Теперь мы можем получить все связанные с кластером узлы с помощью redis-cli. Флаг -c определяет соединение с кластером:

```
redis-cli -c -h 192.168.3.221 -p 6379
```

### Добавление узлов Slave

Для добавления новых узлов в кластер опять воспользуемся утилитой redis-trib. Для присоединения сервера slave к master будем использовать команду:

**redis-cli --cluster add-node*%IP-Slave:порт%* *%IP-Master:порт%* --cluster-slave --cluster-master-id *%ID-мастера%***

|                                    |                                                              |
| ---------------------------------- | ------------------------------------------------------------ |
| Добавления Slave к первому master  | redis-cli --cluster  add-node 192.168.3.223:6379 192.168.3.221:6379 --cluster-slave --cluster-master-id a0d5d543abd25afa068cb9e250f80c4668f14fb9 |
| Добавления Slave ко второму master | redis-cli --cluster  add-node 192.168.3.221:6380 192.168.3.222:6380--cluster-slave --cluster-master-id 0868bc8ede4375f8e3d260422dbda9bd29771555 |
| Добавления Slave к третьему master | redis-cli --cluster  add-node 192.168.3.222:6381 192.168.3.223:6381 --cluster-slave --cluster-master-id a0d5d543abd25afa068cb9e250f80c4668f14fb9 |
