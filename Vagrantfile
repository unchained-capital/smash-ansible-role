Vagrant.configure("2") do |config|
  config.vm.define :sshd do |sshd|
    sshd.vm.hostname = "sshclient"
	  sshd.vm.box = "ubuntu/trusty64"
    sshd.vm.network :private_network, ip: "192.168.10.88"
    sshd.vm.provision "ansible" do |ansible|
      ansible.sudo = true
      ansible.verbose = "vv"
      ansible.playbook = "test.yml"
    end
  end
end
