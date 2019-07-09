if  !Vagrant.has_plugin?("vagrant-cachier")
  raise 'Please install the vagrant plugin vagrant-cachier. "vagrant plugin install vagrant-cachier".'
end

# for cahcing the vagrant boxes, 

if !Vagrant.has_plugin?("vagrant-hostmanager")
  raise 'Please install the vagrant plugin vagrant-hostmanager.  "vagrant plugin install vagrant-hostmanager".'
end 
# for creating aliases on box,eg rails and railspg vms (expose the hosts of vm to the control machine)


# note- vagrant init makes a defualt template, which Ore uses heere

Vagrant.configure("2") do |config|  # initialize vagrant configuration, ruby syntax
  # Enable HostManager plugin
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true

  # Use ~/.vagrant.d/insecure_private_key -- allows ssh agent on local machine
  config.ssh.insert_key = false
  config.ssh.forward_agent = true

  # Files must be downloaded to /var/cache/jars to be cached
  # Files are cached in ~/.vagrant.d/cache/ if you want to remove them
  config.cache.scope = :box
  config.cache.enable :apt
  config.cache.enable :generic, { :cache_dir => "/var/cache/jars" }


# Ore added this: 
  vms = [
    {
      name: "rails",
      memory: 512,
      aliases: [ "rails.vagrant", "rails.vagrant.com" ],
      private_ip: "172.16.100.102"
    },
    {
      name: "railspg",
      memory: 512,
      aliases: [ 'railspg.vagrant', 'railspg.vagrant.com' ],
      private_ip: "172.16.100.101"
    }
  ]

  last_vm = vms.size - 1

  # TODO: please replace this with something that has less Ruby, and just declares the VMs
  # This is a loop through the Vm's array 
  vms.each_with_index do |vmspec, id|
    config.vm.define vmspec[:name] do |cfg|
      cfg.vm.box = "bento/ubuntu-18.10"  # tells what box to use
      
      cfg.vm.hostname = vmspec[:name] # vmpspec is what we use to iterate through array
      cfg.hostmanager.aliases = vmspec[:aliases]
      cfg.vm.network 'private_network', ip: vmspec[:private_ip]

      cfg.vm.provision :hostmanager
      
      cfg.vm.provider "virtualbox" do | v |
        v.gui = false
        v.customize ["modifyvm", :id, "--memory", vmspec[:memory]]
        v.customize ["modifyvm", :id, "--name", vmspec[:name]]
        v.cpus = 2
      end

      if id == last_vm  # means we finished provisioning all arrays in array, now we
      		# can trigger ansible
        cfg.vm.provision :ansible do |ansible|  # use ansible to provision
          ansible.inventory_path = "hosts"
          ansible.playbook = "deploy.yml"
          ansible.limit = "all"
          # Used to specify the dependency tag when provisioning a base box
          ansible.raw_arguments = Shellwords.shellsplit(ENV['ANSIBLE_ARGS']) if ENV['ANSIBLE_ARGS']
          ansible.verbose = 'vvvv' # ansible has a flag to set, for debugging (vvv)
          if File.exist? 'vagrant_overrides.yml' 
            ansible.extra_vars = 'vagrant_overrides.yml'
          end
        end
      end
    end
  end
end
