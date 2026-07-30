# `spotify_multiroom` role (part of Aardsound)

For a fuller description of the operation of the Aardsound project, see the
[Aardsound README](../../README.md).

The `spotify_multiroom` role is *included* by the `aardsound` role and *imports* the 
`spotify` and `snapserver` roles.
The `aardsound` role, is imported by the `aardsound.yml` playbook.
The paths of these are related as follows:
```
.
└── aardsound                   # Github directory
    ├── aardsound.yml           # aardsound playbook
    └── roles
        ├── aardsound           # aardsound role
        ├── snapserver          # snapserver role
        ├── spotify             # spotify role
        └── spotify_multiroom   # spotify_multiroom role (this role)
```
The use of `include_role` is important for the sequencing of handler definitions: see the
[**Handlers**](#Handlers) section below.
 
This role installs and configures (or removes) four `systemd` services:
- A "oneshot" to create a FIFO to be used by the Snapcast server service
- A Spotify source (using the `spotify` role)
- An FFMPEG bridge service to connect the source service to the sink service
- A Snapcast server (using the `snapserver` role)

The bridge service is a workaround for an incompatibility between recent versions of Spotify Connect
and Snapcast: **Spotify** Connect now uses the positioning of the playback device to determine
whether to keep sending data and **Snapcast** doesn't support this.
This results in **Spotify Connect** skipping to the next track as soon as it attempts playback.
The [solution](https://gist.github.com/itskevinb/59630c0a57148ab63cb6325ae6e26da9) to this problem
(thanks to [itskevinb](https://gist.github.com/itskevinb)) is to insert FFMPEG in front of the
FIFO used by the **Snapcast** server that will provide this information to **Spotify**.
An ALSA loopback device (`snd-aloop`) is used to connect the `librespot` service used for Spotify to
FFMPEG.

## Variables

This section describes the variables defined in the file `./defaults/main.yml`, that are intended to
be modified as required by setting values in the Ansible inventory to suit the required Aardsound
configuration.
Internal variables defined in `./vars/main.yml` have a higher variable precedence than
inventory variables and should not be modified (e.g. by setting extra variables or role parameters).

#### spotify_multiroom_active = *boolean*
Should the Spotify multi-room services be installed and configured or removed

Default: `true`

#### spotify_multiroom_name = *string*
The name of the Spotify Connect service to be advertised via ZeroConf

Sets: `spotify_location`

Default: Multiroom

#### spotify_multiroom_format = F64 | F32 | S32 | S24 | S24_3 | S16
Spotify output format to FFMPEG.

Sets: `spotify_format`

Default: S16

#### spotify_multiroom_volume_control = cubic | fixed | linear | log
Spotify volume control scale type

Sets: `spotify_volume_control`

Default: cubic

#### spotify_multiroom_initial_volume = 0-100
Sets: `spotify_initial_volume`

Spotify initial volume as a percentage from 0 to 100

Default: 50

#### spotify_multiroom_port = 1025-65535
The port the `librespot` daemon binds to and advertises over ZeroConf.

Sets: `spotify_port`

Default: `none` (`librespot` internal default is to bind to a random high port)

#### spotify_multiroom_interface = *ipaddress* | &lsqb;*ipaddress*,&hellip;&rsqb; | *ipaddress*,*ipaddress*&hellip;
Interface IP addresses or a list of IP addresses or a comma-separated string of IP addresses to
which the `librespot` daemon will bind and advertise over ZeroConf.
Example: "192.168.0.10,10.0.0.10".

Sets: `spotify_interface`

Default: `none`  (`librespot` internal default is to bind to all interfaces)

#### spotify_multiroom_source_verbose = *boolean*
Whether to enable verbose logging in `librespot`
Logs are seent to the `systemd` journal.

Sets: `spotify_verbose`

Default: `false`

#### spotify_multiroom_fifo = *path*
The path for the FIFO that will provde input to the Snapcast server.
Does not require "multiroom" in the name because it is only used for multi-room Spotify.

Default: /tmp/spotify-fifo

#### spotify_multiroom_rate = *integer*
The audio sample rate to send to snapserver

Sets: `snapserver_rate`

Default: 44100 (untested with any other value)

#### spotify_multiroom_depth = *integer*
Sets the sampling depth for the analogue audio

Sets: `snapserver_depth`

Default: 16 (untested with any other value)

#### spotify_multiroom_channels = *integer*
The number of channels 

Sets: `snapserver_channels`

Default: 2 (untested with any other value)

#### spotify_multiroom_snap_port_offset = -699 to -2, 2 to 63755
Aritmetic offset to the default ports used by the Snapcast server (1704, 1705, 1780) for this
instance.

Note that the values above are the ones that will not cause any direct clash with this instance
of the Snapcast server only.
No checking is done on the input value.

Sets:`snapserver_port_offset`

Default: 0

#### spotify_multiroom_sink_debug = *boolean*
Enable debugging of the Snapcast server.
May be overridden by `spotify_multiroom_sink_logging`.

Sets: `snapserver_debug`

Default: false

#### spotify_multiroom_sink_logging = *string*
Set log levels for the Snapcast server.
Provides more fine-grained control of debugging than `spotify_multiroom_sink_debug`

Sets: `snapserver_logging`

Default: *:debug if `spotify_multiroom_sink_debug` is true. otherwise `none`
(Snapcast internal default: *:info)


## Variable interactions with imported roles

### spotify
Note that there are several variables described here where `spotify_multiroom_foo` corresponds to a
`spotify` role variable called `spotify_foo`.
When `spotify_multiroom` imports `spotify` it assigns variables as:
```
  vars:
    spotify_foo: "{{ spotify_multiroom_foo }}
    spotify_bar: "{{ spotify_multiroom_bar }}
```
Because of the high predecence of variables assigned at role imports, any default or inventory
defintions of `spotify_foo` will be ignored in favour of `spotify_multiroom_foo` (see 
[Ansible variable precedence](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_variables.html#understanding-variable-precedence)).

This is by design and is intended for the case where a single-room instance and a multi-room
instance of Spotify may or may not co-exist.
Take the variables `spotify_active` and `spotify_multiroom_active`, which determine whether
the two instances are active, and both default to `false`:
- setting `spotify_active: true` will enable the single-room instance but not the multi-room
  instance
- setting `spotify_multiroom_active: true` will enable the multi-room instance but not the
  single-room instance

In some cases, the variable names do not match exactly, notably `spotify_multiroom_name` sets
`spotify_location`: the `spotify` role assumes it is a single-room deployment but the concept
of `location` isn't helpful for a multi-room deployment, so `spotify_multiroom_name` is used
instead.

Where `spotify` role variables that have no `spotify_multiroom` instance, this is for one of two
reasons:
- The variable does not apply to the multi-room instance, e.g., `spotify_output_device`
- The same value is valid for both the single- and multi-room instances, e.g.,
  `spotify_volume_normalization`

### snapserver
A similar situation arises with variables for the `snapserver` role where there may be multiple
instances of the Snapcast server, for example for Spotify and Mopidy.
When `spotify_multiroom` imports `snapserver` it assigns variables as:
```
  vars:
    snapserver_foo: "{{ spotify_multiroom_foo }}
    snapserver_bar: "{{ spotify_multiroom_bar }}
```
The most significant `snapserver` role variable is `snapserver_port_offset`.
The Snapcast server uses three TCP ports, which by default are: 1704, 1705 and 1780.
With multiples instances, of `snapserver` all of them bar one must change the ports that are used.
The `snapserver` role uses the variable `snapserver_port_offset` (with a default value of 0) to
apply a single arithmetical offset to the three ports.  A value of `100` would change them to
`1804`, `1805` and `1880` respectively.
Setting `snapserver_port_offset` as a variable directly would apply the same offset to each
instance and defeat the purpose of the offset.
Therefore, the variable `spotify_multiroom_snap_port_offset` is used (note the additional string
`_snap` in the name) which is only applied to the `snapserver` instance created by
`spotify_multiroom`.

## Handlers

Handlers of a given name will be overrwritten by subsequent definitions and the dynamic definitions
of an include are always the last to be applied.
See [Ansible Handlers: running operations on change](
    https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_handlers.html#handlers-running-operations-on-change)

The including role is responsible for the flushing of handlers if there is a possibility of
including this role again (or including either of its imported roles) and thus re-writing variables
(such as service names) before the end of the play, when handlers are normally flushed.

### Handlers in this role:
- Enable Spotify Multiroom FIFO servvice
- Enable and (re)start Spotify Multiroom bridge servvice
- Reload systemd


## License

See [License.md](../../License.md)
