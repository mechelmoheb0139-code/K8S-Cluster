Vagrant.configure("2") do |config|
  config.hostmanager.enabled = true 
  config.hostmanager.manage_host = true
  
### Control plan  ####
  config.vm.define "cont01" do |cont01|
    cont01.vm.box = "ubuntu/jammy64"
    cont01.vm.hostname = "cont01"
    cont01.vm.network "private_network", ip: "192.168.56.30"
	cont01.vm.provision "shell", path: "Controller.sh"
    cont01.vm.provider "virtualbox" do |vb|
     vb.memory = "2048"
   end
  end
  
### worker  #### 
  config.vm.define "work01" do |work01|
    work01.vm.box = "ubuntu/jammy64"
    work01.vm.hostname = "work01"
    work01.vm.network "private_network", ip: "192.168.56.31"
	work01.vm.provision "shell", path: "worker.sh"
    work01.vm.provider "virtualbox" do |vb|
     vb.memory = "2048"
   end
  end
  
### worker  ####
  config.vm.define "work02" do |work02|
    work02.vm.box = "ubuntu/jammy64"
    work02.vm.hostname = "work02"
    work02.vm.network "private_network", ip: "192.168.56.32"
	work02.vm.provision "shell", path: "worker.sh"
    work02.vm.provider "virtualbox" do |vb|
     vb.memory = "2048"
   end
  end
 
  
end
