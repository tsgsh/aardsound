# Aardsound

Ansible Infrastructure as Code setup of (possibly multi-room) Spotify Connect, Mopidy, on a
Raspberry Pi (any model with ARMv7 or ARMv8 hardware) running RasPiOS and with a suitable sound
card driven by ALSA.

**aardsound** is intended to be used on newly‑built Raspberry Pi OS image, however it is written to be
idempotent, assuming the same inventory variables (and/or any extra variables) are passed to it, so
re‑running it should work as expected.

## Meaning of **Aardsound** (or **AARDSound**)

**Automated Ansible RasPiOS Deployment of Sound**...

Only joking, that is not why it's called **Aardsound**, it's because my Raspberry Pi's are all named
after characters from Aardman Animation's Wallace and Gromit films
(and [Morph](https://en.wikipedia.org/wiki/Morph_(TV_series))).


This is a project I started to learn how to use **Ansible**.
It was refactored in 2026 after the multi‑room element stopped working: this improved the strudture,
removed hundreds of linting errors, added support for Debian Trixie, dropped code that was specific
to Debian Bullseye and older (e.g., code to compile [librespot](
  https://github.com/librespot-org/librespot) from source).

## Components

The primary purpose of **Aardsound** is to automate the deployment of audio software to Raspberry
Pis connected to Hi‑Fi systems.
There are three main parts:

- [Spotify Connect](https://connect.spotify.com/) &ndash; specifically for listening to Spotify
  Premium (a free Spotify account won't work);
  **Aardsound** uses the [Raspotify](https://dtcooper.github.io/raspotify/) package, which
  provides [librespot](https://github.com/librespot-org/librespot) pre‑packaged for Debian‑based
  systems
- [Mopidy](https://mopidy.com/) &ndash; as a more general and extensible music server (Mopidy is 
  not used for Spotify support, but in theory you could do that if you wanted to)
- [Snapcast](https://github.com/snapcast/snapcast) for optional synchronous multiroom audio

Bluetooth will be added in future (as a source and a sink, ideally)

All of these drive the Advanced Linux Sound Architecture (ALSA) subssytem (Aardsound does not
use PulseAudio, Pipewire or JACK).

## Target Audio 

Aardsound is intended to configure one or more Raspberry Pi servers to play music from **Spotify**
or [**Mopidy**](https://mopidy.com/) via an attached sound card and speakers.
For multi‑room audio, **Snapcast** clients can connect each Raspberry Pi to one or more **Snapcast**
servers.
**Snapcast** servers can run alongside **Snapcast** clients or on a dedicated Raspberry Pi or a
Debian Linux server.

## Invocation
Once the [**Ansible** Inventory](#ansible-inventory) has been set up, Raspberry Pis are connected
to the network with known hostnames (in DNS or `/etc/hosts`) then **Aardsound** is installed and
configured using the command, from the directory where **Aardsound** is installed:
```BASH
steve@linux:~/github/aardsound $ ansible‑playbook aardsound.yml
```
The option `‑‑limit` or `‑l` may be useful if you want to install it on one of several possible
targets.

Note that, the playbook doesn't set up any of the sound capabilities.  To do that you need to
configure **Ansible** inventory variables as explained in the
[example configurations](./Examples.md).

These explain how the various components deployed by **Aardsound** combine to create different
single-, multi‑room and hybrid combinations of Spotify and Mopidy.
Read this if you are unsure what the best audio setup is for you: there are 12 example setups with
a corresponding Ansible inventories and the variables needed to deploy that configuration.

**The point of Aardsound** is to make the configuration of your environment to match one of these
(or some other setup you need) a matter of configuring a few simple Ansible inventory variables
then running one `ansible‑playbook` command and waiting for the magic (technically,
[Infrastructure-as-Code](https://en.wikipedia.org/wiki/Infrastructure_as_code) (IaC) automation)
to happen.

## Installation

You need a computer that can serve the **Ansible control node**, which includes Linux, BSD, macOS
and Windows using the
[Windows Subsystem for Linux (WSL)](https://docs.microsoft.com/en-us/windows/wsl/about).
See the [Ansible Installation Guide](https://docs.ansible.com/projects/ansible/latest/installation_guide/intro_installation.html#installation-guide) for more information.
The control node could be one of the managed nodes, although that has not been tested.

On the Ansible control node you need to install a
[current version of Ansible](https://docs.ansible.com/projects/ansible/latest/reference_appendices/release_and_maintenance.html).
At the time of writing, that was Ansible 13 (based on **ansible-core** 2.20).
Older versions are not supported but will probably work back to **ansible-core** 2.14
(Ansible 7).

If your Linux distribution has a `ansible‑core` package instead of an `ansible` package (e.g., 
RHEL and its clones) that will work (subject the provisos about versions above), but you will need
to install supported versions of the following Ansible collections to be
installed:
- **ansible.posix**
- **ansible.utils**
- **community.general**

These are available as packages in the **EPEL** repoitory as **ansible-collection-ansible-posix**,
etc.

If the **ansible-core** version is 2.18 or earlier, native Jinja2 templating operations must be set
(using the environment variable `ANSIBLE_JINJA2_NATIVE=true` or the Ansible configuration option
`DEFAULT_JINJA2_NATIVE=true`).

### Downloading Aardsound

Download from Github, e.g. with `git clone`, to a directory, e.g. `~/aardsound`: this directory will
be the location for running the `ansible‑playbook` command.

Note that, although **Aardsound** contains a `roles` directory, as described in
[Aardsound Role Structure](#aardsound-role-structure) below, **Aardsound** is *not* an Ansible
Collection.
This is because the roles are not intended for general use in playbooks other than `aardsound.yml`.

### Configuring the Control node

### Ansible Inventory

In addtion, you will need an Ansible inventory file to define the managed nodes and the
inventory variables that will define what is to be deployed to to each target.
See the [**Inventory**](#ansible-inventory) section below.

MacOS may work as a control host operating system, but this has not been tested.

### Ansible forks

If you have more than 5 hosts in your inventory that you might run **Aardsound** against
simultaneously, then consider increasing the number of `forks` available to **Ansible** by setting
a value of `forks` in the `[defaults]` section of your 
[**Ansible** configuration file](https://docs.ansible.com/projects/ansible/latest/reference_appendices/config.html)
from the default value of 5.
Without this, each time the `aardsound` playbook has to executes a task on all hosts, it will do so
on 5 hosts, then repeat the task on another 5 until all are done.
You can also set the Environment Variable `ANSIBLE_FORKS` or use the `‑‑forks` (or `f`) option on
the `ansible‑playbook` command.
Be aware that increasing the number of forks increases the memory usage on the control host.

**Aardsound** does not make use of the Ansible `serial` feature for rolling deployments, because
the individual hosts do not provide the same service as each other.
If you want to deploy changes to a subset of your hosts, use the `‑‑limit` (or `‑l`) option on the
`ansible‑playbook` command.

### Managed Nodes (the audio players)

You need one or more Raspberry Pi's (not the original Model A/A+ or Model B/B+ or Zero/Zero W) with
a USB- or [HAT](https://github.com/raspberrypi/hats/tree/master)-attached Digital-to-Analog
Converter (DAC) or a or Digital Amplifier driving powered or unpowered speakers respectively.
The Raspberry Pi's should be running the RasPiOS "Trixie" (or later) Operating System (the "Lite"
version is acceptable).
The servers *should* have new installations of RasPiOS.

You can also use a Debian Trixie node as a source for multiroom audio.

At a minimum you need *fixed* (or, at least, *known*) IP addresses for all devices for the
installation of **Aardsound** and *fixed* addresses for any nodes that will be sources for
multi‑room audio.
If you have that, but no DNS service, then it is probably a good idea to set up `/etc/hosts` entries
on each device, including the Ansible control node, to map each IP address to a hostname.

Your id on your Ansible control host must be authorised to connect to each managed node using
SSH and to run the `sudo` command with **no** limitations on which privlieged commands it can run
(this is an Ansible [requirement](
https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_privilege_escalation.html#privilege-escalation-must-be-general)).

If your userid on the Ansible control host is not the same as the userid on the target servers
you will need to set the Ansible variable `ansible_user` to the target userid.
This can be done by:
- Setting `ansible_user` as an inventory variable (the most straightforward way)
- Setting the environment variable [`ANSIBLE_REMOTE_USER`](
  https://docs.ansible.com/projects/ansible/latest/reference_appendices/config.html#default-remote-user)
- Setting `remote_user` in the `[defaults]` section of the Ansible [configuration file](
  https://docs.ansible.com/projects/ansible/latest/reference_appendices/config.html#generating-a-sample-ansible-cfg-file)
- Adding the variable to each `ansible` or `ansible‑playbook` command with
  `‑e ansible_user=<my_user>`

If your user does not have an SSH authorized_key on the target, then fix this!  Failing that, if
password login is allowed over SSH, add the option `‑k` to each `ansible` or `ansible‑playbook`
command, but please note that SSH-key-based login is more secure than using a password to login.

If your user does not have the privileges to run the `sudo` command on the managed node without
entering a password, add the option `‑K` to each `ansible` or `ansible‑playbook`command.

For a quick guide to setting up your Raspberry Pi(s) like this, see the
[Quick Guide to Setting up you Raspberry Pi(s) for Aardsound](../PiSetup.md).
See also the [**Limitations**](#limitations) section for the types of Raspberry Pi you can use.

## Major To‑Do Items
- Bluetooth as input and output

## Minor To‑Do Items
- HTTPS support for Mopidy (nginx reverse proxy)

## Limitations
- Spotify/librespot will not work on [ARM](
  https://en.wikipedia.org/wiki/Arm_architecture_family)v6-based  Raspberry Pis, i.e., the original
  Model A and B, the A+ and B+, and the Zero, Zero W and Zero WH.
  This is because recent versions of **librespot** have dropped ARMv6 support.
- No testing has been done with
  - Raspberry Pi 2s (the Zero 2 W has been tested)
  - Raspberry Pi 5s (which would be over‑specified for the need)
  - Raspberry Pi 400 or 500, the former should work and the latter should work if the Raspberry Pi 5
    works
  - Raspberry Pi Compute Modules with IO Boards
  - Physical debian systems (testing has been done with KVM VMs only)
  - Soundcards for Debian systems have not been not tested, but redirected USB sound cards should
    work at least
- 512MB RAM Raspberry Pi models &ndash; the 3A, 3A+ and Zero 2W &ndash; seem to struggle with the
  desktop version of RasPiOS Trixie: use the "Lite" version instead.
  Even with the "Lite" version installed, these are not recommended as **Snapcast** servers because
  they can easily fill the available `/tmp` space (approximately 200M).
- Non-Debian-based Linux distributions are not supported: none of the installation tasks or roles
  will work and the systemd service definitions will be incorrect even if packages are compiled and
  installed manually.
- Playing Mopidy does not stop Spotify playing, and vice versa; the two sources are just mixed.
- Sound volume levels for the differing sources are not consistent, e.g. Mopidy at 50% volume sounds
  louder than Spotify at 50% volume.
- Multi‑room currently only works at CD quality (44.1 kHz 16 bit depth); in fact no testing has been 
  done at higher qualities for any part of **Aardsound**.
  You could force a higher quality by explicitly setting the appropriate role variables but this is
  untested and will probably result in white noise or silence.
- If you have a multi‑room server with no active `snapclients` but active **Spotify Connect** and/or
  **Mopidy** clients it is possible that the **Snapcast** server FIFO(s) will fill the entirety of
  the in‑memory `/tmp` filesystem.
  If this happens, a `reboot` of the affected server is likely to be needed (and an **Ansible**
  ac‑hoc `ansible ‑bm reboot` command is likely to fail because of the shortage of space on /tmp).
  For this reason, it is recommended *not* to use a 512GiB RAM Raspberry Pi (3A+ or Zero 2W) as
  a production **Snapcast** server.

## Untested
- HDMI outputs are explicitly excluded from use with Aardsound; it should be possible force the
  selection of an HDMI device by explicitly setting the variable `aardsound_card` to match the ALSA
  card name or number (as reported by the command `aplay ‑l`) but this is untested.
  If HDMI hot‑plugging is enabled (see
  [aardsound_hdmi_force_hotplug](./roles/aardsound/README.md#aardsound_hdmi_force_hotplug--true--false)
),
  this should be possible even if the output HDMI device is disconnected. 
  With HDMI hotplugging, the HDMI card(s) should be the lowest numbered (`0` or `0` and `1`,
  depending on the Raspberry Pi model) reported by `aplay ‑l`.
- 3.5mm jack (headphone) outputs are explicitly excluded from use with **Aardsound**; it should be
  possible to force the selection of the headphone device by explicitly setting the variable
  `aardsound_card` to match the ALSA card name or number (as reported by the command `aplay ‑l`) but
  this is untested.
  The headphone "sound card" should be the next device after the HDMI ALSA card(s), once the latter
  are stabilised, i.e. `0`, `1` or `2` depending on the number of HDMIs connected, or present if
  HDMI hotplugging is enabled.
- All recent testing has been done with 64‑bit versions of RasPiOS/Debian Trixie, but the previous
  Bookworm version should work, as should 32‑bit versions.
- Ubuntu or other Debian derivatives are not supported but *may* work if they have repositories that
  provide the required Mopidy and Snapcast packages (a non‑multiroom Spotify‑only installation does
  not need them).

## Rebuilding your Aardsound Device

Aardsound is designed so that the entire configuration is held in Ansible (this is a concept
called Infrastructure as Code).
This means you can always redeploy your Raspberry Pis.
There are two approaches you can use.

### Re‑running the Ansible Playbook

If you re‑run the Ansible playbook, it will make no changes to your Rapberry Pi if you have not
changed the configuration in your Ansible Inventory (see below).
This is a concept called *idempotency*.
If you change the Ansinle inventory variables defining your audio configuration, the configuration
will change accordingly; specifically `systemd` services will be added, changed or removed.

Note that, installed software remains installed even if you have no further use for it.
For example, if you configure **Snapcast** (either the server or the client), the `snapcast` Debian
package will will be installed.
If you later remove Snapcast from the configuration, the software will not be removed.

There are a few other configuration options that are not reversible, such as enabling HDMI 
hotplugging.

### Replacing the Micro SD card

Provided that you haven't made manual changes to the server, you can create a new RasPiOS SD
card, insert it in place of the old card and re‑run the `ansible‑playbook aardsound.yml` command.
You may want the `‑‑limit` (or `‑l`) option to restrict the command to work against one device
at a time.

This approach is strongly recommended when upgrading to a new major version of RasPiOS.

## Ansible Inventory
There are many ways to build an [Ansible inventory](
https://docs.ansible.com/projects/ansible/latest/inventory_guide/intro_inventory.html).
If you already use Ansible, then add the **Aardsound** hosts in an `aardsound` group to your current
inventory.
If you are new to Ansible, then the inventory approach described here is a simple one suitable for
**Aardsound**.

### A basic YAML inventory
Although Ansible supports many inventory formats, we will use one of the built‑in ones: YAML.
This should be saved in the file `/etc/ansible/hosts`, although if you already have an Ansible
inventory that you don't want to or can't modify, you can save the inventory somewhere else and
pass the path to the file to the `ansible` or `ansible‑playbook` command with the `‑i` or
`‑‑inventory` option.

A very simple inventory, for one host that supports Spotify Connect, might be:
```YAML
aardsound:
  hosts:
    wallace:
  vars:
    aardsound_spotify: true
```
This corresponds to the setup shown in Figure 4 of the [Example Setup README](./Examples.md).

A more complex example might be this, which corresponds to the setup shown in Figure 15 of
the [Example Setup README](./Examples.md).
This defines one multroom server (`wallace`) supporting Spotify and Mopidy, with no output
audio device, and a group of two clients (`gromit` and `feathers`) that can run Spotify and
Mopidy locally and also play either of the two multi‑room streams supplied by `wallace`.
```YAML
aardsound:
  hosts:
    wallace:
      vars:
        aardsound_mopidy_multiroom: true
        aardsound_spotify_multiroom: true
  children:
    aardclient:
aardclient:
  hosts:
    gromit:
      vars:
        aardsound_location: Lounge
    feathers:
      vars:
        aardsound_location: Kitchen
  vars:
    aardsound_mopidy: true
    aardsound_spotify: true
    aardsound_snapclient: true
    snapclient_of:
    - name: Wallace Mopidy
      host: wallace
      port: 11704
    - name: Wallace Spotify
      host: wallace
```

## Patching your servers
Assuming that you have set up the inventory as described above, you can patch all of the servers
using the commands
```
ansible -bm apt -a "name='*' state=latest" aardsound
ansible -bm reboot aardsound
```
If you don't have passwordless `sudo`, include `‑K` in the options for each.
If you want to patch one server at a time, replace `aardsound` with the hostname of the server.

Alternatively, you can create a simple **Ansible** playbook to manage this:
```YAML
# patch_aardsound.yml
---
- name: Patch and reboot aardsound servers
  hosts: aardsound
  gather_facts: false
  tasks:
  - name: Patch
    notify: Reboot
    apt:
      name: '*'
      state: latest
  handlers:
  - name: Reboot
    reboot:
```
This has the advantage that the reboot will not occur if no updates were applied.
To restrict this to a particular server or servers, use the `‑‑limit` (or `‑l`) option:
```BASH
steve@linux:~ $ ansible-playbook -l wallace,gromit reboot_aardsound.yml
```

## Multiple Subnets
The above assumes that your Raspberry Pi's, and any clients (**Spotify Connect**, **Mopidy MPD**
and/or **Mopidy HTTP**) share a single subnet.
**Spotify Connect**, in particular, relies upon ZeroConf, which is normally restricted to a single
subnet.

If you want your devices on multiple subnets (or have your **Spotify Connect** and **Mopidy** clients
on a different subnet from your Raspberry Pis), you will need to:
- Set the `aardsound_spotify_port` and/or  `aardsound_spotify_multiroom_port` variables to define
  the static port(s) that `librespot` will listen on and advertise over ZeroConf.
- Set up firewall rules between your subnets to allow **Spotify Connect**, **Mopidy MPD** and/or
  **Mopidy HTTP** traffic.
- Setup mDNS forwarding rules for ZeroConf to allow **Spotfiy Connect** clients to find the
  **Spotify** service (see, for example,
https://www.declarativesystems.com/2026/01/31/homeassistant-mdns-forwarding.html)

See the [`aardsound` role README](roles/aardsound/README.md#Player-specific-variables) for the
variables that define which ports are used and their default values.

## Ansible Tags

Ansible tags do not need to be used to run **Aardsound**.
However, they can speed the execution time for the playbook when, e.g., performing reconfigurations
particularly when testing.

If you want to consider using tags, see [Ansible Tags for Aardsound](./tags.md).  

## Aardsound Role Structure

Aardsound consists of the `aardsound.yml`
[Ansible playbook](https://docs.ansible.com/projects/ansible/latest/playbook_guide/index.html),
which just calls the `aardsound` role for the 
[Ansible inventory](https://docs.ansible.com/projects/ansible/latest/inventory_guide/index.html)
group `aardsound` and eleven
[Ansible roles](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_reuse_roles.html),
including the main `aardsound` role.
```BASH
.
└── roles
    ├── aardsound
    ├── alsa_scontrols
    ├── dmixer
    ├── mopidy
    ├── mopidy_installer
    ├── mopidy_multiroom
    ├── raspotify_installer
    ├── snapclient
    ├── snapserver
    ├── spotify
    └── spotify_multiroom
```
- `aardsound` &ndash; the main role which calls all of the other roles
- `alsa_scontrols` &ndash; used to set any ALSA controls required, e.g., to boost the playback volume
  for a specific device (the available controls will be specific to the sound card in use)
- `dmixer` &ndash; creates an ALSA `dmixer` in `/etc/asound.conf`; `dmixer` allows multiple input
  streams to connect the same soundcard simultaneously
- `mopidy` &ndash; configures (or removes) systemd services for Mopidy
- `mopidy_installer` &ndash; installs Mopidy and extensions
- `mopidy_multiroom` &ndash; configures Mopidy as a source for Snapserver; calls `mopidy`
  and `snapserver`
- `snapclient` &ndash; configures one or more instances of the Snapcast client to connect to
   Snapcast servers
- `snapserver` &ndash; configures an instance of Snapserver
- `raspotify_installer` &ndash; installs Raspotify (and thus librespot)
- `spotify` &ndash; configures (or removes) Raspotify‑style systemd services for Spotify
- `spotify_multiroom` &ndash; configures librespot as a source for Snapserver; calls `spotify` and
  `snapserver`

The hierarchy of these roles is:
```
                                           ┌───────────┐
                                           │ aardsound │
                                           └─────┬─────┘
┌──────────────────────────┬───────────┬─────────┴──────────┬────────────┬──────┐
│                          │           │                    │            │      │
│  ┌─────────────────────┐ │ ┌─────────┴─────────┐ ┌────────┴─────────┐  │  ┌───┴────┐
├──┤ raspotify_installer │ │ │ spotify_multiroom │ │ mopidy_multiroom │  │  │ dmixer │
│  └─────────────────────┘ │ └─────┬───────┬─────┘ └──────┬─────┬─────┘  │  └────────┘
│                          │       │       │              │     │        │
│  ┌──────────────────┐    │       │       │              │     │        │
├──┤ mopidy_installer │    └───────┤       └──────┬───────┘     ├────────┴───┐
│  └──────────────────┘            │              │             │            │
│                                  │              │             │            │
│  ┌────────────────┐         ┌────┴────┐  ┌──────┴─────┐  ┌────┴───┐  ┌─────┴──────┐
└──┤ alsa_scontrols │         │ spotify │  │ snapserver │  │ mopidy │  │ snapclient │
   └────────────────┘         └─────────┘  └────────────┘  └────────┘  └────────────┘
```
In other words, `aardsound` calls all of the other roles directly, except `snapserver`, which is 
called by `spotify_multiroom` and `mopidy_multiroom`, which also call `spotify` and `mopidy`
respectively.

### Role Variables

In keeping with good Ansible practice, Ansible variables used by **Aardsound** are prefixed with the
role name and an underscore.
Most important variables for **Aardsound** are `aardsound_` role variables with some notable
exceptions such as `snapclient_of_` (see the [Example Configurations](./Examples.md) README for how
this is used).

Each role has a set of default values for its variables that are reasonable for that role *when
it is running independently of the role that is calling it*.
For example, the defaults for the `spotify` role are intended for the setup of a custom instance
of Raspotify/librespot but are not specific to the requirements of either the `aardsound` or the
`spotify_multiroom` roles.
Both of those roles have their own defaults, similarly the defaults for `spotify_mutliroom` are
not specific to `aardsound`.

When `aardsound` invokes `spotify` or `spotify_multiroom` or when `spotify_multiroom` invokes
`spotify` the `include_role` or `import_role` task has a `vars` statement that applies the
values of variables in the calling role to equivalent variables in the called role, like this
```YAML
- name: Import the bar role into foo
  vars:
    bar_variable: "{{ foo_variable }}"
  import_role:
    name: bar
```
[Ansible variable precedence](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_variables.html#understanding-variable-precedence)
plays a significant role here.
The default values for both `foo_variable` and `bar_variable`are very low precedence: setting the
same variable anywhere else will change the value.
However the `vars:` statement on the `import_role` task is a high precedence variable assignment.
In effect, it promotes the precedence of the assignment of `bar_variable` to equal `foo_variable`
higher than many other possible assignments, including, signifiantly, all flavours of inventory
variables.

This means that setting `bar_variable` in the Ansible inventory has no effect: the default
value of `foo_variable` overrides it: `foo_variable` must be set in the inventory instead.

This is as intended, but it means that it is necessary to check that a role variable does not
have a matching variable in the `aardsound` role before setting it.
The mapping of `aardsound` role variables to other roles is described in the
[`aardsound` role README](./roles/aardsound/README.md).

Ths same situation applies to default variables in the `spotify_multiroom` role overrriding
variables in the `spotify` role and to default variables in the `mopidy_multiroom` role
overrriding variables in the `mopidy` role.
The variable to which this applies are described in the
[`spotify_multiroom` role README](./roles/spotify_multiroom/README.md) and the
[`mopidy_multiroom` role README](./roles/mopidy_multiroom/README.md).

See the following:
- **Aardsound** configuration variables
  - [`aardsound`](./roles/aardsound/README.md) role README
- Package installation variables
  - [`raspotify_installer`](./roles/raspotify_installer/README.md) role README
  - [`mopidy_installer`](./roles/mopidy_installer/README.md) role README
- **Spotify** audio source configuration variables
  - [`spotify`](./roles/spotify/README.md) role README
  - [`spotify_multiroom`](./roles/spotify_multiroom/README.md) role README
- **Mopidy** audio source configuration variables
  - [`mopidy`](./roles/mopidy/README.md) role README
  - [`mopidy_multiroom`](./roles/mopidy_multiroom/README.md) role README
- **Snapcast**  configuration variables
  - [`snapclient`](./roles/snapclient/README.md) role README
  - [`snapserver`](./roles/snapserver/README.md) role README
- **ALSA** configuration variables
  - [`alsa_scontrols`](./roles/alsa_scontrols/README.md) role README
  - [`dmixer`](./roles/dmixer/README.md) role README

# License
See [License.md](./License.md)
