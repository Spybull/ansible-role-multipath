# Ansible Role: Device Mapper Multipath

An Ansible Role that installs multipath on RHEL/CentOS, Debian/Ubuntu.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
multipath_enablerepo: ""
```

The repository to use when installing multipath (only used on RHEL/CentOS systems). If you'd like later versions of multipath than are available in the OS's core repositories, use a custom repository.

```yaml
multipath_dir: "/etc/multipath"
multipath_config_path: "/etc/multipath.conf"
multipath_dropin_path: "/etc/multipath/conf.d"
```

The default multipath directory, the main multipath configuration file, and the default drop-in directory for additional configuration files.

```yaml
multipath_service: multipathd
```

The name of the multipath daemon's systemd service.

```yaml
multipath_config:
  defaults:
    config_dir: "{{ multipath_dropin_path }}"
  multipaths:
    multipath:
```
The default configuration of the **multipath.conf** file. This configuration will be merged with the configuration from the `multipath_files` variable.  
The default configuration also includes `config_dir` parameter with the `multipath_dropin_path` as the default variable.  
Note that this is not necessary in the latest versions of multipath, as the `config_dir` parameter has been deprecated.  
See: https://github.com/opensvc/multipath-tools/pull/31


```yaml
multipath_packages:
  - [platform-specific]
```

The list of packages to be installed. This defaults to a set of platform-specific packages for RedHat or Debian-based systems (see `vars/RedHat.yml` and `vars/Debian.yml` for the default values).

```yaml
multipath_state: started
```

Set initial multipath daemon state to be enforced when this role is run. This should generally remain `started`, but you can set it to `stopped` if you need to fix the multipath config during a playbook run or otherwise would not like multipath started at the time this role is run.

```yaml
multipath_enabled: true
```

Set the multipath daemon service boot time status. This should generally remain `true`, but you can set it to `false` if you need to run Ansible while leaving the service disabled.

```yaml
multipath_restart_state: reloaded
```

Set the multipath daemon state after updates to configuration files. The `reloaded` state causes
the daemon to re-read the configuration files. You can also set this option to `restart`, which will fully restart the multipath daemon.

```yaml
multipath_packages_state: present
```

If you have enabled any additional repositories, you may want an easy way to upgrade versions. You can set this to `latest` (combined with `multipath_enablerepo` on RHEL) and can directly upgrade to a different multipath version from a different repo (instead of uninstalling and reinstalling multipath).


## Dependencies

None.


## Example inventory
```yaml
all:
  vars:
    multipath_files:
      # /etc/multipath.conf
      multipath:
        defaults:
          verbosity: 4
          polling_interval: 10
          max_polling_interval: 30
          reassign_maps: "no"
          prio: "sysfs"

        blacklist:
          devnode:
            - "!^(sd[a-z]|dasd[a-z])"
            - "^sd[a-z]"

          wwid:
            - "26353900f02796769"

          device:
            - vendor: "IBM"
              product: "3S42"

            - vendor: "DELL"
              product: "Universal Xport"

          property:
            - "ID_ATA"

          protocol:
            - "scsi:unspec"
            - "undef"
        
        blacklist_exceptions:
          wwid:
            - "3600d0230000000000e13955cc3757803"
          property:
            - "(SCSI_IDENT_|ID_WWN)"

        devices:
          - device:
              vendor: "NVME"
              product: "Pure Storage FlashArray"
              path_selector: "queue-length 0"
              path_grouping_policy: group_by_prio
              prio: ana
              failback: immediate
              fast_io_fail_tmo: 10
              user_friendly_names: "no"
              no_path_retry: 0
              features: 0
              dev_loss_tmo: 60

        multipaths:
          multipath:
            - wwid: 3624a93705598f4dec0624d4e00001020
              alias: EXAMPLE01
              prio: const
            - { wwid: "3624a93705598f4dec0624d4e00001021", alias: "EXAMPLE02" }

        overrides:
          dev_loss_tmo: 60
          fast_io_fail_tmo: 8
          protocol:
            type: "scsi:iscsi"
            dev_loss_tmo: 60
            fast_io_fail_tmo: 120

      dropin_file01:
        multipaths:
          multipath:
            - { wwid: "3624a93705598f4dec0624d4e00001022", alias: "EXAMPLE03" }
            - { wwid: "3624a93705598f4dec0624d4e00001023", alias: "EXAMPLE04" }

      dropin_file02:
        multipaths:
          multipath:
            - { wwid: "3624a93705598f4dec0624d4e00001024", alias: "EXAMPLE05" }
            - { wwid: "3624a93705598f4dec0624d4e00001025", alias: "EXAMPLE06" }
databases:
  hosts:
    db01:
      ansible_host: db01.example.com
```

## Example playbook

```yaml
- hosts: databases
  roles:
    - { role: Spybull.multipath }
```

## License

MIT / BSD

## Author Information

This role was created in 2024 by [spybull](https://spybull.github.io/).
