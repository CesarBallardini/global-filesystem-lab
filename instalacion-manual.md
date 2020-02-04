# Instalación manual de Global File System 2 sobre Virtualbox y NBD


* creamos las VM `san`, `c1`y `c2`

```bash
cp Vagrantfile.manual Vagrantfile
vagrant up
vagrant reload
```

* en `san` instalamos módulos extras y suite de programas servidor y cliente para NBD; creamos un disco y lo exponemos mediante NBD

```bash
vagrant ssh san

sudo apt-get install debconf-utils -y
sudo apt install linux-modules-extra-$(uname -r) -y
sudo apt-get install xnbd-client xnbd-server -y

sudo dd if=/dev/zero of=/srv/disk.img bs=4096 count=1 seek=1000000
sudo xnbd-server --target --lport 8992 /srv/disk.img


```

* en `c1` y `c2`, instalamos módulos extras para OCFS2, y levantamos el cliente de NBDa; instalamos el paquete de utilidades de OCFS2

```bash
vagrant ssh c1

sudo apt-get install debconf-utils -y
sudo apt install linux-modules-extra-$(uname -r) -y
sudo apt-get install xnbd-client xnbd-server -y

sudo modprobe nbd
echo deadline | sudo tee /sys/block/nbd0/queue/scheduler
sudo xnbd-client bs=4096 192.168.40.13 8992 /dev/nbd0

sudo apt-get install ocfs2-tools -y

```

* en `c1` y `c2`, Configuramos en `/etc/ocfs2/cluster.conf` lo siguiente:

```yaml
cluster:
    node_count = 2
    name = cluster1

node:
    ip_port = 7777
    ip_address = 192.168.40.14
    number = 1
    name = c1
    cluster = cluster1

node:
    ip_port = 7777
    ip_address = 192.168.40.15
    number = 2
    name = c2
    cluster = cluster1
```

* en `c1` y `c2`, Configuramos en `/etc/default/o2cb` lo siguiente:

```bash
O2CB_ENABLED=true
O2CB_BOOTCLUSTER=cluster1
O2CB_HEARTBEAT_THRESHOLD=61
O2CB_IDLE_TIMEOUT_MS=30000
O2CB_KEEPALIVE_DELAY_MS=2000
O2CB_RECONNECT_DELAY_MS=2000
```

* reiniciamos configuración de OCFS2

```bash
sudo service o2cb start
sudo service o2cb load
sudo service o2cb online
```


* en `c1` formatear el disco

```bash
sudo mkfs.ocfs2 /dev/nbd0
```


* en `c1`y `c2` montar el disco

```bash
sudo mkdir /mnt/ocfs2
sudo mount -t ocfs2 /dev/nbd0 /mnt/ocfs2
```


