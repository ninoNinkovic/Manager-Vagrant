# -*- mode: ruby -*-
# vi: set ft=ruby :
#
# Create Vagrant VM on vSphere
# using: vagrant-vsphere
#
# vagrant plugin install vagrant-vsphere
# https://github.com/nsidc/vagrant-vsphere

vSphereUserName  = 'signiantott\mboudreau'
vSpherePassword  = :ask

shortMachineName = 'marctestinternal'
machineName      = shortMachineName + '.ott.signiant.com'
vmName           = 'Marc Test VM - internal'
signiantOrgName  = 'Marc' # Signiant CA Org
userFolder       = 'Marc'

isInternalOnly   = true

managerVersion   = '10.5'

cpus       = 4
memoryInGb = 8

Vagrant.configure(2) do |config|

  # Dummy box needed for local Vagrant
  config.vm.box = "sharpie/dummy"

  # This is to prevent the provider from trying to connect with nfs (a bug in the provider)
  # This forces vagrant to sync with rsync
  config.nfs.functional = false

  # The VM doesn't seem to like this - it's done by brute force in the provisioning script
  config.vm.hostname = machineName

  config.vm.provider :vsphere do |vsphere|
    # The vSphere host we're going to connect to
    vsphere.host = 'ottvcs01.ott.signiant.com'

    # vSphere login
    vsphere.user = vSphereUserName

    # vSphere password
    vsphere.password = vSpherePassword

    vsphere.cpu_count = cpus
    vsphere.memory_mb = memoryInGb * 1024

    # Linked Clones not supported currently see:
    # http://www.vmware.com/files/pdf/techpaper/vsphere-storage-drs-interoperability.pdf
    # Essentially, for vSphere, the base image's disks have to be read only (delta-disks)
    #vsphere.linked_clone = 'true'

    # The ESX host for the new VM
    vsphere.compute_resource_name = 'VMware Cluster'

    # The resource pool for the new VM
    #vsphere.resource_pool_name = ''

    # Where to put the new VM
    vsphere.vm_base_path = '/Virtual Machines/Development/' + userFolder

    # The template we're going to clone
    #vsphere.template_name = '/Templates/Linux/Centos 6.5 64bit'
    vsphere.template_name = '/Virtual Machines/Development/Marc/Vagrant Template - Centos 6.5 64bit'

    # The name of the new machine
    vsphere.name = vmName

    if isInternalOnly
      vsphere.vlan = 'Internal Only'
    end

    # SSL isn't configured correctly, set this to 'true'
    vsphere.insecure = true
  end

  # Don't use a file provisioner for the sigsetup.rec file - change it inline
  config.vm.provision "shell", inline: <<-SHELL
    # Setup the hosts file
    echo Preparing the hosts file for Signiant
    cat > /etc/hosts << "EOF"
127.0.0.1 localhost.localdomain localhost
EOF
    echo `hostname -I` #{machineName} #{shortMachineName} >> /etc/hosts

    # Create the /tmp/sigsetup.rec file
    echo Preparing the /tmp/sigsetup.rec file
    cat > /tmp/sigsetup.rec << "EOF"
[RECPARMS]
SetupType=Standard
InstallDir=/usr/signiant/dds
OrgName=#{signiantOrgName}
IsOrgKeyless=FALSE
RBIEnabled=TRUE
DefaultUserUnix=transusr
DefaultUserWin=transusr
WinDomain=
DefaultDirUnix=/usr/signiant/dds/transfers
DefaultDirWin=C:\\Program Files\\Signiant\\Mobilize\\transfers
Admin[1]=root
GroupName=dtm
AgentPort=49221
EventServerPort=5222
RulesServerPort=49223
SchedulerPort=49229
CAOrgName=#{signiantOrgName}
Locality=City
StateProv=State
CountryCode=US
OrgUnit=Certificate Authority
CACommonName=#{machineName} DTM Certificate Authority
WebAdminPasswd=hmfmaroznyixizezgtkkjwdzfzapbmet
CAAdminPassPhrase=hmfmaroznyixizezgtkkjwdzfzapbmet
CAPassPhrase=bzlxpvcmcvgwirarktjmmymqlypqotpsevhwemonoxencpjxjqdnbvdnmvbwnkezercwexnyflbspvfmfrbqbvhxamhwixav
EOF

    # Install the manager
    echo Starting Signiant software install
    /Releases/#{managerVersion}.0_SIGNIANT/bundles/Latest/DTM_Linux_RH6_64/dtm/sigsetup -type=manager -mode=silent
SHELL

end
