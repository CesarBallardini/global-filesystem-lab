# -*- mode: ruby -*-
# vi: set ft=ruby :

# Para aprovechar este Vagrantfile necesita Vagrant y Virtualbox instalados:
#
#   * Virtualbox
#
#   * Vagrant
#
#   * Plugins de Vagrant:
#       + vagrant-proxyconf y su configuracion si requiere de un Proxy para salir a Internet
#       + vagrant-cachier
#       + vagrant-disksize
#       + vagrant-hostmanager
#       + vagrant-share
#       + vagrant-vbguest

VAGRANTFILE_API_VERSION = "2"

primera_vez = false
#en Ubuntu se puede usar en false siempre:
#primera_vez = false


gfs2_nodes = [
    { :name  =>"san", :public_ip => "192.168.50.3", :internal_ip => "192.168.40.13", :memory => "1024", :vcpus => "1" }, # exporta las unidades de bloques
    { :name => "c1",  :public_ip => "192.168.50.4", :internal_ip => "192.168.40.14", :memory => "1024", :vcpus => "1" }, # cliente 1 con GFS2
    { :name => "c2",  :public_ip => "192.168.50.5", :internal_ip => "192.168.40.15", :memory => "1024", :vcpus => "1" }, # cliente 2 con GFS2
]


Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  if Vagrant.has_plugin?("vagrant-hostmanager")
    config.hostmanager.enabled = true
    config.hostmanager.manage_host = true
    config.hostmanager.manage_guest = true
    config.hostmanager.ignore_private_ip = false
    config.hostmanager.include_offline = true

    # uso cachier con NFS solamente si el hostmanager gestiona los nombres en /etc/hosts del host
    if not primera_vez && Vagrant.has_plugin?("vagrant-cachier")

      config.cache.auto_detect = false
      # W: Download is performed unsandboxed as root as file '/var/cache/apt/archives/partial/xyz' couldn't be accessed by user '_apt'. - pkgAcquire::Run (13: Permission denied)

      config.cache.synced_folder_opts = {
        owner: "_apt"
      }
      # Configure cached packages to be shared between instances of the same base box.
      # More info on http://fgrehm.viewdocs.io/vagrant-cachier/usage
      config.cache.scope = :box
    end
  end

  gfs2_nodes.each do | nodo |

    config.vm.define nodo[:name] do |n|
      n.vm.hostname = nodo[:name]


    n.vm.box = "ubuntu/bionic64"
    #n.vm.box = "debian/buster64" # Antes de usar Buster, revisar Debian-Buster-fix.md
    #n.vm.box = "shumbert/stretch64"
    n.disksize.size = '10GB'

    n.vm.boot_timeout = 3600
    n.vm.box_check_update = true
    n.ssh.forward_agent = true
    n.ssh.forward_x11 = true

    if Vagrant.has_plugin?("vagrant-hostmanager")
      n.hostmanager.aliases = %W(#{nodo[:name]+".dominio.local.tld'"} #{nodo[:name]} )
    end

    if Vagrant.has_plugin?("vagrant-vbguest") then
        n.vbguest.auto_update = ! primera_vez
        n.vbguest.no_install = ! primera_vez
    end

    n.vm.synced_folder ".", "/vagrant", type: "virtualbox", disabled: primera_vez
      
    n.vm.network "private_network", ip: nodo[:public_ip]   #, virtualbox__intnet: true # interfaz externa virtual
    n.vm.network "private_network", ip: nodo[:internal_ip] #, virtualbox__intnet: true # interfaz interna 
    #n.vm.network "forwarded_port", guest: 80, host: 8080
    #n.vm.network "forwarded_port", guest: 443, host: 8443

    n.vm.provider "virtualbox" do |v|
      v.gui = false
      v.memory = nodo[:memory]
      v.cpus =   nodo[:vcpus]
    end

    ##
    # Aprovisionamiento
    #
    n.vm.provision "fix-no-tty", type: "shell" do |s|
      s.privileged = false
      s.inline = "sudo sed -i '/tty/!s/mesg n/tty -s \\&\\& mesg n/' /root/.profile"
    end

    n.vm.provision "actualiza", type: "shell" do |s|  # http://foo-o-rama.com/vagrant--stdin-is-not-a-tty--fix.html
      s.privileged = false
      s.inline = <<-SHELL
        export DEBIAN_FRONTEND=noninteractive
        export APT_LISTCHANGES_FRONTEND=none
        export APT_OPTIONS=' -y --allow-downgrades --allow-remove-essential --allow-change-held-packages -o Dpkg::Options::=--force-confdef -o Dpkg::Options::=--force-confold '

        sudo -E apt-get --purge remove apt-listchanges -y > /dev/null 2>&1
        sudo -E apt-get update -y -qq > /dev/null 2>&1
        sudo dpkg-reconfigure --frontend=noninteractive libc6 > /dev/null 2>&1
        sudo -E apt-get -q --option "Dpkg::Options::=--force-confold" --assume-yes install libssl1.1  > /dev/null 2>&1 # https://bugs.launchpad.net/ubuntu/+source/openssl/+bug/1832919

        [ Debian = $(lsb_release -is) ] && sudo -E apt-get install linux-image-amd64 ${APT_OPTIONS}

        LINUX_IMAGE_PACKAGE=$( apt-cache search "linux-image-4.*generic"  | cut --delimiter=" " --fields=1 | sort -nr | head -n1 )
        LINUX_HEADERS_PACKAGE=$( echo ${LINUX_IMAGE_PACKAGE} | sed "s/image/headers/" )
        [ Ubuntu = $(lsb_release -is) ] && sudo apt-get install ${LINUX_IMAGE_PACKAGE} ${LINUX_HEADERS_PACKAGE} ${APT_OPTIONS}

        sudo -E apt-get      upgrade ${APT_OPTIONS} > /dev/null 2>&1
        sudo -E apt-get dist-upgrade ${APT_OPTIONS} > /dev/null 2>&1
        sudo -E apt-get autoremove -y > /dev/null 2>&1
        sudo -E apt-get autoclean -y > /dev/null 2>&1
        sudo -E apt-get clean > /dev/null 2>&1
      SHELL
    end

    n.vm.provision "ssh_pub_key", type: :shell do |s|
      begin
        ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
        s.inline = <<-SHELL
          mkdir -p /root/.ssh/
          touch /root/.ssh/authorized_keys
          echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
          echo #{ssh_pub_key} >> /root/.ssh/authorized_keys
        SHELL
      rescue
        puts "No hay claves publicas en el HOME de su pc"
        s.inline = "echo OK sin claves publicas"
      end
    end

    if  "san" == nodo[:name]
      n.vm.provision "instala_san", type: "shell" do |s|
        s.privileged = false
        s.inline = <<-SHELL
          export DEBIAN_FRONTEND=noninteractive
          export APT_LISTCHANGES_FRONTEND=none
          export APT_OPTIONS=' -y --allow-downgrades --allow-remove-essential --allow-change-held-packages -o Dpkg::Options::=--force-confdef -o Dpkg::Options::=--force-confold '

          sudo apt-get install debconf-utils  ${APT_OPTIONS}
          sudo apt-get install linux-modules-extra-$(uname -r)  ${APT_OPTIONS}
          sudo apt-get install xnbd-client xnbd-server  ${APT_OPTIONS}

          sudo dd if=/dev/zero of=/srv/disk.img bs=4096 count=1 seek=1000000

        SHELL
      end
    end

    if  "san" == nodo[:name]
      n.vm.provision "enciende_san", type: "shell", run: 'always' do |s|
        s.privileged = false
        s.inline = <<-SHELL
          export DEBIAN_FRONTEND=noninteractive
          export APT_LISTCHANGES_FRONTEND=none
          export APT_OPTIONS=' -y --allow-downgrades --allow-remove-essential --allow-change-held-packages -o Dpkg::Options::=--force-confdef -o Dpkg::Options::=--force-confold '

          sudo apt-get install linux-modules-extra-$(uname -r)  ${APT_OPTIONS}
          #echo deadline | sudo tee /sys/block/nbd0/queue/scheduler # FIXME /sys/block/nbd0/queue/scheduler: Invalid argument
          sudo modprobe nbd
          nohup sudo xnbd-server --daemonize --syslog --target --lport 8992 /srv/disk.img

        SHELL
      end
    end

    if  "san" != nodo[:name]
      n.vm.provision "instala_cliente", type: "shell" do |s|
        s.privileged = false
        s.inline = <<-SHELL
          export DEBIAN_FRONTEND=noninteractive
          export APT_LISTCHANGES_FRONTEND=none
          export APT_OPTIONS=' -y --allow-downgrades --allow-remove-essential --allow-change-held-packages -o Dpkg::Options::=--force-confdef -o Dpkg::Options::=--force-confold '

          sudo apt-get install debconf-utils  ${APT_OPTIONS}
          sudo apt-get install linux-modules-extra-$(uname -r) ${APT_OPTIONS}
          sudo apt-get install xnbd-client xnbd-server ${APT_OPTIONS}
          sudo apt-get install ocfs2-tools ${APT_OPTIONS}

          sudo service o2cb stop


cat <<CONFIG | sudo tee /etc/default/o2cb
O2CB_ENABLED=true
O2CB_BOOTCLUSTER=cluster1
O2CB_HEARTBEAT_THRESHOLD=61
O2CB_IDLE_TIMEOUT_MS=30000
O2CB_KEEPALIVE_DELAY_MS=2000
O2CB_RECONNECT_DELAY_MS=2000
CONFIG

sudo dpkg-reconfigure -f noninteractive  ocfs2-tools

          
cat <<EOF | sudo tee /etc/ocfs2/cluster.conf
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
EOF
          
          
        SHELL
      end
    end

    if  "san" != nodo[:name]
      n.vm.provision "enciende_cliente", type: "shell", run: 'always' do |s|
        s.privileged = false
        s.inline = <<-SHELL
          export DEBIAN_FRONTEND=noninteractive
          export APT_LISTCHANGES_FRONTEND=none
          export APT_OPTIONS=' -y --allow-downgrades --allow-remove-essential --allow-change-held-packages -o Dpkg::Options::=--force-confdef -o Dpkg::Options::=--force-confold '

          sudo apt-get install linux-modules-extra-$(uname -r)  ${APT_OPTIONS}
          #echo deadline | sudo tee /sys/block/nbd0/queue/scheduler
          sudo modprobe nbd
          sudo xnbd-client bs=4096 192.168.40.13 8992 /dev/nbd0

          sudo service o2cb load
          sudo service o2cb online
          sudo service o2cb start
          
        SHELL
      end
    end

    if  "c1" == nodo[:name]
      n.vm.provision "formatea_unidad", type: "shell" do |s|
        s.privileged = false
        s.inline = <<-SHELL

          sudo mkfs.ocfs2 /dev/nbd0
          
        SHELL
      end
    end


    if  "san" != nodo[:name]
      n.vm.provision "monta_ocfs2", type: "shell", run: 'always' do |s|
        s.privileged = false
        s.inline = <<-SHELL

          [ -d /mnt/ocfs2 ] || sudo mkdir /mnt/ocfs2
          sudo mount -t ocfs2 /dev/nbd0 /mnt/ocfs2
          
        SHELL
      end
    end


  if  "san" != nodo[:name]
    n.trigger.before :halt do |trigger|
      trigger.info = "desmonto ocfs2"
      trigger.run_remote = { inline: "sudo umount /mnt/ocfs2 ; sudo xnbd-client --disconnect /dev/nbd0" }
    end
  end


  if  "san" == nodo[:name]
    n.trigger.before :halt do |trigger|
      trigger.info = "elimino server xnbd"
      trigger.run_remote =  { inline:  "sudo pkill -f xnbd-server" }
    end
  end

    proxy_host_port = ENV['all_proxy'] || ENV['http_proxy']  || ""
    proxy_host_port = if proxy_host_port.empty? then "" else proxy_host_port.scan(/\/\/([0-9\.]*):/)[0][0]+':'+proxy_host_port.scan(/:([0-9]*)$/)[0][0] end

  end
end
end
