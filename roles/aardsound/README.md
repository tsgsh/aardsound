# `aardsound` role  (part of Aardsound)

For a fuller description of the operation of the Aardsound project, see the
[Aardsound README](../../README.md).

The `aardsound` role is called by the `aardsound.yml` playbook.
The paths of the two are related as follows
```
.
└── aardsound               # Github directory
    ├── aardsound.yml       # aardsound playbook
    └── roles
        └── aardsound       # aardsound role (this role)
```
The `aardsound.yml` playbook executes the `aardsound` role against all hosts in the `aardsound`
inventory group with `become` = `true`.

## Activities

This role controls the installation of software for creating a (multi-room) audio player on one
or more Raspberry Pis and the configuration of that software.

The role performs the following "activities" &ndash; groups of Ansible tasks.

1.  Confirm that the target hosts and their variables are valid
2.  Make cchanges to the Raspberry Pi boot configuration if required
3.  Patch using APT
4.  Install APT and/or PIP packages
5.  Determine the default ALSA output device, if not specified by the host's Ansible variables
6.  Set ALSA simple controls for the device, if required
7.  Create an ALSA direct mixer (`dmix`) device if there are multiple audio inputs sharing an
    output device
8.  Install (or remove) some of the following services as declared by the host's Ansible Variables
    - Spotify [Connect] &ndash; single- and/or multi-room
    - Mopidy (with extensions) &ndash; single- and/or multi-room
    - Bluettooth A2DP single- or multi-room source
    - Snapcast (for multi-room operation)

Activities can be controlled by Ansible tags, see the [Aardsound README](../../README.md).

## Variables

### Variable that define which Components are in use

#### aardsound_spotify = *boolean*
Run a single-room Spotify instance

Sets: `spotify_active`

Default: `false`

#### aardsound_mopidy = *boolean*
Run a single-room Mopidy instance

Sets: `mopidy_active`

Default: `false`

#### aardsound_bluetooth = *boolean*
Run a multi-room Bluetooth speaker instance

Sets: `bluetooth_active`

Default: `false`

#### aardsound_spotify_multiroom = *boolean*
Run a multi-room Spotify instance

Sets: `spotify_multiroom_active`

Default: `false`

#### aardsound_mopidy_multiroom = *boolean*
Run a multi-room Mopidy instance

Sets: `mopidy_multiroom_active`

Default: `false`

#### aardsound_bluetooth_multiroom = *boolean*
Run a multi-room Bluetooth speaker instance

Sets: `bluetooth_multiroom_active`

Default: `false`

#### aardsound_snapclient = *boolean*
Run one or more instances of the Snapcast client to connect to multi-room instances of Spotify
and/or Mopidy.

Default: `false`

### Initial Setup Variables

#### aardsound_allow_all_inventory = *boolean*
The `aardsound` role will not operate against the whole `aardsound` group unless the variable
`aardsound_allow_all_inventory` is `true`; if it is `false` (the default) then the playbook
has to be called with a value of `--limit` (or `-l`) that constrains the number of hosts to
a subset of the group (which may be the entire group).
This is intended to prevent the accidental deployment of an untested or invalid configuration to a
whole housefull of Raspberry Pi audio systems.

If you don't want this behaviour, set `aardsound_allow_all_inventory: true` as an inventory variable
for the `aardsound group`

Default: `false`

#### aardsound_upgrade = true | false
Whether to perform an APT upgrade

Default: `true`

#### aardsound_aardsound_reboot = *boolean*
Reboot if the APT update changed the installation?

Default: `false`

#### aardsound_aardsound_reboot_prompt = *boolean*
Prompt before performing a reboot?

Default: `false`

### Raspberry Pi-specific Initial Setup Variables
These are not relevant if the host is not a Raspbery Pi.

#### aardsound_hdmi_force_hotplug = true | false
Force HDMI hotplug in `/boot/firmware/config.txt`

This is important for ALSA card numbers if the Raspberry Pi sometimes (but not always) has an HDMI
connection because the ALSA device number will not be stable if this is not set.

It's safe to leave it as `false` if your Raspberry Pi is always headless, or always has the same
number of monitors attached or if you do not refer to ALSA cards by their index numbers.

Default: false

#### aardsound_disable_onboard_bt = true | false
Disable the onboard Bluetooth adapter in  `/boot/firmware/config.txt`

Default: false

## Initial Setup Variables for the Mopidy installation

#### aardsound_mopidy_use_venv
Should Mopidy be installed in a Python virtual_environment?

Sets:
- `mopidy_installer_use_venv`
- `mopidy_use_venv`
- `mopidy_multiroom_use_venv`

