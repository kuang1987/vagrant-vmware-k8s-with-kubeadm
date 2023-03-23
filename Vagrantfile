require "yaml"
settings = YAML.load_file "settings.yaml"

NUM_WORKER_NODES = settings["nodes"]["workers"]["count"]

Vagrant.configure("2") do |config|
  (1..NUM_WORKER_NODES).each do |i|

    config.vm.define "node0#{i}" do |node|
      node.vm.box = settings["software"]["box"]
      node.vm.hostname = "worker-node0#{i}"
      if settings["shared_folders"]
        settings["shared_folders"].each do |shared_folder|
          node.vm.synced_folder shared_folder["host_path"], shared_folder["vm_path"], owner: "vagrant", group: "vagrant"
        end
      end

      node.ssh.forward_agent = true

      node.vm.provider "vmware_desktop" do |v|
          # vb.cpus = settings["nodes"]["workers"]["cpu"]
          # vb.memory = settings["nodes"]["workers"]["memory"]
          # v.ssh_info_public = true
          v.gui = true
          v.vmx["memsize"] = settings["nodes"]["workers"]["memory"]
          v.vmx["numvcpus"] = settings["nodes"]["workers"]["cpu"]
          v.vmx["ethernet0.virtualdev"] = "vmxnet3"
          v.linked_clone = false
          v.base_mac = "00:50:56:31:2#{i}:90"
      end
      node.vm.provision "shell",
        env: {
          "DNS_SERVERS" => settings["network"]["dns_servers"].join(" "),
          "KUBERNETES_VERSION" => settings["software"]["kubernetes"],
          # place cluster info under shared dir
          "SHARED_DIR" => settings["config_share_folder"],
        },
        path: "scripts/common.sh"
      node.vm.provision "shell",
        env: {
          # place cluster info under shared dir
          "SHARED_DIR" => settings["config_share_folder"],
        },
        path: "scripts/node.sh"

      # Only install the dashboard after provisioning the last worker (and when enabled).
      if i == NUM_WORKER_NODES and settings["software"]["dashboard"] and settings["software"]["dashboard"] != ""
        node.vm.provision "shell",
          env: {
            # place cluster info under shared dir
            "SHARED_DIR" => settings["config_share_folder"],
          },
          path: "scripts/dashboard.sh"
      end
    end

  end


  # config.vm.define "node01" do |node01|
  #   node01.vm.box = "starboard/ubuntu-arm64-20.04.5"
  #   node01.vm.hostname = 'node01'
  #
  #   # node01.vm.network :private_network, ip: "192.168.56.101"
  #   node01.vm.network :forwarded_port, guest: 22, host: 10122, id: "ssh"
  #   node01.vm.network :forwarded_port, guest: 80, host: 10180, id: "http"
  #   node01.ssh.forward_agent = true
  #   node01.vm.synced_folder "/Users/I501425/vagrant", "/home/vagrant/sync", owner: "vagrant", group: "vagrant"
  #
  #
  #
  #   node01.vm.provider :vmware_desktop do |v|
  #     v.ssh_info_public = true
  #     v.gui = true
  #     v.vmx["memsize"] = "2048"
  #     v.vmx["numvcpus"] = "2"
  #     v.vmx["ethernet0.virtualdev"] = "vmxnet3"
  #     v.linked_clone = false
  #   end
  # end
  #
  # config.vm.define "node02" do |node02|
  #   node02.vm.box = "starboard/ubuntu-arm64-20.04.5"
  #   node02.vm.hostname = 'node02'
  #
  #   # node02.vm.network :private_network, ip: "192.168.56.102"
  #   node02.vm.network :forwarded_port, guest: 22, host: 10222, id: "ssh"
  #   node02.vm.network :forwarded_port, guest: 80, host: 10280, id: "http"
  #   node02.vm.synced_folder "/Users/I501425/vagrant", "/home/vagrant/sync", owner: "vagrant", group: "vagrant"
  #
  #
  #   node02.vm.provider :vmware_desktop do |v|
  #     v.ssh_info_public = true
  #     v.gui = true
  #     v.vmx["memsize"] = "2048"
  #     v.vmx["numvcpus"] = "2"
  #     v.vmx["ethernet0.virtualdev"] = "vmxnet3"
  #     v.linked_clone = false
  #   end
  # end
  #
  # config.vm.define "node03" do |node03|
  #   node03.vm.box = "starboard/ubuntu-arm64-20.04.5"
  #   node03.vm.hostname = 'node03'
  #
  #   # node03.vm.network :private_network, ip: "192.168.56.103"
  #   node03.vm.network :forwarded_port, guest: 22, host: 10322, id: "ssh"
  #   node03.vm.synced_folder "/Users/I501425/vagrant", "/home/vagrant/sync", owner: "vagrant", group: "vagrant"
  #
  #
  #   node03.vm.provider :vmware_desktop do |v|
  #     v.ssh_info_public = true
  #     v.gui = true
  #     v.vmx["memsize"] = "2048"
  #     v.vmx["numvcpus"] = "2"
  #     v.vmx["ethernet0.virtualdev"] = "vmxnet3"
  #     v.linked_clone = false
  #   end
  # end

  config.vm.define "control01" do |control01|
    control01.vm.box = settings["software"]["box"]
    control01.vm.hostname = 'control01'

    # control01.vm.network :private_network
    # control01.vm.network :forwarded_port, guest: 22, host: 11022, id: "ssh"
    # control01.vm.network :forwarded_port, guest: 80, host: 11080, id: "http"
    if settings["shared_folders"]
      settings["shared_folders"].each do |shared_folder|
        control01.vm.synced_folder shared_folder["host_path"], shared_folder["vm_path"], owner: "vagrant", group: "vagrant"
      end
    end
    # control01.vm.synced_folder "/Users/I501425/vagrant", "/home/vagrant/sync", owner: "vagrant", group: "vagrant"


    control01.vm.provider "vmware_desktop" do |v|
      v.gui = true
      v.vmx["memsize"] = settings["nodes"]["control"]["memory"]
      v.vmx["numvcpus"] = settings["nodes"]["control"]["cpu"]
      v.vmx["ethernet0.virtualdev"] = "vmxnet3"
      v.linked_clone = false
      v.base_mac = "00:50:56:31:20:90"
    end
    control01.vm.provision "shell",
      env: {
        "DNS_SERVERS" => settings["network"]["dns_servers"].join(" "),
        "ENVIRONMENT" => settings["environment"],
        "KUBERNETES_VERSION" => settings["software"]["kubernetes"],
        # place cluster info under shared dir
        "SHARED_DIR" => settings["config_share_folder"],
      },
      path: "scripts/common.sh"
    control01.vm.provision "shell",
      env: {
        "POD_CIDR" => settings["network"]["pod_cidr"],
        "SERVICE_CIDR" => settings["network"]["service_cidr"],
        # place cluster info under shared dir
        "SHARED_DIR" => settings["config_share_folder"],
      },
      path: "scripts/master.sh"
  end
end
