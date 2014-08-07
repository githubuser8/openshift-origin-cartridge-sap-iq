SAP-IQ
=======

Cartridge for SAP IQ

Installation
============
Install as a Downloadable Cartridge, please see https://www.openshift.com/developers/download-cartridges

Important OpenShift Enterprise 2.0 System Settings
==================================================
The following settings are used to make the installation succeed:

1. OpenShift Resource Limits
In /etc/openshift/resource_limits.conf 

quota_files=300000
quota_blocks=18874368 #18GB
max_active_gears=100
limits_nproc=10000
limits_nproc=500        # max number of processes
cpu_shares=1024
cpu_cfs_quota_us=300000
memory_limit_in_bytes=2147483648
memory_memsw_limit_in_bytes=2684354560 # 2048M + 512M (512M swap)

2. Increase the Hard Quota for Gear User <gear_uuid>
Increase the hard quota of gear user to the same as 'quota_blocks' as in previous step. A gear user is identified by the uuid of the gear. For example for gear with uuid 52cb2708d573fbb051000001, the gear userid is 52cb2708d573fbb051000001
# edquota 52cb2708d573fbb051000001 
Disk quotas for user 52cb2708d573fbb051000001 (uid 1000):
  Filesystem                   blocks       soft       hard     inodes     soft     hard
  /dev/sda1                   3960408          0    18874368       9526        0   200000

3. Time Out
SAP IQ as an enterprise database takes about 10 minutes to install, thus the default settings will time out. To make installation successful, please adjust the following time out settings:
1)  Increase the mcollective timeout:
      Component:     broker
      File Location: /etc/openshift/plugins.d/openshift-origin-msg-broker-mcollective.conf
      Directive:     MCOLLECTIVE_TIMEOUT=800

      About:         This timeout is for every mcollective action that is sent from the broker. Any cartridge or gear actions will get this timeout.

2) Increase the apache timeout:
     Component:      broker
     File Location:  /etc/httpd/conf.d/000002_openshift_origin_broker_proxy.conf
     Directive:      ProxyTimeout 800

     Component:      node
     File Location:  /etc/httpd/conf.d/000001_openshift_origin_node.conf
     Directive:      ProxyTimeout 800

     About:          This timeout is for every http request that comes through the node. 

3) Increase the node agent timeout:
     Component:      node
     File Location:  /opt/rh/ruby193/root/usr/libexec/mcollective/mcollective/agent/openshift.ddl
     Directive:      :timeout     => 800   # this is under 'metadata'

     About:          This timeout is specifically for actions using the 'openshift' mcollective agent.

4) Increase the hard-coded hourglass timeout:
     Component:      node
     File Location:  /opt/rh/ruby193/root/usr/libexec/mcollective/mcollective/agent/openshift.rb

     Find the following method by searching: "get_app_container_from_args(args)"
     In this method, you will see the following (the last thing before the method end):
        namespace, quota_blocks, quota_files, OpenShift::Runtime::Utils::Hourglass.new(235))
     Change the '235' to '800':
        namespace, quota_blocks, quota_files, OpenShift::Runtime::Utils::Hourglass.new(800))

     About:          This timeout is for all cartridge/gear actions made on a cartridge/gear AFTER creation.

5) Increase the rhc timeout:
     Component:      client
     File Location:  ~/.openshift/express.conf
     Directive:      timeout=800

    About:          This is how long the rhc client will wait for the action to complete before giving up.
 
After the above changes have been made, please reboot the nodes and restart the `openshift-broker` service on the broker:
# service openshift-broker restart

