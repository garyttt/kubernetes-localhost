ENV["LC_ALL"] = "en_US.UTF-8"
CODE_NAME = "focal"
IMAGE_NAME = "ubuntu/focal64"
# https://portal.cloud.hashicorp.com/vagrant/discover/ubuntu/focal64
IMAGE_VERN = "20240821.0.1"
N = 2

Vagrant.configure("2") do |config|
    config.ssh.insert_key = false
    # config.vm.synced_folder ".", "/vagrant", create: true

    # Fix for 'Timed out while waiting for the machine to boot', also reset (disable/re-enable) Virtualbox network adapter(s) and then re-try 
    config.vm.boot_timeout = 600
     
    # Fix for 'failed to open/create internal network' - reset the network adapter by disable/re-enable, see:
    # https://stackoverflow.com/questions/33725779/failed-to-open-create-the-internal-network-vagrant-on-windows10

    # Fix for 'supHardenedWinVerifyProcess failed with -5607: ntdll.dll' on Win11 Insider version - use GUI normal start instead of 'headless', see:
    # https://forums.virtualbox.org/viewtopic.php?t=109295
    config.vm.provider "virtualbox" do |vb|
      vb.gui = true
    end

    config.vm.define "k8s-nfs-server" do |nfs|
        nfs.vm.box = IMAGE_NAME
        nfs.vm.box_version = IMAGE_VERN
        nfs.vm.box_check_update = "True"
        nfs.vm.network "private_network", ip: "192.168.100.150"
        nfs.vm.hostname = "k8s-nfs-server"
        nfs.vm.provider "virtualbox" do |nfs|
          nfs.name = 'k8s-nfs-server'
          nfs.memory = "768"
          nfs.cpus = 1
          # Fix for vagrant hangs at 'SSH auth method: Private key', see:
          # https://stackoverflow.com/questions/38463579/vagrant-hangs-at-ssh-auth-method-private-key
          nfs.customize ["modifyvm", :id, "--cableconnected1", "on"]
          nfs.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
          nfs.customize ["modifyvm", :id, "--uartmode1", "file", File::NULL]
        end 
        # workaround as per https://github.com/hashicorp/vagrant/issues/11544
        config.vm.provision :shell, inline: "apt-get update -y && apt-get upgrade -y && apt-get install -qy ansible"
        nfs.vm.provision "ansible_local" do |ansible|
            ansible.playbook = "ansible/playbooks/nfs/nfs-server.yaml"
            ansible.extra_vars = {
                node_ip: "192.168.100.150",
                ansible_python_interpreter: "/usr/bin/python3",
                code_name: CODE_NAME
            }
            vagrant_synced_folder_default_type = ""
        end
    end

    config.vm.define "k8s-master" do |master|
        master.vm.box = IMAGE_NAME
        master.vm.box_version = IMAGE_VERN
        master.vm.box_check_update = "True"
        master.vm.network "private_network", ip: "192.168.100.100"
        master.vm.hostname = "k8s-master"
        master.vm.provider "virtualbox" do |master|
          master.name = 'k8s-master'
          master.memory = "2048"
          master.cpus = 2
          # Fix for vagrant hangs at 'SSH auth method: Private key', see:
          # https://stackoverflow.com/questions/38463579/vagrant-hangs-at-ssh-auth-method-private-key
          master.customize ["modifyvm", :id, "--cableconnected1", "on"]
          master.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
          master.customize ["modifyvm", :id, "--uartmode1", "file", File::NULL]
        end
        config.vm.provision :shell, inline: "apt-get update -y && apt-get upgrade -y && apt-get install -qy ansible"
        master.vm.provision "ansible_local" do |ansible|
            ansible.playbook = "ansible/playbooks/kubernetes/kubernetes-master-install.yaml"
            ansible.extra_vars = {
                node_ip: "192.168.100.100",
                ansible_python_interpreter: "/usr/bin/python3",
                code_name: CODE_NAME
            }
        end
        # It is also possible to specify this playbook to run as part of
        # kubernetes-master-install.yaml, if you prefer it that way.
        #master.vm.provision "ansible_local" do |ansible|
        #    ansible.playbook = "ansible/playbooks/lb/lb-deployment.yaml"
        #    ansible.extra_vars = {
        #        ansible_python_interpreter: "/usr/bin/python3"
        #    }
        #end
        #master.vm.provision "ansible_local" do |ansible|
        #    ansible.playbook = "ansible/playbooks/dashboard/kubernetes-dashboard-install.yaml"
        #     ansible.extra_vars = {
        #        ansible_python_interpreter: "/usr/bin/python3"
        #    }
        #end
        master.vm.provision "ansible_local" do |ansible|
            ansible.playbook = "ansible/playbooks/nfs/nfs-kubernetes-deploy.yaml"
        end

    end

    (1..N).each do |node_id|
        config.vm.define "k8s-node-#{node_id}" do |node|
            node.vm.box = IMAGE_NAME
            node.vm.box_version = IMAGE_VERN
            node.vm.box_check_update = "True"
            node.vm.network "private_network", ip: "192.168.100.10#{node_id}"
            node.vm.hostname = "k8s-node-#{node_id}"
            node.vm.provider "virtualbox" do |node|
              node.name = "k8s-node-#{node_id}"
              node.memory = "2048"
              node.cpus = 2
              # Fix for vagrant hangs at 'SSH auth method: Private key', see:
              # https://stackoverflow.com/questions/38463579/vagrant-hangs-at-ssh-auth-method-private-key
              node.customize ["modifyvm", :id, "--cableconnected1", "on"]
              node.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
              node.customize ["modifyvm", :id, "--uartmode1", "file", File::NULL]
            end
            config.vm.provision :shell, inline: "apt update -y && apt upgrade -y && apt install -qy ansible"
            node.vm.provision "ansible_local" do |ansible|
                ansible.playbook = "ansible/playbooks/kubernetes/kubernetes-node-install.yaml"
                ansible.extra_vars = {
                    node_ip: "192.168.100.10#{node_id}",
                    ansible_python_interpreter: "/usr/bin/python3",
                    code_name: CODE_NAME
                }
            end
        end
    end
end
