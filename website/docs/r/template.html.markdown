---
layout: "opennebula"
page_title: "OpenNebula: opennebula_template"
sidebar_current: "docs-opennebula-resource-template"
description: |-
  Provides an OpenNebula template resource.
---

# opennebula_template

Provides an OpenNebula template resource.

This resource allows you to manage templates on your OpenNebula clusters. When applied,
a new template is created. When destroyed, this template is removed.

## Example Usage

```hcl
resource "opennebula_template" "example" {
  name        = "template"
  description = "VM template"
  cpu         = 1
  vcpu        = 1
  memory      = 1024
  group       = "terraform"
  permissions = "660"

  context = {
    NETWORK      = "YES"
    HOSTNAME     = "$NAME"
    START_SCRIPT = "yum upgrade"
  }

  graphics {
    type   = "VNC"
    listen = "0.0.0.0"
    keymap = "fr"
  }

  os {
    arch = "x86_64"
    boot = "disk0"
  }

  cpumodel {
    model = "host-passthrough"
  }

  disk {
    image_id = opennebula_image.example.id
    size     = 10000
    target   = "vda"
    driver   = "qcow2"
  }

  nic {
    model           = "virtio"
    network_id      = var.vnetid
    security_groups = [opennebula_security_group.example.id]
  }

  vmgroup {
    vmgroup_id = 42
    role       = "vmgroup-role"
  }

  sched_requirements = "FREE_CPU > 60"

  user_inputs = {
    BLOG_TITLE = "M|text|Blog Title",
  }

  tags = {
    environment = "example"
  }

  template_section {
   name = "example"
   elements = {
      key1 = "value1"
   }
  }
}
```

## Argument Reference

The following arguments are supported:

* `name` - (Required) The name of the virtual machine template.
* `description`: (Optional) The description of the template.
* `permissions` - (Optional) Permissions applied on template. Defaults to the UMASK in OpenNebula (in UNIX Format: owner-group-other => Use-Manage-Admin).
* `group` - (Optional) Name of the group which owns the template. Defaults to the caller primary group.
* `cpu` - (Optional) Amount of CPU shares assigned to the VM. **Mandatory if `template_****id` is not set**.
* `vcpu` - (Optional) Number of CPU cores presented to the VM.
* `memory` - (Optional) Amount of RAM assigned to the VM in MB. **Mandatory if `template_****id` is not set**.
* `features` - (Optional) See [Features parameters](#features-parameters) below for details.
* `context` - (Optional) Array of free form key=value pairs, rendered and added to the CONTEXT variables for the VM. Recommended to include: `NETWORK = "YES"` and `SET_HOSTNAME = "$NAME"`.
* `cpumodel` - (Optional) See [CPUmodel parameters](#cpumodel-parameters) below for details.
* `graphics` - (Optional) See [Graphics parameters](#graphics-parameters) below for details.
* `os` - (Optional) See [OS parameters](#os-parameters) below for details.
* `disk` - (Optional) Can be specified multiple times to attach several disks. See [Disks parameters](#disks-parameters) below for details.
* `nic` - (Optional) Can be specified multiple times to attach several NICs. See [Nic parameters](#nic-parameters) below for details.
* `nic_alias` - (Optional) Can be specified multiple times to attach several NIC Aliases. See [Nic alias parameters](#nic-alias-parameters) below for details.
* `raw` - (Optional) Allow to pass hypervisor level tuning content. See [Raw parameters](#raw-parameters) below for details.
* `vmgroup` - (Optional) See [VM group parameters](#vm-group-parameters) below for details. Changing this argument triggers a new resource.
* `user_inputs` - (Optional) Ask the user instantiating the template to define the values described.
* `sched_requirements` - (Optional) Scheduling requirements to deploy the resource following specific rule
* `sched_ds_requirements` - (Optional) Storage placement requirements to deploy the resource following specific rule.
* `tags` - (Optional) Map of tags (`key=value`) assigned to the resource. Override matching tags present in the `default_tags` atribute when configured in the `provider` block. See [tags usage related documentation](https://registry.terraform.io/providers/OpenNebula/opennebula/latest/docs#using-tags) for more information.
* `template` - (Deprecated) Text describing the OpenNebula template object, in Opennebula's XML string format.
* `lock` - (Optional) Lock the template with a specific lock level. Supported values: `USE`, `MANAGE`, `ADMIN`, `ALL` or `UNLOCK`.
* `template_section` - (Optional) Allow to add a custom vector. See [Template section parameters](#template-section-parameters)

### Graphics parameters

`graphics` supports the following arguments:

* `type` - (Required) Generally set to VNC.
* `listen` - (Required) Binding address.
* `port` - (Optional) Binding Port.
* `keymap` - (Optional) Keyboard mapping.
* `passwd` - (Optional) VNC's password, conflicts with random_passwd.
* `random_passwd` - (Optional) Randomized VNC's password, conflicts with passwd.

### OS parameters

`os` supports the following arguments:

* `arch` - (Required) Hardware architecture of the Virtual machine. Must fit the host architecture.
* `boot` - (Optional) `OS` disk to use to boot on.
* `machine` - (Optional) libvirt machine type.
* `kernel` - (Optional) Path to the `OS` kernel to boot the image in the host.
* `kernel_ds` - (Optional) Image to be used as kernel. (see !!)
* `initrd` - (Optional) Path to the initrd image in the host.
* `initrd_ds` - (Optional) Image to be used as ramdisk. (see !!)
* `root` - (Optional) Device to be mounted as root.
* `kernel_cmd` - (Optional) Arguments for the booting kernel.
* `bootloader` - (Optional) Path to the bootloader executable.
* `sd_disk_bus` - (Optional) Bus for disks with sd prefix, either `scsi` or `sata`, if attribute is missing, libvirt chooses itself.
* `uuid` - (Optional) Unique ID of the VM.
* `firmware` - (Optional) Firmware type or firmware path. Possible values: `BIOS` or path for KVM, `BIOS` or `UEFI` for vCenter.
* `firmware_secure` - (Optional) Enable Secure Boot. (Can be `true` or `false`).
* (!!) Use one of `kernel_ds` or `kernel` (and `initrd` or `initrd_ds`).

### Features parameters

`features` supports the following arguments

* `pea` - (Optional) Physical address extension mode allows 32-bit guests to address more than 4 GB of memory. (Can be `YES` or `NO`)
* `acpi` - (Optional) Useful for power management, for example, with KVM guests it is required for graceful shutdown to work. (Can be `YES` or `NO`)
* `apic` - (Optional) Enables the advanced programmable IRQ management. Useful for SMP machines. (Can be `YES` or `NO`)
* `localtime` - (Optional) The guest clock will be synchronized to the host’s configured timezone when booted. Useful for Windows VMs. (Can be `YES` or `NO`)
* `hyperv` - (Optional) Add hyperv extensions to the VM. The options can be configured in the driver configuration, HYPERV_OPTIONS.
* `guest_agent` - (Optional) Enables the QEMU Guest Agent communication. This only creates the socket inside the VM, the Guest Agent itself must be installed and started in the VM. (Can be `YES` or `NO`)
* `virtio_scsi_queues` - (Optional) Numer of vCPU queues for the virtio-scsi controller.
* `iothreads` - (Optional) umber of iothreads for virtio disks. By default threads will be assign to disk by round robin algorithm. Disk thread id can be forced by disk IOTHREAD attribute.

### CPUModel parameters

`cpumodel` supports the following argument:

* `model` - (Required) CPU model to be used on the virtual machine.

### Disk parameters

`disk` supports the following arguments

* `image_id` - (Optional) ID of the image to attach to the virtual machine. Conflicts with `volatile_type` and `volatile_format`.
* `size` - (Optional) Size (in MB) of the image attached to the virtual machine. Not possible to change a cloned image size.
* `target` - (Optional) Target name device on the virtual machine. Depends of the image `dev_prefix`.
* `driver` - (Optional) OpenNebula image driver.
* `dev_prefix` - (Optional) Prefix for the emulated device this image will be mounted at. For instance, attribute of the Image will be used.
* `cache` - (Optional) Selects the cache mechanism for the disk. Values are default, none, writethrough, writeback, directsync and unsafe.
* `discard` - (Optional) Controls what’s done with with trim commands to the disk, the values can be ignore or discard.
* `io` - (Optional) Set IO policy. Values are threads, native.
* `volatile_type` - (Optional) Type of the volatile disk: `swap` or `fs`. Type `swap` is not supported in vcenter. Conflicts with `image_id`.
* `volatile_format` - (Optional) Format of the volatile disk: `raw` or `qcow2`. Conflicts with `image_id`.

Minimum 1 item. Maximum 8 items.

### NIC parameters

`nic` supports the following arguments

* `name` - (Optional) The name of the NIC. This could be used for reference the NIC as a parent of a NIC Alias.
* `network_id` - (Optional) ID of the virtual network to attach to the virtual machine.
* `ip` - (Optional) IPv4 of the virtual machine on this network.
* `ip6` - (Optional) IPv6 of the virtual machine on this network.
* `ip6_ula` - (Optional) IPv6 ULA (Unique Local Address) of the virtual machine on this network.
* `ip6_global` - (Optional) IPv6 global address of the virtual machine on this network.
* `ip6_link` - (Optional) IPv6 link-local address of the virtual machine on this network.
* `mac` - (Optional) MAC of the virtual machine on this network.
* `model` - (Optional) Nic model driver. Example: `virtio`.
* `virtio_queues` - (Optional) Virtio multi-queue size. Only if `model` is `virtio`.
* `physical_device` - (Optional) Physical device hosting the virtual network.
* `security_groups` - (Optional) List of security group IDs to use on the virtual network.
* `method` - (Optional) Method of obtaining IP addresses (empty or `static`, `dhcp`, `skip`).
* `gateway` - (Optional) Default gateway set for the NIC.
* `dns` - (Optional) DNS server set for the NIC. **Please make sure `INHERIT_VNET_ATTR="DNS"` is added to `/etc/one/oned.conf`.**
* `network_mode_auto` - (Optional) A boolean letting the scheduler pick the Virtual Networks the VM NICs will be attached to. Can only be used at VM creation.
* `sched_requirements` - (Optional) A boolean expression to select virtual networks (evaluates to true) to attach the NIC,  when `network_mode_auto` is true. Can only be used at VM creation.
* `sched_rank` - (Optional) Arithmetic expression to sort the suitable Virtual Networks for this NIC, when `network_mode_auto` is true. Can only be used at VM creation.

Minimum 1 item. Maximum 8 items.

### NIC Alias parameters

`nic_alias` supports the following arguments

* `parent` - The parent NIC name.
* `name` - (Optional) The name of the NIC Alias.
* `network` - (Optional) The name of the virtual network to attach to the virtual machine.
* `network_id` - (Optional) ID of the virtual network to attach to the virtual machine.
* `ip` - (Optional) IP of the virtual machine on this network.
* `ip6` - (Optional) IPv6 of the virtual machine on this network.
* `ip6_ula` - (Optional) IPv6 ULA (Unique Local Address) of the virtual machine on this network.
* `ip6_global` - (Optional) IPv6 global address of the virtual machine on this network.
* `ip6_link` - (Optional) IPv6 link-local address of the virtual machine on this network.
* `mac` - (Optional) MAC of the virtual machine on this network.
* `security_groups` - (Optional) List of security group IDs to use on the virtual network.
* `method` - (Optional) Method of obtaining IP addresses (empty or `static`, `dhcp`, `skip`).
* `gateway` - (Optional) Default gateway set for the NIC.
* `dns` - (Optional) DNS server set for the NIC. **Please make sure `INHERIT_VNET_ATTR="DNS"` is added to `/etc/one/oned.conf`.**

### Raw parameters

`raw` supports the following arguments:

* `type` - (Required) - Hypervisor. Supported values: `kvm`, `lxd`, `vmware`.
* `data` - (Required) - Raw data to pass to the hypervisor.

### VM group parameters

`vmgroup` supports the following arguments:

* `vmgroup_id` - (Required) ID of the VM group to use.
* `role` - (Required) role of the VM group to use.

### Template section parameters

`template_section` supports the following arguments:

* `name` - (Optional) The vector name.
* `elements` - (Optional) Collection of custom tags.

## Attribute Reference

The following attribute are exported:

* `id` - ID of the template.
* `uid` - User ID whom owns the template.
* `gid` - Group ID which owns the template.
* `uname` - User Name whom owns the template.
* `gname` - Group Name which owns the template.
* `reg_time` - Registration time of the template.
* `tags_all` - Result of the applied `default_tags` and then resource `tags`.
* `default_tags` - Default tags defined in the provider configuration.

## Import

`opennebula_template` can be imported using its ID:

```shell
terraform import opennebula_template.example 123
```
