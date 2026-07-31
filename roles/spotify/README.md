# `spotify` role (part of Aardsound)

For a fuller description of the operation of the Aardsound project, see the
[Aardsound README](../../README.md).

The `spotify` role is either
- *Included* by the `aardsound` role; or
- *Imported* by the `spotify_multiroom` role, which is in turn *included* by the `aardsound`
  role.

The `aardsound` role, is imported by the `aardsound.yml` playbook.
The paths of these are related as follows:
```
.
└── aardsound                   # Github directory
    ├── aardsound.yml           # aardsound playbook
    └── roles
        ├── aardsound           # aardsound role
        ├── spotify             # spotify role (this role)
        └── spotify_multiroom   # spotify_multiroom role
```
The use of `include_role` is important for the sequencing of handler definitions: see the
[**Handlers**](#Handlers) section below.
 
This role installs and configures (or removes) an instance of
[`librespot`](https://github.com/librespot-org/librespot)
as provided by the 
[Raspotify](https://dtcooper.github.io/raspotify/) package.

## Variables

This section describes the variables defined in the file `./defaults/main.yml`, that are intended to
be modified as required by setting values in the Ansible inventory to suit the required Aardsound
configuration.
Internal variables defined in `./vars/main.yml` have a higher variable precedence than
inventory variables and should not be modified (e.g. by setting extra variables or role parameters).

#### spotify_active = *boolean*
Should the `librespot` service be installed and configured or removed

Default: `true`

#### spotify_service = *string*
The name of the `systemd` service running Spotify.
Also sets the name of the user that owns the service.

Default: raspotify

#### spotify_location = *string*
The place where the audio output can be heard, e.g. 'Kitchen'

Used to name the Spotify service advertised via ZeroConf.
If not defined then the capitalised hostname of the server is used as a name.

Default: `none`

#### spotify_output_device = *string*
The ALSA PCM to which `librespot` sends its audio streams

Default: default

#### spotify_mixer = *boolean*
Use an ALSA hardware mixer?

Default: `false`

#### spotify_mixer_control = *string*
ALSA hardware mixer control as reported by the sound card

Default: Master; ignored if `spotify_mixer` = `false`

#### spotify_mixer_plugin = *string*
ALSA plugin to use for the mixer control

Default: hw; ignored if `spotify_mixer` = `false`

#### spotify_mixer_card = *string* | *integer* 
The sound card to use for the hardware mixer

Default: 0; ignored if `spotify_mixer` = `false`

#### spotify_mixer_device = *integer*
The device on the sound card to use for he hardware mixer

Default: 0; ignored if `spotify_mixer` = `false`

#### spotify_volume_control = cubic | fixed | linear | log
Spotify volume control scale type

Default: cubic

#### spotify_initial_volume = 0-100
Spotify initial volume as a percentage from 0 to 100

Default: 50

#### spotify_autoplay = *boolean*
Automatically play similar songs when your music ends.

Default: `true`

#### spotify_bitrate = 96 | 160 | 320
Spotify bit rate in kbps

Default: 320

#### spotify_format = F64 | F32 | S32 | S24 | S24_3 | S16
Spotify output format to ALSA.

Default: S24

#### spotify_volume_normalisation = *boolean*
Play all tracks at approximately the same apparent volume.

Default: `false`

#### spotify_normalisation_method = basic | dynamic
Specify the normalisation method to use:

Default: `none` (`librespot` internal default: dynamic)

#### spotify_normalisation_gain_type = track | album | auto
Specify the normalisation gain type to use

Default: `none` (`librespot` internal default: auto)

#### spotify_normalisation_pregain = *floar*
Pregain (dB) applied by the normalisation.

Default: `none` (`librespot` internal default: 0)

#### spotify_normalisation_threshold = *float*
Threshold (dBFS) to prevent clipping.

Default: `none` (`librespot` internal default: -2.0)

#### spotify_normalisation_attack = *integer*
Attack time (ms) in which the dynamic limiter is reducing gain.

Default: `none` (`librespot` internal default: 5)

#### spotify_normalisation_release = *integer*
Release or decay time (ms) in which the dynamic limiter is restoring gain

Default: `none` (`librespot` internal default: 100)

#### spotify_normalisation_knee = *float*
Knee steepness of the dynamic limiter

Default: `none` (`librespot` internal default: 1.0)

#### spotify_port = 1025-65535
The port the `librespot` daemon binds to and advertises over ZeroConf.

Default: `none` (`librespot` internal default is to bind to a random high port)

#### spotify_interface = *ipaddress* | &lsqb;*ipaddress*,&hellip;&rsqb; | *ipaddress*,*ipaddress*&hellip;
Interface IP address or a list of IP addresses or a comma-separated string of IP addresses to
which the `librespot` daemon will bind and advertise over ZeroConf.
Example: "192.168.0.10,10.0.0.10".

Default: `none`  (`librespot` internal default is to bind to all interfaces)

#### spotify_extra_groups = *list*
A list of groups that the user running Spotify (as defined by 
[`spotify_service`](#spotify_service--string)) should belong to, in addition to `raspotify` (the
primary group) and `audio`

Default: []

#### spotify_start_after_services = *list*
A list of systemd services that you want to start before the Spotify service.

Each entry is added to the `systemd` unit definition as an `After=` statement  and must be a valid
`systemd` service or the creation of the Spotify systemd unit will fail.
Do not include the `.service` suffix.

Default: []

#### spotify_verbosity = 0 | 1 | 2
Level of logging logging in `librespot`

0 corresponds to the `--quiet` option, ` to the default behaviour and 2 to the `--verbose` option

Logs are seent to the `systemd` journal

Default: 0


## Handlers

Handlers of a given name will be overrwritten by subsequent definitions and the dynamic definitions
of an include are always the last to be applied.
See [Ansible Handlers: running operations on change](
    https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_handlers.html#handlers-running-operations-on-change)

The including role is responsible for the flushing of handlers if there is a possibility of
including this role again and re-writing variables (such as service names) before the end of the
play, when handlers are normally flushed.

### Handlers in this role:
- Enable and (re)start librespot service
- Enable and (re)start spotify crash service
- Reload systemd


## License

See [License.md](../../License.md)
