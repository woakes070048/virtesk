# Configuring virtual rooms

The following steps explain the config file of amoothei-vm-rollout, focusing on how to define and configure virtual rooms.

-------------

## Config structure
Syntax of `amoothei-vm-rollout.conf`:
```
# This is a comment
[section1]
	key1 = value1
	key2 = value2
[section2]
	[[subsection 1]]
	key3 = "value" # values can optionally be enclosed in double quotes
	[[subsection 2]]
	# empty subsection
```
For syntax details, please refer to [ConfigObj library documentation](http://www.voidspace.org.uk/python/configobj.html)

Structure of `amoothei-vm-rollout.conf`:
```
[general]
	# General Tool configuration
	[[connect]]
	# Parameters for connecting to Ovirt REST API	

[logging]
	# Configuration of logging library

[room room01]  # Defines a virtual room with name "room01"
	[[student_vms]]
	# Complete definition of all VMs

	[[teacher_vms]]
	# Complete definition of all VMs

	[[other_vms]]
	# Complete definition of all VMs
	
[room room02]  # Defines a virtual room with name "room02"
	[[vms]]
	# Complete definition of all VMs
```

All configuration is mandatory. No configuration may be omitted. There are no default values.


File locations: 

* Absolute paths are valid
* all other paths are handled relative to the location of `amoothei-vm-rollout.conf`.


## General configuration: section [general]
```
[general]
        sftp_floppy_upload_cmd = "(echo put {0}; echo chmod 666 {1}; echo ls -l {1}) | sftp sftp-floppy-upload@infrastructure-server:/floppy/"
        sftp_floppy_cleanup_cmd = "echo rm {0} | sftp sftp-floppy-upload@infrastructure-server:/floppy/"
        ovirt_worker_floppy_prefix = "/rhev/data-center/mnt/..../floppy"
        [[connect]]
	# ...
```

## Ovirt REST-API connection parameters: section [general][[connect]]
```
[general]
	# ...
        [[connect]]
        url = "https://ovirt-manager/api"
        username = "admin@internal"
        password = "PASSWORD"
        ca_file = "ca.crt"
```

* **url:** URL used to connect to REST-API of your RHEV/oVirt Manager. 
* **username** and **password:** Credentials of some Administrator account in Ovirt/RHEV. This account is used for all operations on VMs.
* **ca_file:** Path to CA certificate file. Used for validating SSL connection. Can be downloaded from RHEV/Ovirt Manager: `https://ovirt-manager/ca.crt`

## Logging: section [logging]
```
[logging]
config_file=logging.conf
log_file=logs/rollout.log
```

* **config_file:** Path to `logging.conf`
* **log_file:** Path to logfile

Sample `logging.conf`:
```
[loggers]
keys		= root

[logger_root]
level		= DEBUG
handlers	= console,file

[formatters]
keys		= simple,complex

[formatter_simple]
format		= %(asctime)s - %(name)s - %(levelname)s - %(message)s

[formatter_complex]
format		= %(asctime)s - %(name)s - %(levelname)s - %(module)s : %(lineno)d - %(message)s

[handlers]
keys		= file,console

[handler_file]
class		= FileHandler
formatter	= complex
level		= DEBUG
args 		= (r'%(log_file)s', r'%(file_mode)s')

[handler_console]
class		= StreamHandler
formatter	= simple
level		= DEBUG
args		= (sys.stdout,)
```

Details: see https://docs.python.org/2/library/logging.config.html#configuration-file-format

## Room definitions: section [room room01]
Structure:
```
[room room01]  # Defines a virtual room with name "room01"
    [[student_vms]]
    # Complete definition of all VMs

    [[teacher_vms]]
    # Complete definition of all VMs

    [[other_vms]]
    # Complete definition of all VMs

[room room02]  # Defines a virtual room with name "room02"
    [[vms]]
    # Complete definition of all VMs
```


Room definition:
```
[room test01]
	[[student_vms]]
		# Numbering and naming
		ids = "range(1,10+1)"
		names = "${roomname}-vd${id}"
	
		# Network
		network_name="TEST-NET-1"
		ip_addresses = "192.0.2.${suffix}"
		ip_addresses_suffix = 1
		netmask_suffix=24
		default_gateway=10.165.208.1

		# VM
		cluster = Default
		template_name = "win7-goldimage-01"
		memory = 4 * 1024 * 1024 * 1024
		os="windows_7x64"
		description = "student VM"
		timezone="W. Europe Standard Time"
		usb = enabled
		stateless = False

		# Windows Autounattend
		autounattend_templatefile = "Autounattend-production.xml.template"
		workaround_os="rhel_7x64"
		workaround_timezone="Etc/GMT"
	
		# Permissions
		tc_user = "ovirt.thinclient@ourdomain"

		# Snapshots
		snapshot_description = "Automatic snapshot after amoothei-vmrollout, IP=${ip}/${netmask_as_suffix}, scripttime=${scripttime}"
		reset_to_snapshot_regex = "Automatic snapshot after amoothei-vmrollout, .*"

		# Starting VM
		rollout_startvm = True
		reset_startvm = Auto
```

Numbering and naming:

* **ids:** FIXME
* **names:** FIXME

Network:

* **network_name:** Name of Ovirt network to attach to VM.
* **ip_adresses:** IP address to configure. This parameter is computed, but not used, by amoothei-vm-rollout. It can be used inside Autounattend.xml to configure a static IP adress. FIXME
* **ip_adresses_suffix:** Suffix of the first VM. FIXME
* **netmask_suffix:** Prefix length of network mask. Passed directly to Autounattend.xml.
    + `netmask_suffix=24` ===> same as netmask 255.255.255.0
    + `netmask_suffix=21` ===> same as netmask 255.255.248.0
* **default_gateway**: Passed directly to Autounattend.xml

VM:

* **cluster**: Ovirt cluster for creating VMs.
* **template_name**: Name of Ovirt template (e.g. windows goldimage) to use for creating VMs
* **memory**: RAM of virtual machine in Bytes.
* **os**: Operating System to assign to this VM after Autounattend completed. This is the OS as configured in Ovirt (Edit VM dialog).
* **timezone**: Timezone to assign to this VM after Autounattend completed. This is the Timezone as configured in Ovirt (Edit VM dialog). Please note: Ovirt uses different timezone names for Linux than for Windows VMs.
* **description**: VM description
* **usb**: Shall USB be enabled for the VM? Valid values: *enabled*, *disabled*
* **stateless**: Shall the VM be [stateless](stateless_and_snapshot_features.md)? Valid values: *True*, *False*.

Windows Autounattend:

* **autounattend_templatefile**: Path to Mako template for [Autounattend.xml](Autounattend.md).
* **workaround_os**: [Workaround Operating System](workaround_os.md) to use during Autounattend process. Can be any valid OS available in Ovirt.
* **workaround_timezone**: [Workaround timezone](workaround_os.md) to use during Autounattend process. Must be a timezone that is valid for the configured *workaround_os*.

Permissons:

* **tc_user**: Technical account used by TCs to access VMs. After rollout, permissions are granted to *tc_user*.

Snapshots:

* **snapshot_description**: If empty, no snapshot is created. Otherwise, a [snapshot](stateless_and_snapshot_features.md) with the configured description is created after rolling out VMs. 
* **reset_to_snapshot_regex**: Must be a valid python regex. Used by amoothei-virtroom-reset to identify the [snapshot](stateless_and_snapshot_features.md) to reset the VM to. The regex is applied to the snapshot description.

Starting VMs:

* **rollout_startvm:** If *True*, VMs will be started after rolling out a virtual room.
* **reset_startvm:** If *Always*, VMs will always be started after resetting a virtual room to their snapshots. If *Auto*, VMs will be started if they were running before.


