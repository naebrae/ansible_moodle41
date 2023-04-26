Vagrant.configure(2) do |config|

  config.vm.define :dc1 do |dc1_config|
    dc1_config.vm.box = "generic/alpine317"
    dc1_config.vm.network "private_network", ip: "172.16.1.251"

    dc1_config.vm.provider "virtualbox" do |dc1_vb|
      dc1_vb.name = "dc1"
    end
    dc1_config.vm.provision "shell", inline: <<-SHELL
      echo "172.16.1.251	dc1.ad.lab.home	dc1" >> /etc/hosts
    SHELL
    dc1_config.vm.provision "ansible" do |ansible|
      ansible.playbook = "samba_dc.yml"
    end
  end

  config.vm.define :moodle41, primary: true do |moodle41_config|
    moodle41_config.vm.box = "almalinux/9"
    moodle41_config.vm.network "private_network", ip: "172.16.1.141"
    #moodle41_config.vm.network "forwarded_port", guest:  80, host: 1080
    #moodle41_config.vm.network "forwarded_port", guest: 443, host: 1443
    moodle41_config.vm.network "public_network", ip: "192.168.1.141", bridge: "enp3s0"
    moodle41_config.vm.provider "virtualbox" do |moodle41_vb|
      moodle41_vb.name = "moodle41"
      moodle41_vb.memory = "4096"
    end
    moodle41_config.vm.provision "shell", inline: <<-SHELL
      echo "172.16.1.251	dc1.ad.lab.home	dc1" >> /etc/hosts
      sed -i "/UseDNS/cUseDNS no" /etc/ssh/sshd_config
      systemctl restart sshd
    SHELL
    moodle41_config.vm.provision "ansible" do |ansible|
      ansible.playbook = "playbook.yml"
      ansible.host_vars = {
        "moodle41"  => { "moodle_site" => "https://moodle41.lab.home",
                         "moodle_ldaphost" => "ldap://dc1.ad.lab.home", "moodle_ldapbase" => "DC=ad,DC=lab,DC=home",
                         "moodle_adminpass" => "Passw0rd!", "moodle_adminemail" => "root@localhost.localdomain",
                         "moodle_fullname" => "Moodle Four Point One Point Two", "moodle_shortname" => "Moodle 4.1.2",
                         "usegit" => "false" },
      }
    end
  end

  config.vm.synced_folder ".", "/home/vagrant/sync", type: "rsync", disabled: true
  config.vm.synced_folder ".", "/vagrant", disabled: true

end