Default: `true` unless `mopidy_installer_extensions` is an empty list, RAM is less than or equal to
512MiB and the `systemd` default target is 'graphical'.

#### aardsound_mopidy_venv
The location for the Python virtual environment where Mopiy and its extensions will be installed.

Sets:
- `mopidy_installer_venv`
- `mopidy_venv`
- `mopidy_multiroom_venv`

Default: `'/usr/local/mopidy/venv'`

#### aardsound_nfs_media
Location of an external NFS volume that will contain the media to be accessed by `mopidy_local`.

Default: `none` (no NFS media)

#### aardsound_local_media
The mount point to use for the external NFS meda

Default: `none` if there is no NFS media, otherwise /mnt/mopidy

### General Aardsound Options
These apply to multiple services, although not all of them are relevant to all of them.

#### aardsound_location = *string*
User-friendly string where the device is located, e.g., 'Kitchen'.
This is used by the `spotify` and `mopidy` roles to advertise this device.

Default: `none`

Sets:
- `spotify_location`
- `mopidy_location`

#### aardsound_multiroom_name = *string*
User-friendly to identify the name of the Multiroom service.
This is used by the mukti-room invocations of `spotify` and `mopidy` roles to advertise the
multi-room service.

Default: `Multiroom`

Sets:
- `spotify_multiroom_name`
- `mopidy_multiroom_name`

#### aardsound_output_device = *string*
An ALSA PCM device to use for the ultimate sound output.
If not specified, the device is based on the `aardsound_plugin`, `aardsound_card` and
`aardsound_device` values, i.e: *`plugin`*:*`card`*,*`device`*

Sets:
- If `dmix` is in use:
  - `dmixer_output_device`
- If `dmix` is not in use:
  - `spotify_output_device`
  - `mopidy_output_device`
  - `snapclient_output_device`

#### aardsound_plugin = plughw | hw | dmix | dsnoop
The ALSA PCM plugin to use for the device.

Ignored if `aardsound_output_device` is specified

Default: `'hw'` if `aardsound_dmixer` is `true` or `aardsound_mixer` is `false`; otherwise `'plughw'`

#### aardsound_card = *string*|*integer*
ALSA card name or index number as reported by `aplay -l`.
Default is the first card that is not HDMI nor Headphones nor Loopback.

If `aardsound_output_device` is specified, this is only used to set the default for
`aardsound_mixer_card`.

Sets:
- `alsa_scontrols_card`
- `dmixer_card`

Default: the name of the first card that is not Loopback, HDMI or Headphones

#### aardsound_device = *index_number*
If the selected or default ALSA card has more than one device then specify a number if a device
other than than the first device on the card as reported by `aplay -L`.

If `aardsound_output_device` is specified, this is only used to set the default for
`aardsound_mixer_device`, which, in turn. is only used by librespot (Spotify).

Default: the first device on `aardsound_card` (usually 0).

#### aardsound_volume_control = linear | log | cubic | fixed
Specify the form of the volume control for Spotify and Mopidy

Sets:
- `spotify_volume_control`
- `mopidy_volume_control`

Default: `'cubic'`

#### aardsound_initial_volume = *1-100*
Set the initial playback volume

Sets:
- `spotify_initial_volume`
- `spotify_multiroom_initial_volume`
- `mopidy_initial_volume`
- `mopidy_multiroom_initial_volume`
- `bluetooth_initial_volume`
- `bluetooth_multiroom_initial_volume`

Default: 50

#### aardsound_mixer = *boolean*
Route sound via ALSA mixer or are volume, balance, etc., controlled solely by the source
(softmixer).

Sets:
- `spotify_mixer`
- `mopidy_mixer`
- `snapclient_mixer`

Default: `false`, untested with `true`

#### aardsound_mixer_plugin = *string*
The plugin to use for ALSA amixer.

Ignored if `aardsound_mixer` is `false`

Sets: `spotify_mixer_plugin`

Default: hw if `aardsound_dmixer` is `true` or `aardsound_mixer` is `false`; otherwise plughw

#### aardsound_mixer_card = *string*|*integer*
The sound card to use for ALSA amixer.

Sets:
- `spotify_mixer_card`
- `mopidy_mixer_card`
- `snapclient_mixer_card`

Default: `aardsound_card` if this is explicitly specified, or `aardsound_output_device` is not
explicilty specified, otherwise the card defined by `aardsound_output_device`.

#### aardsound_mixer_device = *integer*
The index of the mixer device on `aardsound_mixer_card`.
Only used by librespot (Spotify)

Untested, may be incorrect

Sets: `spotify_mixer_device`

