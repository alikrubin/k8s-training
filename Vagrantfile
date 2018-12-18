Vagrant.configure("2") do |config|
  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.box = "ubuntu/bionic64"
  config.vm.provider "lxc" do |v, override|
    override.vm.box = "emptybox/ubuntu-bionic-amd64-lxc"
  end
  config.vm.provider "azure" do |az, override|
    override.vm.box = "azure-dummy"
    override.ssh.private_key_path = '~/.ssh/id_rsa'
    # Specify VM parameters
    az.vm_name = 'tutorialengine'
    az.vm_size = 'Standard_B1s'
    az.vm_image_urn = 'Canonical:UbuntuServer:18.04-LTS:latest'
    az.resource_group_name = 'vagrant'
  end
  config.vm.provider "aws" do |aws, override|
    override.vm.box = "dummy"
    override.ssh.private_key_path = '~/.ssh/Percona-Training.key'
    aws.keypair_name = "Percona-Training"
    aws.instance_type = "t2.xlarge"
    aws.region = "eu-central-1"
    aws.availability_zone = "eu-central-1a"
    # for eu-central-1
    aws.ami = "ami-0dfd7cad24d571c54"
    override.ssh.username = "ubuntu"
    aws.block_device_mapping = [{ 'DeviceName' => '/dev/sda1', 'Ebs.VolumeSize' => 50, 'Ebs.VolumeType' => 'gp2' }]
  end
  config.vm.provision "shell" do |s|
    s.inline = "(grep -q vagrant /etc/passwd || useradd -m vagrant );mkdir -p /vagrant /home/vagrant/tutorial; chown vagrant:adm /vagrant /home/vagrant/tutorial; chmod g+w -R /vagrant /home/vagrant/tutorial"
    s.privileged = true
  end
  config.vm.provision "file", source: "files", destination: "/home/vagrant/tutorial"
  config.vm.provision "file", source: "playbook.yml", destination: "/vagrant/playbook.yml"
  config.vm.provision "ansible_local" do |ansible|
    ansible.compatibility_mode = "auto"
    ansible.playbook = "playbook.yml"
    ansible.compatibility_mode = "2.0"
  end
  config.vm.provider "virtualbox" do |v|
    v.memory = 4096
    v.cpus = 2
  end

  config.vm.define "default", primary: true do |default|
    default.vm.network "forwarded_port", guest: 22, host: 3622, host_ip: "0.0.0.0"
  end

end
