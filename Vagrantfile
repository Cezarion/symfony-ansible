# -*- mode: ruby -*-
# vi: set ft=ruby :

# TODO: Change the name
projectname = 'symfnony-bootstrap'
local_path  = './www'
remote_path = '/var/www'


# Check to determine whether we're on a windows or linux/os-x host,
# later on we use this to launch ansible in the supported way
# source: https://stackoverflow.com/questions/2108727/which-in-ruby-checking-if-program-exists-in-path-from-ruby
def which(cmd)
    exts = ENV['PATHEXT'] ? ENV['PATHEXT'].split(';') : ['']
    ENV['PATH'].split(File::PATH_SEPARATOR).each do |path|
        exts.each { |ext|
            exe = File.join(path, "#{cmd}#{ext}")
            return exe if File.executable? exe
        }
    end
    return nil
end

Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/trusty64"
  config.ssh.insert_key = false
  config.ssh.forward_agent = true
  config.ssh.username = 'vagrant'
  config.ssh.password = 'vagrant'
  config.ssh.insert_key = 'true'

  config.vm.provider :virtualbox do |v|
    v.name = projectname+".vb"
    v.customize [
        "modifyvm", :id,
        "--name", projectname,
        "--memory", 512,
        "--natdnshostresolver1", "on",
        "--cpus", 2,
        "--ioapic", "on"
    ]
  end

  config.vm.hostname = projectname+".vb"
  config.vm.network :private_network, ip: "10.0.0.7"


  config.vm.synced_folder local_path, remote_path, id: "vagrant-root",
                                                    owner: "vagrant",
                                                    group: "www-data",
                                                    mount_options: ["dmode=775,fmode=664"]

  # Ansible see https://docs.vagrantup.com/v2/provisioning/ansible.html
  # If ansible is in your path it will provision from your HOST machine
  # If ansible is not found in the path it will be instaled in the VM and provisioned from there
  if which('ansible-playbook')
    config.vm.provision "ansible" do |ansible|
      ansible.sudo = true
      ansible.playbook = "provisioning/playbook.yml"
      ansible.extra_vars = {
        ansible_ssh_user: 'vagrant',
        projectname: projectname,
        mysql_user: 'vagrant',
        mysql_pass: 'vagrant',
        document_root: "/var/www/current/htdocs"
      }
      ansible.limit = 'vagrant'
      ansible.inventory_path = "provisioning/hosts/vagrant"
      ansible.verbose = "v" #Use vvvv to get more log
      ansible.sudo = true
      #ansible.ask_vault_pass = true #ask for the vault password
    end
  else
    config.vm.provision :shell, path: "provisioning/windows.sh", args: [ projectname+".vb" ]
  end

  if Vagrant.has_plugin?("vagrant-hostsupdater")
    config.hostsupdater.aliases = ["www."+projectname+".vb", projectname+".local", projectname+".localnet"]
  else
    fail_with_message "vagrant-hostsupdater missing, please install the plugin with this command:\nvagrant plugin install vagrant-hostsupdater"
  end

  # if Vagrant.has_plugin?("vagrant-cachier")
  #   config.cache.scope = :box
  #   config.cache.synced_folder_opts = {
  #     type: :nfs,
  #     mount_options: ['rw', 'vers=3', 'tcp', 'nolock']
  #   }

  #   config.cache.enable :generic, {
  #     "cache"  => { cache_dir: "/var/www/current/app/cache" },
  #     "logs"   => { cache_dir: "/var/www/current/app/logs" },
  #     "vendor" => { cache_dir: "/var/www/current/vendor" }
  #   }
  # end


end