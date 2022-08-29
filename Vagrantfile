Vagrant.configure(2) do |config|
 config.vm.box = "centos7"
 config.vm.define "ipa.yarkozloff.local" do |vmconfig|
 vmconfig.vm.hostname = "ipa.yarkozloff.local"
 vmconfig.vm.network "private_network", ip: "192.168.10.10"
 vmconfig.vm.provider "virtualbox" do |vbx|
        vbx.memory = "3000"
        vbx.cpus = "2"
        vbx.customize ["modifyvm", :id, '--audio', 'none']
        end
 config.vbguest.auto_update = false
 config.vm.provision "ansible" do |ansible|
        ansible.playbook = "play_server.yaml"
      end
 end
 config.vm.define "client.yarkozloff.local" do |vmconfig|
 vmconfig.vm.hostname = "client.yarkozloff.local"
 vmconfig.vm.network "private_network", ip: "192.168.10.20"
 config.vbguest.auto_update = false
 config.vm.provision "ansible" do |ansible|
        ansible.playbook = "play_client.yaml"
      end
 end
end
