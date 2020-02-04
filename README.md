# README - global-filesystem-lab

Laboratorio para experimentar con cluster filesystems.

Se realiza una prueba de concepto con OCFS2 Oracle Cluster Filesystem v2, 
el dispositivo de bloques que será el disco compartido se proporciona mediante 
xNBD (Network Block Device).

Una VM denominada `san` expone un dispositivo NBD imitando el servicio de una
SAN que proporciona una LUN. Dos VMs, denominadas `c1` y `c2` son los clientes que necesitan compartir
un filesystem.  En los clientes se instala el cliente de NBD y se configura OCFS2 en un cluster de dos nodos.

Las direcciones de IP, puertos del NBD, nombres de VM, y nombres de dispositivo 
de bloques están hardcodeadas en el `Vagrantfile`.


Este laboratorio muestra cómo eliminar un cluster de servidores GlusterFS y sustituirlo mediante un dispositivo de bloques
compartido a través de un filesystem de cluster.

Si el dispositivo de bloques es proporcionado por una SAN externa, se aprovechan las capacidades de
replicación y distribución de la SAN para dar confiabilidad al servicio de filesystem compartido.

Los pasos siguientes son:

* crear un disco independiente con Virtualbox y conectarlo a `c1`y `c2`, eliminando así la VM `san` y todo el software de NBD.
* utilizar una SAN externa, crear una LUN y presentarla a Virtualbox o VMware, para usarla como disco RAW compartido e independiente.


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
cp Vagrantfile.auto Vagrantfile
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

## `san`

La VM tiene el servidor de xNBD; verifique que el proceso está corriendo.

## `c1`y `c2`

En ambas VMs debe estar montado un filesystem tipo `ocfs2` en i`/mnt/ocfs2/`.
Cuando en `c1` se escribe en ese directorio, se puede verificar que las modificaciones
se pueden ver desde `c2`.

Además debe estar corriendo el cliente de NBD, una precondici{on para que funcione el cluster
OCFS" y que  gracias a eso se pueda haber montado el directorio mencionado.

Cuando se instala por primera vez, desde `c1`se procede al formateo del dispositivo  `/dev/nbd0`, lo 
cual se debe hacer sólo una vez, pues es una operaci{on a nivel del dispositivo de bloque.


# Apéndice: Instalación manual

Para crear `Vagrantfile.auto` primero usé `Vagrantfile.manual` y las instrucciones de [Instalación manual](instalacion-manual.md) 
