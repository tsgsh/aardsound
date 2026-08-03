# `mopidy_multiroom` role (part of Aardsound)

For a fuller description of the operation of the Aardsound project, see the
[Aardsound README](../../README.md).

The `mopidy_multiroom` role is *included* by the `aardsound` role and *imports* the 
`mopidy` and `snapserver` roles.
The `aardsound` role, is imported by the `aardsound.yml` playbook.
The paths of these are related as follows:
```
.
└── aardsound                   # Github directory
    ├── aardsound.yml           # aardsound playbook
    └── roles
        ├── aardsound           # aardsound role
        ├── mopidy              # mopidy role
        ├── mopidy_multiroom    # mopidy_multiroom role (this role)
        └── snapserver          # snapserver role
```
The use of `include_role` is important for the sequencing of handler definitions: see the
[**Handlers**](#Handlers) section below.
 
This role installs and configures (or removes) four `systemd` services:
- A "oneshot" to create a FIFO to be used by the Snapcast server service
- A Mopidy source (using the `mopidy` role)
- A Snapcast server (using the `snapserver` role)

## Variables

This section describes the variables defined in the file `./defaults/main.yml`, that are intended to
be modified as required by setting values in the Ansible inventory to suit the required Aardsound
configuration.
Internal variables defined in `./vars/main.yml` have a higher variable precedence than
inventory variables and should not be modified (e.g. by setting extra variables or role parameters).

#### mopidy_multiroom_active = *boolean*
Should the Mopidy multi-room services be installed and configured or removed

Default: `true`

#### mopidy_multiroom_name = *string*
The name of the Mopidy service to be advertised

Sets: `mopidy_location`

Default: Multiroom

#### mopidy_multiroom_use_venv = *boolean*
Should Mopidy be installed in a Python virtual environment.
This should be set to `true` if there are mopidy extensions defined that are not available as APT
packages
(see [mopidy_extensions](#mopidy_extensions--dictionary) below).

Sets: `mopidy_use-venv`

Default: false

#### mopidy_multiroom_venv = *path*
The path where the Mopidy virtual environment should be created.

Sets: `mopidy_venv`

Default: /usr/local/mopidy/venv

#### mopidy_multiroom_volume_control = cubic | fixed | linear | log
Mopidy volume control scale type

Sets: `mopidy_volume_control`

Default: cubic

#### mopidy_multiroom_initial_volume = 0-100
Mopidy initial volume as a percentage from 0 to 100

Default: 50

#### mopidy_multiroom_media_library = *path*
The media library for `mopidy-local`

If this is not `/var/lib/mopidy/media` then the specified directory is bind mounted to there using a
`systemd` mount unit.

Sets: `mopidy_media_library`

Default: /var/lib/mopidy/media

#### mopdiy_multiroom+mount_after = *list of strings*
A list of `systemd` mount units that are needed before the `systemd` unit that performs the bind
mount can run: each entry is used to create an `After=` entry in the latter.

Sets: `mopidy_mount_after`

Default: []

#### mopidy_multiroom_http_bind_address =  *IP address*

*IP address*

Address to bind the `mopidy-http` extension to

Sets: `mopidy_http_bind_address`

Default: 
- '::' if `mopidy_trusted_network` and `mopidy_ipv6` are true
- '0.0.0.0' if `mopidy_trusted_network` is true and `mopidy_ipv6` is false
- '::1' if `mopidy_trusted_network` is false and `mopidy_ipv6` is true
- '127.0.0.1' if `mopidy_trusted_network` and `mopidy_ipv6` are false

Default: 127.0.0.1

#### mopidy_multiroom_http_port = 1025-65535
Port to bind the `mopidy-http` extension to

Sets: `mopidy_http_bind_port`

Default: 6681

#### mopidy_multiroom_mpd_bind_address = *IP address*
Address to bind the `mopidy-mpd` extension to

Sets: `mopidy_mpd_bind_address`

Default:
- '::' if `mopidy_trusted_network` and `mopidy_ipv6` are true
- '0.0.0.0' if `mopidy_trusted_network` is true and `mopidy_ipv6` is false
- '::1' if `mopidy_trusted_network` is false and `mopidy_ipv6` is true
- '127.0.0.1' if `mopidy_trusted_network` and `mopidy_ipv6` are false

#### mopidy_multiroom_mpd_port = 1025-65535
Port to bind the `mopidy-mpd` extension to

Sets: `mopidy_mpd_bind_port`

Default: 6601

#### mopidy_multiroom_format = F64 | F32 | S32 | S24 | S24_3 | S16
Output format to send to the Snapcast server

Sets:
- `mopidy_format`
- `snapserver_format`

Default: S16

#### mopidy_multiroom_rate = *integer*
The audio sample rate to send to the Snapcast server

Sets:
- `mopidy_rate`
- `snapserver_rate`

Default: 44100 (untested with any other value)

#### mopidy_multiroom_depth = *integer*
Sets the sampling depth for the analogue audio

Sets: `snapserver_depth`

Default: 16 (untested with any other value)

#### mopidy_multiroom_channels = *integer*
The number of channels 

Sets: 
- `mopidy_channels`
- `snapserver_channels`

Default: 2 (untested with any other value)

#### mopidy_multiroom_snap_port_offset = -699 to -2, 2 to 63755
Aritmetic offset to the default ports used by the Snapcast server (1704, 1705, 1780) for this
instance.

Note that the values above are the ones that will not cause any direct clash with this instance
of the Snapcast server only.
No checking is done on the input value.

Sets:`snapserver_port_offset`

Default: 0

#### mopidy_multiroom_extensions = *dictionary of dictionaries*
This allows for the configuration of arbitrary Mopidy extensions.

Sets: [`mopidy_extensions`](../mopidy/README.md#mopidy_extensions--dictionary-of-dictionaries)

Default: {}

#### mopidy_multiroom_fifo = *path*
The path for the FIFO that will provde input to the Snapcast server.
Does not require "multiroom" in the name because it is only used for multi-room Spotify.

Default: /tmp/mopidy-fifo

#### mopidy_multiroom_sink_debug = *boolean*
Enable debugging of the Snapcast server.
May be overridden by `mopidy_multiroom_sink_logging`.

Sets: `snapserver_debug`

Default: false

#### mopidy_multiroom_sink_logging = *string*
Set log levels for the Snapcast server.
Provides more fine-grained control of debugging than `mopidy_multiroom_sink_debug`

Sets: `snapserver_logging`

Default: *:debug if `mopidy_multiroom_sink_debug` is true, otherwise `none`
(Snapcast internal default: *:info)


## Variable interactions with imported roles

### mopidy
Note that there are several variables described here where `mopidy_multiroom_foo` corresponds to a
`mopidy` role variable called `mopidy_foo`.
When `mopidy_multiroom` imports `mopidy` it assigns variables as:
```
  vars:
    mopidy_foo: "{{ mopidy_multiroom_foo }}
    mopidy_bar: "{{ mopidy_multiroom_bar }}
```
Because of the high predecence of variables assigned at role imports, any default or inventory
defintions of `mopidy_foo` will be ignored in favour of `mopidy_multiroom_foo` (see 
[Ansible variable precedence](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_variables.html#understanding-variable-precedence)).

This is by design and is intended for the case where a single-room instance and a multi-room
instance of Mopidy may or may not co-exist.
Take the variables `mopidy_active` and `mopidy_multiroom_active`, which determine whether
the two instances are active, and both default to `false`:
- setting `mopidy_active: true` will enable the single-room instance but not the multi-room
  instance
- setting `mopidy_multiroom_active: true` will enable the multi-room instance but not the
  single-room instance

In some cases, the variable names do not match exactly, notably `mopidy_multiroom_name` sets
`mopidy_location`: the `mopidy` role assumes it is a single-room deployment but the concept
of `location` isn't helpful for a multi-room deployment, so `mopidy_multiroom_name` is used
instead.

Where `mopidy` role variables that have no `mopidy_multiroom` instance, this is for one of two
reasons:
- The variable does not apply to the multi-room instance, e.g., `mopidy_output_device`
- The same value is valid for both the single- and multi-room instances, e.g.,
  `mopidy_local_scan`

### snapserver
A similar situation arises with variables for the `snapserver` role where there may be multiple
instances of the Snapcast server, for example for Spotify and Mopidy.
When `mopidy_multiroom` imports `snapserver` it assigns variables as:
```
  vars:
    snapserver_foo: "{{ mopidy_multiroom_foo }}
    snapserver_bar: "{{ mopidy_multiroom_bar }}
```
The most significant `snapserver` role variable is `snapserver_port_offset`.
The Snapcast server uses three TCP ports, which by default are: 1704, 1705 and 1780.
With multiples instances, of `snapserver` all of them bar one must change the ports that are used.
The `snapserver` role uses the variable `snapserver_port_offset` (with a default value of 0) to
apply a single arithmetical offset to the three ports.  A value of `100` would change them to
`1804`, `1805` and `1880` respectively.
Setting `snapserver_port_offset` as a variable directly would apply the same offset to each
instance and defeat the purpose of the offset.
Therefore, the variable `mopidy_multiroom_snap_port_offset` is used (note the additional string
`_snap` in the name) which is only applied to the `snapserver` instance created by
`mopidy_multiroom`.

## Handlers

Handlers of a given name will be overrwritten by subsequent definitions and the dynamic definitions
of an include are always the last to be applied.
See [Ansible Handlers: running operations on change](
    https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_handlers.html#handlers-running-operations-on-change)

The including role is responsible for the flushing of handlers if there is a possibility of
including this role again (or including either of its imported roles) and thus re-writing variables
(such as service names) before the end of the play, when handlers are normally flushed.

### Handlers in this role:
- Enable Mopidy Multiroom FIFO servvice
- Reload systemd


## License

See [License.md](../../License.md)
