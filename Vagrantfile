# -*- mode: ruby -*-
# vi: set ft=ruby :

# The calicoctl download URL.
calicoctl_url = "https://github.com/projectcalico/calicoctl/releases/download/v3.2.0/calicoctl"

# etcd version
etcd_version = "3.3.9"

# Size of the cluster created by Vagrant
num_instances=2

# Change basename of the VM
instance_name_prefix="calico"

# The IP address of the first server
primary_ip = "172.17.8.101"

Vagrant.configure(2) do |config|
  # always use Vagrants insecure key
  config.ssh.insert_key = true
  config.ssh.username = "vagrant"

  # Use Bento Ubuntu 16.04 box (officially-recommended box by Vagrant)
  # https://www.vagrantup.com/boxes.html
  config.vm.box = "bento/ubuntu-16.04"

  # Workaround 16.04 issue with Virtualbox where Box waits 5 minutes to start
  # if network "cable" is not connected: https://github.com/chef/bento/issues/682
  config.vm.provider "virtualbox" do |vb|
    vb.customize ["modifyvm", :id, "--cableconnected1", "on"]
  end

  # Set up each box
  (1..num_instances).each do |i|
    vm_name = "%s-%02d" % [instance_name_prefix, i]
    config.vm.define vm_name do |host|
      host.vm.hostname = vm_name

      ip = "172.17.8.#{i+100}"
      host.vm.network :private_network, ip: ip

      # Fix stdin: is not a tty error (http://foo-o-rama.com/vagrant--stdin-is-not-a-tty--fix.html)
      config.vm.provision "fix-no-tty", type: "shell" do |s|
        s.privileged = false
        s.inline = "sudo sed -i '/tty/!s/mesg n/tty -s \\&\\& mesg n/' /root/.profile"
      end

      # The docker provisioner installs docker.
      host.vm.provision :docker, images: [
          "busybox:latest"
      ]

      # Calico uses etcd for calico and docker clustering. Install it on the first host only.
      if i == 1
        # Download etcd and start.
        host.vm.provision :shell, inline: <<-SHELL
          # sudo apt-get install -y unzip
          curl -L --silent https://github.com/coreos/etcd/releases/download/v#{etcd_version}/etcd-v#{etcd_version}-linux-amd64.tar.gz -o etcd-v#{etcd_version}-linux-amd64.tar.gz
          tar xzvf etcd-v#{etcd_version}-linux-amd64.tar.gz
          nohup etcd-v#{etcd_version}-linux-amd64/etcd --advertise-client-urls=http://#{primary_ip}:2379 --listen-client-urls=http://#{primary_ip}:2379 > etcd.log &
        SHELL
      end

      # download calicoctl.
      host.vm.provision :shell, inline: "curl -L --silent #{calicoctl_url} -o /usr/local/bin/calicoctl"
      host.vm.provision :shell, inline: "chmod +x /usr/local/bin/calicoctl"

      # Ensure the vagrant and root users get the ETCD_ENDPOINTS environment.
      host.vm.provision :shell, inline: %Q|echo 'export ETCD_ENDPOINTS="http://#{primary_ip}:2379"' >> /home/vagrant/.profile|
      host.vm.provision :shell, inline: %Q|sudo sh -c 'echo "Defaults env_keep +=\"ETCD_ENDPOINTS\"" >>/etc/sudoers'|
    end
  end
end
