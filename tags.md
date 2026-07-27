# Ansible Tags for Aardsound

[Ansible tags](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_tags.html)
are used to control which parts of Aardsound are executed when you run the playbook.  Tags are
applied to various tasks in the roles used by Aardsound.
The default tag used by Ansible is `all` which means execute every task unless it has the `never`
tag.
If you apply the `-t` or `--tags` option followed by a comma-separated list of tags, Ansible will
only execute tasks tagged with one of the tags in the list and any tags with the `always` tag.
If you apply the `--skip-tags` option followed by a comma-separated list of tags, Ansible will
not execute tasks tagged with one of the tags in the list; `--skip-tags` overrides `--tags`.
Tags that are not present in the playbook are silently ignored: typos do not cause errors.

The `--list-tags` option doesn't execute the playbook: it lists the available tags instead
```
steve@trillian:~/shared/ansible/aardsound$ ansible-playbook --list-tags aardsound.yml 

playbook: aardsound.yml

  play #1 (aardsound): Setup multi-room Spotify and/or Mopidy on a RasPiOS server wih a sound card	TAGS: []
      TASK TAGS: [always, apt, asound, assert, detect, initial, install, install-ffmpeg, install-mopidy, install-raspotify, install-snapclient, install-snapserver, mopidy, mopidy-multiroom, nfs, patch, pi, pip, scontrols, snapclient, spotify, spotify-multiroom]
```

In alphabetical order, these tags have the following purposes:
Those marked * have an accompanying `always` tag
| tag | Controls |
|-----|---------|
| `always` | as explained above
| `activate` | See notes
| `apt` | APT installation of software packages
| `asound` | Configuration of `/etc/asound.conf`, e.g., to create the `aardmixer` PCM
| `assert` | Whether Ansible [assert](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/assert_module.html) tasks are performed
| `deactivate` | See notes
| `detect` | Whether an attempt is made to detect a default playback sound card 
| `initial` | Whether to run "one-off" initial setup tasks: equivalent to `install,pi,patch`
| `install` | Installation, equivalent to `apt,pip` or to all of the `install-*` tags
| `install-ffmpeg` | APT installation of FFMPEG
| `install-mopidy` | APT and/or PIP installation of Mopidy and itse extensions
| `install-raspotify` | APT setup and installation of the Raspotify repository and package
| `install-snapclient` | APT installation of the Snapserver client package
| `install-snapserver` | APT installation of the Snapserver client package
| `mopidy` | Installation and creation/removal of configuration or removal of Mopidy and extensions
| `mopidy-multiroom` | Ditto, but includes snapserver and configuration is for the multi-room instance
| `nfs` | Configuration of an NFS server (or NAS) for the Mopidy local extension
| `patch` | APT upgrade
| `pi` | Pi-specific configuration, notably of `/boot/firmware/config.txt`
| `pip` | PIP installation of software packages
| `scontrols` | Configuration of ALSA controls for specific playback devices
| `snapclient` | Installation and creation/removal of configuration of snapclient to connect to multi-room services
| `spotify` | Installation (as per `raspotify`) and creation/removal of configuration of librespot for Spotify
| `spotify-multiroom` | Ditto, but includes FFMPEG & snapserver and configuration is for the multi-room instance

**Notes:**
- When re-configuring the already-installed service, `--skip-tags initial` can save execution time
  provided that all of the software packages are installed.
- The `activate` and `deactivate` tags apply to the creation and removal, respectively, of 
  configuration and service files for **Spotify**, **Mopidy** and **Snapcast**

  When reconfiguring existing services, `--skip-tags deactivate` or
  `--skip-tags initial,deactivate` can save execution time spent on identifying files and services
  for removal that don't exist
- Many tasks are associated by multiple tags, e.g., the installation of the `raspotify` package is
  covered by any of the following tags: `initial`, `install`, `install-raspotify`, `spotify` and
  `spotify-multiroom`.
  The `-t` and `--skip-tags` options can be used to control this more finely, e.g.,
  `-t spotify --skip-tags install` will configure Spotify but won't attempt to install the
  `raspotify` package (whether this works is a good idea)
- There is no `reboot` tag.
  Rebooting is performed by an Ansible
  [handler](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_handlers.html)
  and handlers ignore tags.
  Reboots can be prevented by setting the variable `aardsound_reboot` to false, for example with
  `-e ansible_reboot=false`.
  
  Suppressing required reboots is not advised, if you encounter problems, reboot the device and re-run
  the playbook without supressing further reboots.

