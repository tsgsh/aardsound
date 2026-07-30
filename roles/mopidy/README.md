# `mopidy` role (part of Aardsound)

For a fuller description of the operation of the Aardsound project, see the
[Aardsound README](../../README.md).

The `mopidy` role is either
- *Included* by the `aardsound` role; or
- *Imported* by the `mopidy_multiroom` role, which is in turn *included* by the `aardsound`
  role.

The `aardsound` role, is imported by the `aardsound.yml` playbook.
The paths of these are related as follows:
```
.
└── aardsound                   # Github directory
    ├── aardsound.yml           # aardsound playbook
    └── roles
        ├── aardsound           # aardsound role
        ├── mopidy              # mopidy role (this role)
        └── mopidy_multiroom    # mopidy_multiroom role
```
The use of `include_role` is important for the sequencing of handler definitions: see the
[**Handlers**](#Handlers) section below.
 
This role installs and configures (or removes) an instance of [Mopidy](https://mopidy.com/) and
associated extensions.

## Variables

This section describes the variables defined in the file `./defaults/main.yml`, that are intended to
be modified as required by setting values in the Ansible inventory to suit the required Aardsound
configuration.
Internal variables defined in `./vars/main.yml` have a higher variable precedence than
inventory variables and should not be modified (e.g. by setting extra variables or role parameters).

#### mopidy_active = false|true
Should the Mopidy services be installed and configured or removed

Default: `true`

#### mopidy_service = *string*
The name of the `systemd` service running mpidy.
Also sets the name of the user that owns the service.

Default: mopidy

#### mopidy_location = *string*
The place where the audio output can be heard, e.g. 'Kitchen'.

Used to name the Mopidy service in ZeroConf, etc..
If not defined then the capitalised hostname of the server is used as a name.

Default: `none`

#### mopidy_use_venv = false|true
Should Mopidy be installed in a Python virtual environment?

This should be set to `true` if there are mopidy extensions defined that are not available as APT
packages
(see [mopidy_extensions](#mopidy_extensions--dictionary) below).

Default: false

#### mopidy_venv = *path*
The path where the Mopidy virtual environment should be created

Default: '/usr/local/mopidy/venv'

#### mopidy_output_device = *string*
The ALSA PCM to which Mopidy sends its audio streams

Default: default

#### mopidy_mixer = *boolean*
Use an ALSA hardware mixer instead of an internal software mixer?

Default: `false`

#### mopidy_mixer_control = *string*
ALSA hardware mixer control as reported by the sound card.

Ignored if mopidy_mixer = false.

Default: Master

#### mopidy_mixer_card = *string* | *integer*
The sound card to use for the hardware mixer

Ignored if mopidy_mixer = false.

Default: `none`

#### mopidy_volume_ctrl = cubic | fixed | linear | log
Mopidy volume control scale type

Default: cubic

#### mopidy_initial_volume = 0-100
Mopidy initial volume as a percentage from 0 to 100

Default: 50

#### mopidy_verbosity = -1 to 4
The verbosity level for Mopidy

Default: 0

#### mopidy_media_library = *path*
The media library for `mopidy-local`

If this is not `/var/lib/mopidy/media` then the specified directory is symbolically linked to there
with a `systemd` mount unit.

Default: /var/lib/mopidy/media

#### mopidy_local_scan = *boolean*
Should a `systemd` "oneshot" service to run `mopidy local scan` be defined?

Default: `true`

#### mopidy_local_scan_limit = *integer*
Value of the `--limit` option on the number of files to be scanned by `mopidy local scan`

Default: 0 (no limit)

#### mopidy_local_scan_time = hh:mm | OnCalendar_specification
Time in hours and minutes each day when the `mopidy_local_scan` will run.

Can also be any supported format for the `OnCalendar` statement in a `systemd` timer unit file.

Default: 04:00

#### mopidy_trusted_network = *boolean*
Should Mopidy services be bound to network addresses (or only to loopback addresses)?

May be overriden by
[`mopidy_http_bind_address`](#mopidy_http_bind_address--ip-address)
or
[`mopidy_mpd_bind_address`](#mopidy_mpd_bind_address--ip=address)

Default: `false`

#### mopidy_ipv6 = *boolean*
Should IPv6 addresses be used for Mopidy?

May be overriden by
[`mopidy_http_bind_address`](#mopidy_http_bind_address--ip-address)
or
[`mopidy_mpd_bind_address`](#mopidy_mpd_bind_address--ip=address)

Default: false

#### mopidy_http = *boolean*
Should the `mopidy-http` extension be used?

Default: `true`

#### mopidy_http_bind_address = *IP address*
Address to bind the `mopidy-http` extension to

Default:
- :: if `mopidy_trusted_network` and `mopidy_ipv6` are true
- 0.0.0.0 if `mopidy_trusted_network` is true and `mopidy_ipv6` is false
- ::1 if `mopidy_trusted_network` is false and `mopidy_ipv6` is true
- 127.0.0.1 if `mopidy_trusted_network` and `mopidy_ipv6` are false

#### mopidy_http_port = 1025-65535
Port to bind the `mopidy-http` extension to

Default: 6680

#### mopidy_mpd = *boolean*
Should the `mopidy-mpd` extension be used?

Default: `true`

#### mopidy_mpd_bind_address = *IP address*
Address to bind the `mopidy-mpd` extension to

Default: 
- '::' if `mopidy_trusted_network` and `mopidy_ipv6` are true
- '0.0.0.0' if `mopidy_trusted_network` is true and `mopidy_ipv6` is false
- '::1' if `mopidy_trusted_network` is false and `mopidy_ipv6` is true
- '127.0.0.1' if `mopidy_trusted_network` and `mopidy_ipv6` are false

#### mopidy_mpd_port = 1025-65535
Port to bind the `mopidy-mpd` extension to

Default: 6600

#### mopidy_mpd_password = *string*
Password to be used to access Mopidy MPD.
Note that this is not encrypted over the network and is highly insecure.  

Default: `none`

#### mopidy_extra_groups = *list*
A list of groups that the user running Mopidy (as defined by 
[`mopidy_service`](#mopidy_service--string)) should belong to, in addition to `mopidy` (the primary
group) and `audio`

Default: []

#### mopidy_start_after_services = *list*
A list of systemd services that you want to start before the Mopidy service.
Added to the `systemd` unit definition as an `After=` statement for each entry.
Each entry must be a valid `systemd` service of the creation of the Spotify systemd unit will fail.
Do not include the `.service` suffix.

Default: `[]`

#### mopidy_extensions = *dictionary of dictionaries*
This allows for the configuration of arbitrary Mopidy extensions.

Each key of the top-level dictionary is the name of a section in the Mopidy configuration file
and each value contains a dictionary of configuration settings for that section.
For example, this:
```
iris:
  country: UK
  locale: en_GB
  snapcast_enabled: false
```
causes the following to be added to the configuration file
```
[iris]
country = UK
locale = en_GB
snapcast_enabled = false

```
No validation checking of the keys and values is performed.

Default: `{}`


## Handlers

Handlers of a given name will be overrwritten by subsequent definitions and the dynamic definitions
of an include are always the last to be applied.
See [Ansible Handlers: running operations on change](
    https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_handlers.html#handlers-running-operations-on-change)

The including role is responsible for the flushing of handlers if there is a possibility of
including this role again and re-writing variables (such as service names) before the end of the
play, when handlers are normally flushed.

### Handlers in this role:
- Enable and (re)start mopidy service
- Start mopidy local scan service
- Enable mopidy local scan timer
- Reload systemd
- Mopidy network notification&dagger;

&dagger; Runs a debug task with message that the network used by Mopidy is either insecure or is the
loopback network, neither of which is ideal.
Addition of an `nginx` proxy to remove the need for this is on the "to do" list.

## License

See [License.md](../../License.md)
