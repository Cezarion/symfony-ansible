# -*- mode: ruby -*-
# vi: set ft=ruby :

# TODO: Change the name
projectname = 'symfnony-bootstrap'

Vagrant.configure("2") do |config|
  config.vm.box = "debian/jessie64"
# TODO: Change the directory
  config.vm.network :private_network, ip: "10.0.0.7"
  config.vm.hostname = projectname+".vb"
  config.hostsupdater.aliases = ["www."+projectname+".vb", projectname+".local", projectname+".localnet"]

# TODO: Change the directory
  config.vm.synced_folder "./www", "/var/www/" + projectname + "/current", type: "nfs"

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :machine

    config.cache.synced_folder_opts = {
      type: :nfs,
      mount_options: ['rw', 'vers=3', 'tcp', 'nolock']
    }

    config.cache.enable :generic, {
      "cache"  => { cache_dir: "/var/www/" + projectname + "/current/app/cache" },
      "logs"   => { cache_dir: "/var/www/" + projectname + "/current/app/logs" },
      "vendor" => { cache_dir: "/var/www/" + projectname + "/current/vendor" },
    }
  end

  config.vm.provider "virtualbox" do |v|
    v.customize ["modifyvm", :id, "--cpuexecutioncap", "100"]
    v.customize ["modifyvm", :id, "--memory", 2048]
    v.customize ["modifyvm", :id, "--cpus", 2]
  end

  config.ssh.forward_agent = true

  # Ansible see https://docs.vagrantup.com/v2/provisioning/ansible.html
  config.vm.provision "ansible" do |ansible|
    ansible.sudo = true
    ansible.playbook = "provisioning/playbook.yml"
    ansible.extra_vars = {
      # general
      projectname: projectname,
      document_root: "/var/www/current/htdocs"
    }
    ansible.limit = 'vagrant'
    ansible.inventory_path = "provisioning/hosts/vagrant"
    ansible.verbose = "v" #Use vvvv to get more log
    #ansible.ask_vault_pass = true #ask for the vault password
  end
end
