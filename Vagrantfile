# -*- mode: ruby -*-
# vi: set ft=ruby :

# On Windows, only supported if running under Linux Subsystem for Windows
if Vagrant::Util::Platform.windows?
    puts 'Linux Subsystem for Windows required on Windows hosts.  Please refer to Windows.md for further instruction.'
    abort 
end

VAGRANTFILE_API_VERSION = "2"

UUID = "OGVIFL"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

    config.ssh.insert_key = false

    (1..2).each do |id|
        re_name  = ( "vqfx" + id.to_s ).to_sym
        pfe_name = ( "vqfx" + id.to_s + "-pfe" ).to_sym

        ##############################
        ## Packet Forwarding Engine ##
        ##############################
        config.vm.define pfe_name do |vqfxpfe|
            vqfxpfe.ssh.insert_key = false
            vqfxpfe.vm.box = 'juniper/vqfx10k-pfe'

            # DO NOT REMOVE / NO VMtools installed
            vqfxpfe.vm.synced_folder '.', '/vagrant', disabled: true
            vqfxpfe.vm.network 'private_network', auto_config: false, nic_type: '82540EM', virtualbox__intnet: "#{UUID}_vqfx_internal_#{id}"

            # In case you have limited resources, you can limit the CPU used per vqfx-pfe VM, usually 50% is good
            # vqfxpfe.vm.provider "virtualbox" do |v|
            #    v.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
            # end
        end

        ##########################
        ## Routing Engine  #######
        ##########################
        config.vm.define re_name do |vqfx|
            vqfx.vm.hostname = "vqfx#{id}"
            vqfx.vm.box = 'juniper/vqfx10k-re'

            # VM can be really slow unless COM1 is connected to something.
            (File.exist?("/proc/version") and File.readlines("/proc/version").grep(/(Microsoft|WSL)/).size > 0) ? ( re_log = "NUL" ) : ( re_log = "/dev/null" )
            vqfx.vm.provider "virtualbox" do |v|
                v.customize ["modifyvm", :id, "--uartmode1", "file", re_log]
            end

            # DO NOT REMOVE / NO VMtools installed
            vqfx.vm.synced_folder '.', '/vagrant', disabled: true

            # Management port
            vqfx.vm.network 'private_network', auto_config: false, nic_type: '82540EM', virtualbox__intnet: "#{UUID}_vqfx_internal_#{id}"
            vqfx.vm.network 'private_network', auto_config: false, nic_type: '82540EM', virtualbox__intnet: "#{UUID}_reserved-bridge"

            # Dataplane ports
            (1..6).each do |seg_id|
               vqfx.vm.network 'private_network', auto_config: false, nic_type: '82540EM', virtualbox__intnet: "#{UUID}_seg#{seg_id}"
            end
        end
    end

    ##############################
    ## Box provisioning        ###
    ##############################
    config.vm.provision "ansible" do |ansible|
        ansible.compatibility_mode = "2.0"
        ansible.config_file = "ansible.cfg"
        ansible.groups = {
            "vqfx10k" => ["vqfx1", "vqfx2" ],
            "vqfx10kpfe"  => ["vqfx1-pfe", "vqfx2-pfe"],
            "all:children" => ["vqfx10k", "vqfx10kpfe"]
        }
        ansible.playbook = "pb.conf.all.commit.yaml"
    end
end
