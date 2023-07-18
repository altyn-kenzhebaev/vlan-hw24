# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
:inetRouter => {
    :box_name => "almalinux/9",
    :nets => {
              :net2 => {
                :adapter => 2, 
                :virtualbox__intnet => "router-net-central"
              },
              :net3 => {
                :adapter => 3, 
                :virtualbox__intnet => "router-net-central"
              },
             }
  },

  :centralRouter => {
    :box_name => "almalinux/9",
    :nets => {
              :net2 => {
                :adapter => 2, 
                :virtualbox__intnet => "router-net-central"
              },
              :net3 => {
                :adapter => 3, 
                :virtualbox__intnet => "router-net-central"
              },
              :net4 => {
                :ip => '192.168.255.9', 
                :netmask => "255.255.255.252", 
                :adapter => 4, 
                :virtualbox__intnet => "office1-central"
              },
            }
  },
  
  :office1Router => {
    :box_name => "almalinux/9",
    :nets => {
              :net2 => {
                :ip => '192.168.255.10', 
                :netmask => "255.255.255.252", 
                :adapter => 2, 
                :virtualbox__intnet => "office1-central"
              },
              :net3 => {
                :adapter => 3, 
                :virtualbox__intnet => "vlan1"
              },
              :net4 => {
                :adapter => 4, 
                :virtualbox__intnet => "vlan1"
              },
              :net5 => {
                :adapter => 5, 
                :virtualbox__intnet => "vlan2"
              },
              :net6 => {
                :adapter => 6, 
                :virtualbox__intnet => "vlan2"
              },
            }
  },

  :testClient1 => {
        :box_name => "almalinux/9",
        :nets => {
                  :net2 => {
                    :adapter => 2,
                    :virtualbox__intnet => "testLAN"
                  },
                }
  },

  :testServer1 => {
    :box_name => "almalinux/9",
    :nets => {
              :net2 => {
                :adapter => 2,
                :virtualbox__intnet => "testLAN"
              },
            }
  },

  :testClient2 => {
        :box_name => "ubuntu/jammy64",
        :nets => {
                  :net2 => {
                    :adapter => 2,
                    :virtualbox__intnet => "testLAN"
                  },
                }
  },

  :testServer2 => {
    :box_name => "ubuntu/jammy64",
    :nets => {
              :net2 => {
                :adapter => 2,
                :virtualbox__intnet => "testLAN"
              },
            }
  },

}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|

        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxname.to_s
        

        boxconfig[:nets].each do |netname, netconf|
          unless netconf[:ip] == nil
            box.vm.network "private_network", 
            ip: netconf[:ip], 
            adapter: netconf[:adapter], 
            netmask: netconf[:netmask], 
            virtualbox__intnet: netconf[:virtualbox__intnet]
          else 
            box.vm.network "private_network",
            auto_config: false,
            adapter: netconf[:adapter], 
            virtualbox__intnet: netconf[:virtualbox__intnet]
          end
        end

        if boxconfig[:vm_name] == "testServer2"
          box.vm.provision "ansible" do |ansible|
           ansible.playbook = "ansible/provision.yml"
           ansible.inventory_path = "ansible/hosts"
           ansible.compatibility_mode = "2.0"
          end
         end

      end

  end
  
end
