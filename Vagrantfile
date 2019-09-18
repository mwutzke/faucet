# -*- mode: ruby -*-
# vi: set ft=ruby :

# Based on https://github.com/illotum/vagrant-mininet.git

prov_env = {
  "DEBIAN_FRONTEND" => "noninteractive",
}

$init = <<SCRIPT
  apt-get update
  apt-get -y install \
    build-essential git tmux vim \
    autoconf automake libtool \
    libssl-dev
SCRIPT

$python = <<SCRIPT
  apt-get -y install \
    python3-all python3-dev \
    python3-pip python3-venv
SCRIPT

# This will install 'openvswitch' as a dependency
$mininet = <<SCRIPT
  apt-get -y install \
    mininet
SCRIPT

# WAND Open vSwitch
$wand_ovs = <<SCRIPT
  apt-get install apt-transport-https
  echo "deb https://packages.wand.net.nz $(lsb_release -sc) main" >/etc/apt/sources.list.d/wand.list
  curl -s https://packages.wand.net.nz/keyring.gpg -o /etc/apt/trusted.gpg.d/wand.gpg
  apt-get -y update
  apt-get -y install openvswitch-switch
SCRIPT

# This will install 'Ryu' as a requirement
$pip = <<SCRIPT
  pip3 install --upgrade pip
  pip3 install -r /vagrant/requirements.txt
SCRIPT

$faucet_dev = <<SCRIPT
  ln -sfT /vagrant ~/faucet

  python3 -m venv .venv
  . .venv/bin/activate
  pip install --upgrade pip

  cd faucet
  pip install -r requirements.txt
  pip install -e .

  mkdir -p var/log/faucet
SCRIPT

$cleanup = <<SCRIPT
  apt-get -y clean
  apt-get -y autoremove
  rm -rf /tmp/*
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-16.04"
  config.vm.box_check_update = false

  config.vm.provider :virtualbox do |v|
    v.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
    v.customize ["modifyvm", :id, "--memory", 6 * 1024]
    v.customize ["modifyvm", :id, "--cableconnected1", "on"]
  end

  config.vm.provider :libvirt do |v|
    v.cpus = 2
    v.memory = 4 * 1024
    config.vm.synced_folder ".", "/vagrant",
      type: "nfs", nfs_export: false
  end

  ## Guest config
  config.vm.hostname = "faucet"
  config.vm.network :forwarded_port, guest:6633, host:6633 # OpenFlow

  ## SSH config
  config.ssh.forward_x11 = false

  ## Testing
  config.vm.define :tests do |tests|
    config.vm.hostname = "faucet-tests"
    tests.vm.provision :shell, :env => prov_env, :inline => $init
    tests.vm.provision :shell, :env => prov_env, :inline => $python
    tests.vm.provision :shell, :env => prov_env, :inline => $mininet
    tests.vm.provision :shell, :env => prov_env, :inline => $wand_ovs
    tests.vm.provision :shell, :env => prov_env, :inline => $pip
    tests.vm.provision :shell, :env => prov_env, :inline => $cleanup
    tests.vm.provision :shell, :env => prov_env, :inline => $faucet_dev, privileged: false
  end
end
