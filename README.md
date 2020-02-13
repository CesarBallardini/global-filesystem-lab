# README - global-filesystem-lab

Laboratorio para experimentar con cluster filesystems.

Se realiza una prueba de concepto con OCFS2 Oracle Cluster Filesystem v2, 
el dispositivo de bloques que será un disco compartido.

Los pasos siguientes son:

* utilizar una SAN externa, crear una LUN y presentarla a Virtualbox o VMware, para usarla como disco RAW compartido e independiente.


# Laboratorios

## Lab1 - SAN emulada con NBD

El disco ocompartido se proporciona mediante  una VM `san` a través de xNBD (Network Block Device).

Una VM denominada `san` expone un dispositivo NBD imitando el servicio de una
SAN que proporciona una LUN. Dos VMs, denominadas `c1` y `c2` son los clientes que necesitan compartir
un filesystem.  En los clientes se instala el cliente de NBD y se configura OCFS2 en un cluster de dos nodos.

Las direcciones de IP, puertos del NBD, nombres de VM, y nombres de dispositivo 
de bloques están hardcodeadas en el `Vagrantfile`.


Este laboratorio muestra cómo eliminar un cluster de servidores GlusterFS y sustituirlo mediante un dispositivo de bloques
compartido a través de un filesystem de cluster.

Si el dispositivo de bloques es proporcionado por una SAN externa, se aprovechan las capacidades de
replicación y distribución de la SAN para dar confiabilidad al servicio de filesystem compartido.


## Lab2 - Disco compartido externo a las VMs

Se un disco independiente con Virtualbox y se conectarlo a `c1`y `c2`, eliminando así la VM `san` y todo el software de NBD.

En el `Vagrantfile`  se crea el disco y se lo tipifica como `shareable`, si no existe previamente el archivo que lo sostiene.

Cuando la VM se detiene se debe desconectar el disco, de manera que al destruirla no se elimine el disco del sistema.


Se puede entonces levantar las VMs, y se creará el disco, formateado y montado.  Cuando las VMs se detienen o incluso se destruyen
el disco externo no se ve afectado.

Una vez terminadas las pruebas, para eliminar el disco se debe usar el mandato `vboxmanage` para quitarlo del registro de VirtualBox.

```bash
VBoxManage closemedium disk .vagrant/disk_data3.vdi --delete
```



# Cómo usar este repositorio

* Instale Git

```bash
sudo apt install git
```

* Instale Virtualbox

* Instale Vagrant

* Instale plugins de Vagrant:
  + `vagrant-proxyconf` y su configuracion si requiere de un Proxy para salir a Internet
  + `vagrant-cachier`
  + `vagrant-disksize`
  + `vagrant-hostmanager`
  + `vagrant-share`
  + `vagrant-vbguest`


* Clone este repositorio

```bash
git clone https://github.com/CesarBallardini/global-filesystem-lab.git
cd global-filesystem-lab/
```

* para Lab1:

```bash
cp Vagrantfile.auto Vagrantfile
```

* para Lab2:

```bash
cp Vagrantfile.shared-disk Vagrantfile
```

* levante las VMs

```bash
time vagrant up
```

* haga las comprobaciones como se indica más abajo

* detenga las VMs

```bash
vagrant halt
```

* elimine las VMs cuando no las necesite más

```bash
vagrant destroy -f
```

# Verificaciones y comprobaciones

En cada VM se pueden verificar que las partes están funcionando correctamente.

## `san` (en caso de Lab1)

La VM tiene el servidor de xNBD; verifique que el proceso está corriendo.

## `c1`y `c2` (en Lab1 y Lab2)

En ambas VMs debe estar montado un filesystem tipo `ocfs2` en i`/mnt/ocfs2/`.
Cuando en `c1` se escribe en ese directorio, se puede verificar que las modificaciones
se pueden ver desde `c2`.

Además, en caso de Lab1, debe estar corriendo el cliente de NBD, una precondición para que funcione el cluster
OCFS" y que  gracias a eso se pueda haber montado el directorio mencionado.  Cuando se instala por primera vez, 
desde `c1`se procede al formateo del dispositivo  `/dev/nbd0`, lo cual se debe hacer sólo una vez,
pues es una operaci{on a nivel del dispositivo de bloque.


# Apéndice A: Referencias

* http://www.oracle.com/us/technologies/linux/ocfs2-best-practices-2133130.pdf OCFS2 Best Practices Guide  (2014)
* https://bitbucket.org/hirofuchi/xnbd/wiki/Home#!scenario-4-ocfs2-with-xnbd
* https://blogs.oracle.com/cloud-infrastructure/a-simple-guide-to-oracle-cluster-file-system-ocfs2-using-iscsi-on-oracle-cloud-infrastructure OCFS2 + iSCSI
* https://www.howtoinstall.me/ubuntu/18-04/gfs2-utils/
* https://oss.oracle.com/pipermail/ocfs2-users/2012-October/005902.html OCFS2 hanging on writes -- SOLVED
* https://www.mankier.com/8/defragfs.ocfs2 online defragmenter for ocfs2 filesystem
* https://www.mankier.com/7/ocfs2 OCFS2 volumes can be exported as NFS volumes
* https://news.ycombinator.com/item?id=1475232 en 2010, versión 1.4, The main problem is that when you get fragmented too badly you run into space allocation issues where you have tons of free space.
* https://www.thegeekdiary.com/how-to-resize-an-ocfs2-filesystem-on-linux/ la ampliación se hace con el fs DESMONTADO de todos los nodos del cluster;
  como toda operación peligroso, saólo debe intentarse después de contar con respaldos completos, consistentes  y actuales de los archivos.

# Apéndice B: Instalación manual

Para crear `Vagrantfile.auto` primero usé `Vagrantfile.manual` y las instrucciones de [Instalación manual](instalacion-manual-lab1.md) para el Lab1.