Default: `aardsound_device` if this is explicitly specified, or `aardsound_output_device` is not
explicilty specified, otherwise the device defined by `aardsound_output_device`.

#### aardsound_mixer_control = *string*
The name of an ALSA mixer control supported by `aardsound_mixer_card`.

Default: the first control with either the 'volume' or the 'pvolume' capability as reported by
running `amixer` without a command against the ALSA mixer card.


Sets:
- `spotify_mixer_control`
- `mopidy_mixer_control`

#### aardsound_dmixer = *boolean*
Install a plugin and a PCM in `/etc/asound.conf` that uses the `dmix` plugin to allow multiple sources
to share one output nicely.

Sets: `dmixer_active`

Default: `true` if more than one audio source is defined on the host, otherwise `false`

#### aardsound_dmixer_plugin = *string*
Name of the ALSA dmix plugin

Sets: `dmixer_plugin`

Default: `'aardmixer'`

#### aardsound_dmix_device = *string*
Name used for the ALSA dmix PCM and Control devices

Sets: `dmixer_dmix_device`

Default: `'aarddmix'`

#### aardsound_format = F64 | F32 | S32 | S24 | S24_3 | S16
Sets the format for Spotify, FFMPEG (used for multi-room Spotify) and multi-room Mopidy.

***Experimental***

If the format is *Xnn* or *xnn* (where *X* is 'F' or 'S' and *x* is 'f' or 's', and *nn* is 64, 32,
etc.) then:
- The Spotify format is set to '*Xnn*'
- The FFMPEG format is set to '*xnn*le'
- The FFMPEG codec is set to 'pcm_*xnn*le
- The Mopidy `audiocoonvert` format (used by multi-room Mopidy) set to '*XXX*LE'

Sets:
- `spotify_format`
- `spotify_multiroom_format`
- `mopidy_multiroom_format`

Default: `'S16'`

#### aardsound_rate = *integer*
Sample frequency in Hz.

***Experimental***

Sets:
- `spotify_multiroom_rate`
- `mopidy_multiroom_rate`
- `snapclient_rate`
- `dmixer_rate`

Default: `44100`, untested with anything else

#### aardsound_depth = *integer*
Audio bit depth, used by Snapcast

Sets:
- `spotify_mutiroom_depth`
- `mopidy_mutiroom_depth`
- `snapclient_depth`

Default: `16`, untested with anything else

#### aardsound_channels = *integer*
The number of audio channels.

Sets:
- `spotify_mutiroom_channels`
- `mopidy_mutiroom_channels`
- `snapclient_channels`
- `dmixer_channels`

 Default: `2`, untested with anything else

### Player-specific variables

#### aardsound_spotify_address = *comma-separated string of IP addresses*
Which IP address the `librespot` daemon should bind to

Sets: spotify_interface

Default: spotify_role default

#### aardsound_spotify_port = *1025-65535*
Which port the `librespot` daemon should bind to

Sets: spotify_port

Default: spotify_role_default (random high port)

#### aardsound_multiroom_spotify_interface = *comma-separated string of IP addresses*
Which IP address the multi-room `librespot` daemon should bind to

Sets: spotify_multiroom_interface

Default: spotify_multiroom_role default

#### aardsound_spotify_multiroom_port = *1025-65535*
Which port the multi-room `librespot` daemon should bind to

Sets: spotify_multiroom_port

Default: spotify_multiroom_role default (random high port)

#### aardsound_mopidy_http_address = 127.0.0.1|::1|0.0.0.0|::
Which address the HTTP server should bind to

Sets: mopidy_http_bind_address

Default: mopidy_role default

#### aardsound_mopidy_http_port = *1025-65535*
Which port the HTTP server should bind to

Sets: mopidy_http_bind_port

Default: 6680

#### aardsound_mopidy_mpd_address = 127.0.0.1|::1|0.0.0.0|::
Which address the MPD server should bind to

Sets: mopidy_mpd_bind_address

Default: mopidy_role default

#### aardsound_mopidy_mpd_port = *1025-65535*
Which port the MPD server should bind to

Sets: mopidy_mpd_bind_port

Default: 6600

#### aardsound_multiroom_mopidy_http_address = 127.0.0.1|::1|0.0.0.0|::
Which address the multi-room HTTP server should bind to

Sets: mopidy_multiroom_http_bind_address

Default: mopidy_multiroom role default

#### aardsound_multiroom_mopidy_http_port = *1025-65535*
Which port the multi-room HTTP server should bind to

Sets: mopidy_multiroom_http_bind_port

Default: 6681

#### aardsound_multiroom_mopidy_mpd_address = 127.0.0.1|::1|0.0.0.0|::
Which address the multi-room MPD server should bind to

