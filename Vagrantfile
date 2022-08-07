# this is required to be able to configure the hyper-v vm.
ENV['VAGRANT_EXPERIMENTAL'] = 'typed_triggers'

CONFIG_DNS_DOMAIN      = "test"

VM_WINDOWS_MEMORY_MB = 5*1024

VM_WINDOWS_CPUS = 4

VM_WINDOWS_IP_ADDRESS = "10.0.0.3"

VM_HYPERV_SWITCH_NAME = "docker-windows"

Vagrant.configure("2") do |config|
  config.vm.provider "libvirt" do |lv, config|
    lv.cpu_mode = "host-passthrough"
    lv.nested = false
    lv.keymap = "pt"
  end

  config.vm.provider "hyperv" do |hv, config|
    hv.linked_clone = true
    hv.enable_virtualization_extensions = false # nested virtualization.
    hv.vlan_id = ENV["HYPERV_VLAN_ID"]
    # see https://github.com/hashicorp/vagrant/issues/7915
    # see https://github.com/hashicorp/vagrant/blob/10faa599e7c10541f8b7acf2f8a23727d4d44b6e/plugins/providers/hyperv/action/configure.rb#L21-L35
    config.vm.network :private_network, bridge: ENV["HYPERV_SWITCH_NAME"] if ENV["HYPERV_SWITCH_NAME"]
    # further configure the VM (e.g. manage the network adapters).
    config.trigger.before :'VagrantPlugins::HyperV::Action::StartInstance', type: :action do |trigger|
      trigger.ruby do |env, machine|
        # see https://github.com/hashicorp/vagrant/blob/v2.2.19/lib/vagrant/machine.rb#L13
        # see https://github.com/hashicorp/vagrant/blob/v2.2.19/plugins/kernel_v2/config/vm.rb#L716
        bridges = machine.config.vm.networks.select{|type, options| type == :private_network && options.key?(:hyperv__bridge)}.map do |type, options|
          mac_address_spoofing = false
          mac_address_spoofing = options[:hyperv__mac_address_spoofing] if options.key?(:hyperv__mac_address_spoofing)
          [options[:hyperv__bridge], options[:ip], mac_address_spoofing]
        end
        system(
          'PowerShell',
          '-NoLogo',
          '-NoProfile',
          '-ExecutionPolicy',
          'Bypass',
          '-File',
          'configure-hyperv-vm.ps1',
          machine.id,
          bridges.to_json
        )
      end
    end
  end

  config.vm.define :windows do |config|
    config.vm.box = "windows-2019-amd64"
    config.vm.hostname = "windows"
    config.vm.provider "libvirt" do |lv, config|
      lv.memory = VM_WINDOWS_MEMORY_MB
      lv.cpus = VM_WINDOWS_CPUS
      # copy the files from host to guest.
      # NB this is required because docker build does not work over the SMB share.
      config.vm.synced_folder ".", "/vagrant", type: "rsync", rsync__exclude: [
        ".vagrant/",
        ".git/",
        "*.box"]
    end
    config.vm.network "private_network", ip: VM_WINDOWS_IP_ADDRESS, libvirt__forward_mode: "none", libvirt__dhcp_enabled: false, hyperv__bridge: VM_HYPERV_SWITCH_NAME
    config.vm.provision "shell", path: "configure-hyperv-guest.ps1", args: [VM_WINDOWS_IP_ADDRESS]
    config.vm.provision "shell", path: "ps.ps1", args: "provision-containers-feature.ps1"
    config.vm.provision "reload"
    config.vm.provision "shell", path: "ps.ps1", args: "provision-chocolatey.ps1"
    config.vm.provision "shell", path: "ps.ps1", args: "provision-base.ps1"
    config.vm.provision "shell", path: "ps.ps1", args: "provision-git.ps1"
    config.vm.provision "shell", path: "ps.ps1", args: "provision-containerd.ps1"
    config.vm.provision "shell", path: "ps.ps1", args: "provision-nerdctl.ps1"
    config.vm.provision "shell", path: "ps.ps1", args: "run-example-container.ps1"
    config.vm.provision "shell", path: "ps.ps1", args: "summary.ps1"
  end
end
