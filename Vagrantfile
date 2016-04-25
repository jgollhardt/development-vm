# Only true 1.8+ gangsters need apply
Vagrant.require_version ">= 1.8.0"

require 'yaml'

# Load user specific vars from vagrant_config.yml
current_dir    = File.dirname(File.expand_path(__FILE__))
vconfig        = YAML.load_file("#{current_dir}/vagrant_config.yml")
vagrant_config = vconfig['config']

Vagrant.configure(2) do |config|
  config.vm.box = "bstoots/xubuntu-16.04-desktop-amd64"
  # Set hostname and vagrant name
  config.vm.hostname = vagrant_config['hostname']
  config.vm.define vagrant_config['vmname'].to_sym do |name_config| end
  # Move SSH port elsewhere if sshport is defined
  if vagrant_config.key?("sshport")
    config.vm.network :forwarded_port, guest: 22, host: vagrant_config['sshport'], id: "ssh"
  end
  # Stand up a bridged adapter so we can talk to NFS
  if vagrant_config.key?("publicmac")
    config.vm.network "public_network", :mac => vagrant_config['publicmac'], auto_config: false
  else
    config.vm.network "public_network", auto_config: false
  end
  # Stand up a private adapter so we can talk to other local boxes
  config.vm.network "private_network", type: "dhcp", auto_config: false
  # Virtualbox specific config
  config.vm.provider "virtualbox" do |vb|
    vb.gui = vagrant_config.key?("gui") ? vagrant_config['gui'] : false
    vb.memory = vagrant_config.key?("memory") ? vagrant_config['memory'] : 1024
    vb.name = vagrant_config['vmname']
    vb.customize ["modifyvm", :id, "--vram", "128"]
  end
  
  # Install apt packages
  config.vm.provision "shell" do |s|
    s.path = "vagrant-shell-provisioner/packages/apt-get/update.sh"
  end
  config.vm.provision "shell" do |s|
    s.path = "vagrant-shell-provisioner/packages/apt-get/upgrade.sh"
  end
  config.vm.provision "shell" do |s|
    s.path = "vagrant-shell-provisioner/packages/apt-get/install.sh"
    s.args = ["git", "python", "python-dev", "python-pip"]
  end
  # Install pip packages
  config.vm.provision "shell" do |s|
    s.path = "vagrant-shell-provisioner/packages/pip/install.sh"
    s.args = ["paramiko", "pyyaml", "jinja2", "markupsafe", "ansible"]
  end
  # Apply Ansible playbook from the guest
  config.vm.provision "shell" do |s|
    s.path = "vagrant-shell-provisioner/config-management/ansible/ansible-playbook.sh"
    s.args = [
      # Ansible working directory on the guest
      "/vagrant/ansible",
      # Playbooks
      "master.yml",
      # Extra vars file
      "extra_vars.yml",
      # Options
      "-v"
    ]
  end

  # Just kidding, ansible_local isn't working on Windows hosts as of 01/27/2016 ... maybe some day
  # Get our (built-in) Ansible provisioning on!
  # config.vm.provision "ansible_local" do |ansible|
  #   # This is done via shell provisioning because we can't choose Ansible version in Vagrant 1.8.0
  #   ansible.install = false
  #   ansible.version = "1.9.4"
  #   ansible.playbook = "/vagrant/master.yml"
  #   ansible.provisioning_path = "/vagrant"
  #   ansible.provisioning_path = "/tmp/vagrant-ansible"
  # end

  # # Kind of hooptie but guest_ansible isn't picking up our ansible.cfg
  # config.vm.provision :shell, :inline => "cp /vagrant/ansible.cfg /home/vagrant/.ansible.cfg"
  # # Provision the Ubuntu guest on the guest itself via guest_ansible plugin
  # config.vm.provision :guest_ansible do |ansible|
  #   ansible.playbook = "master.yml"
  #   # I couldn't get vagrant-guest_ansible to read the file from the filesystem
  #   # but it also accepts a Hash so lets just do that and be done with it.
  #   ansible.extra_vars = ansible_config
  # end
end