Sets: mopidy_multiroom_mpd_bind_address

Default: mopidy_multiroom role default

#### aardsound_multiroom_mopidy_mpd_port = *1025-65535*
Which port the multi-room MPD server should bind to

Sets: mopidy_multiroom_mpd_bind_port

Default: 6601

#### aardsound_mopidy_extensions = *dictionary of dictionaries*
This is a front-end to `mopdidy_extensions` that also ensures that the Mopidy extension is installed
by `mopidy_installer`.

Sets:
- `mopidy_installer_packages`
- `mopidy_installer_sudoers`
- `mopidy_installer_install`
- `mopidy_extensions`

Each key of the top-level dictionary is the name of an extension and a section in the Mopidy
configuration file.
The value for each key is itself a dictionary and each key/value pair in that defines either:

- an entry in that section of the Mopidy configuration file (*`key`*` = `*`value`*), or
- entries in a list variable to be passed to the `mopidy_installer` role; there are three options
  - `_packages` defines one or more entries to be added to `mopidy_installer_packages`; if it is
    undefined then the default is the the name of the extension, prefixed with `mopidy-`.
    Note that the `mopidy-` prefix cannot be omitted for any packages listed, it's just assumed for
    the default case.

    Packages are installed with `pip` unless `mopidy_installer_pip` is set to `false`, in which case
    they are installed with `apt`
  - `_sudoers` defines [sudoers] entries needed for that extension.
    Each entry is added to the file `/etc/sudoers.d/50-mopidy` as
    ```
    %mopidy ALL=NOPASSWD: {{ item }}
    ```
    where `{{ item }}` is the list entry.
  - `_install` defines shell commands that need to be done after the packages are installed:
    each entry is a dictionary with one key/value pair, where the key gives the name of the user
    to run the command (e.g. `mopidy` or `root`) and the value is the command to run.

    For example:
    ```
    - mopidy: unprivileged-command
    - root: privileged-command
    ```
    Commands are run as Bash shell commands (preceded by `set -o pipefail` for safety).
    Ensure that privileged commands can be run without a password prompt if they are not run by root:
    if they are run under the `mopidy` user, that can be achieved with a `_sudoers` entry.

The `_packages` keys from each entry in the top level dictionary are consolidated into a list that is
used to set the `mopidy_installer_packages` variable.
The same is done for `_sudoers` and `_install` to set `mopidy_installer_sudoers` and
`mopidy_installer_install` respectively.

The top-level dictionary, with the `_packages`, `_sudoers` and `_install` keys removed from each
entry is used to set `mopidy_extensions`

As a convenience, the location where the Python Virtual Environment is installed is set in
the variable `_`; this means that information about the installation environment is not needed
in the Ansible inventory.
`{{ _ }}` can be used to replace the path of the virtual environment in the `_sudoers` and/or
`_install` keys: the string containing it
[must be encased in single or double quotes](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_variables.html#id21).

`_` is not useful if `mopidy_installer_pip` is set to `false`.

Default: {}

As an example, to install the [Iris](https://mopidy.com/ext/iris/) extension, the following
could be used:
```
aardsound_mopidy_extensions:
  iris:
    _sudoers:
    - "{{ _ }}mopidy_iris/system.sh"
    country: UK
    locale: en_GB
    snapcast_enabled: false
```
- `_packages` &ndash; omitted, installs the PyPi version of
  [`mopidy-iris`](https://pypi.org/project/Mopidy-Iris/)
- `_sudoers` &ndash; `mopidy-iris` requires the `install.sh` to be run with `sudo`:
  see https://github.com/jaedb/Iris/wiki/Getting-started#installing
- `country: UK` &ndash; the default country code for `mopidy-iris` is `NZ`
- `locale: en_GB` &ndash; the default country code for `mopidy-iris` is `en_NZ`
- `snapcast_enabled: false` &ndash; the Snapcast support provided by Iris is not required because
  it is provided by Aardsound

The resulting configuration file section is:
```
[iris]
country = UK
local = en_GB
snapcast_enabled = false

```
**Note**:
at the time of writing, the PyPi version of [Mopidy-Iris](https://pypi.org/project/Mopidy-Iris/) (version
3.70.0) doesn't support Mopidy version&nbsp;4.
The `develop` branch could be downloaded from Github with the appropriate PIP formulation of the URL, i.e.,
```
_packages:
    - 'git+https://github.com/jaedb/Iris.git@develop'
```
However, this only installs the python code, it does not do the required compilation.

So, until the current development branch is released to PyPi, **Mopidy-Iris** doesn't work, without
a manual installation.


## License

See [License.md](../../License.md)
