# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'json'
schema = "#{Dir.pwd}/files/config.json"
configFile = JSON.parse(File.read(schema))

hosts = ''
ansibleGroups = []
configFile['instances'].each do |v|
  hosts = hosts + "#{v['ip_address'].to_s} #{v['hostname'].to_s}\n"
  v['groups'].each do |l|
    ansibleGroups.push l.to_s
  end
end
ansibleGroups = ansibleGroups.uniq
numberOfInstances = configFile['instances'].count
instanceNumber = 0

Vagrant.configure('2') do |config|
  config.vm.box = configFile['box'].to_s
  config.vm.box_version = configFile['box_version'].to_s

  configFile['instances'].each do |v|

    config.vm.define (v['hostname']).to_s do |k|
      instanceNumber = instanceNumber + 1
      k.vm.provider configFile['vm_provider'].to_s do |vb|
        vb.name = (v['hostname']).to_s
        vb.cpus = (v['cpus']).to_s
        vb.memory = (v['memory']).to_s
      end

      k.vm.hostname = (v['hostname']).to_s
      k.vm.network 'private_network', ip: (v['ip_address']).to_s

      k.vm.provision 'shell' do |s|
        ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
        s.inline = <<-SHELL
        nmcli con add con-name eth1 type ethernet ifname eth1 ip4 192.168.33.20/24
        ifup eth1
        mkdir /root/.ssh
        chmod 700 /root/.ssh
        echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
        echo #{ssh_pub_key} >> /root/.ssh/authorized_keys
        chmod 600 /root/.ssh/authorized_keys
        SHELL
      end
      if instanceNumber == numberOfInstances
        k.vm.provision 'ansible' do |ansible|
          ansible.playbook = 'provisioning/playbook.yml'
          ansible.limit = 'all'
          ansible.groups = {}
          ansibleGroups.each do |g|
            instances = []
            configFile['instances'].each do |i|
              if i['groups'].include? g.to_s
                instances.push i['hostname']
              end
            end
            ansible.groups[g] = instances
          end
          ansible.extra_vars = {
            'hosts' => hosts
          }
        end
      end
    end
  end
end
