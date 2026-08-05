# `bluetooth_multiroom` role (part of Aardsound)

For a fuller description of the operation of the Aardsound project, see the
[Aardsound README](../../README.md).

The `bluetooth_multiroom` role is *included* by the `aardsound` role and *imports* the 
`bluetooth` and `snapserver` roles.
The `aardsound` role, is imported by the `aardsound.yml` playbook.
The paths of these are related as follows:
```
.
└── aardsound                     # Github directory
    ├── aardsound.yml             # aardsound playbook
    └── roles
        ├── aardsound             # aardsound role
        ├── snapserver            # snapserver role
        ├── bluetooth             # bluetooth role
        └── bluetooth_multiroom   # bluetooth_multiroom role (this role)
```
The use of `include_role` is important for the sequencing of handler definitions: see the
[**Handlers**](#Handlers) section below.
 
This role installs and configures (or removes) five `systemd` services:
- A "oneshot" to create a FIFO to be used by the Snapcast server service
- A Bluetooth agent (using the `bluetooth` role)
- A BlueALSA aplay service (using the `bluetooth` role)
- A Snapcast server (using the `snapserver` role)


## Variables

This section describes the variables defined in the file `./defaults/main.yml`, that are intended to
be modified as required by setting values in the Ansible inventory to suit the required Aardsound
configuration.
Internal variables defined in `./vars/main.yml` have a higher variable precedence than
inventory variables and should not be modified (e.g. by setting extra variables or role parameters).

#### bluetooth_multiroom_active = *boolean*
Should the Bluetooth multi-room services be installed and configured or removed

Default: `true`

#### bluetooth_multiroom_name = *string*
The name of the Bluetooth service to be advertised via ZeroConf

Sets: `bluetooth_location`

Default: Multiroom

#### bluetooth_multiroom_volume = auto | mixer | none | software
Bluetooth remote volume control

Sets: `bluetooth_volume`

Default: cubic

#### bluetooth_multiroom_initial_volume = 0-100
Sets: `bluetooth_initial_volume`

Bluetooth initial volume as a percentage from 0 to 100

Default: 50

#### bluetooth_multiroom_port = 1025-65535
The port the `librespot` daemon binds to and advertises over ZeroConf.

Sets: `bluetooth_port`

Default: `none` (`librespot` internal default is to bind to a random high port)

#### bluetooth_multiroom_interface = *ipaddress* | &lsqb;*ipaddress*,&hellip;&rsqb; | *ipaddress*,*ipaddress*&hellip;
Interface IP addresses or a list of IP addresses or a comma-separated string of IP addresses to
which the `librespot` daemon will bind and advertise over ZeroConf.
Example: "192.168.0.10,10.0.0.10".

Sets: `bluetooth_interface`

Default: `none`  (`librespot` internal default is to bind to all interfaces)

#### bluetooth_multiroom_source_verbose = true | false
Should verbose logging be set for the BlueALSA daemon?

Sets: `bluetooth_verbose`

Default: 0

#### bluetooth_multiroom_aloop_card = *string*
The name of the ALSA card that is to be used for the loopback device between BlueALSAA and the
Snapcast server

If the card does not exist it will be created with enough substreams to support the value specified
in `bluetooth_multiroom_aloop_substream`

Default: Loopback

#### bluetooth_multiroom_aloop_substream = *int*
The subetream of the loopback ALSA card that is to be used

If the card specified in `bluetooth_multiroom_aloop_card` exists but has insufficient substreams,
am error will occur

Default: 0

#### bluetooth_multiroom_fifo = *path*
The path for the FIFO that will provde input to the Snapcast server.
Does not require "multiroom" in the name because it is only used for multi-room Bluetooth.

Default: /tmp/bluetooth-fifo

#### bluetooth_multiroom_rate = *integer*
The audio sample rate to send to snapserver

Sets: `snapserver_rate`

Default: 44100 (untested with any other value)

#### bluetooth_multiroom_depth = *integer*
Sets the sampling depth for the analogue audio

Sets: `snapserver_depth`

Default: 16 (untested with any other value)

#### bluetooth_multiroom_channels = *integer*
The number of channels 

Sets: `snapserver_channels`

Default: 2 (untested with any other value)

#### bluetooth_multiroom_snap_port_offset = -699 to -2, 2 to 63755
Aritmetic offset to the default ports used by the Snapcast server (1704, 1705, 1780) for this
instance.

Note that the values above are the ones that will not cause any direct clash with this instance
of the Snapcast server only.
No checking is done on the input value.

Sets:`snapserver_port_offset`

Default: 0

#### bluetooth_multiroom_sink_debug = *boolean*
Enable debugging of the Snapcast server.
May be overridden by `bluetooth_multiroom_sink_logging`.

Sets: `snapserver_debug`

Default: false

#### bluetooth_multiroom_sink_logging = *string*
Set log levels for the Snapcast server.
Provides more fine-grained control of debugging than `bluetooth_multiroom_sink_debug`

Sets: `snapserver_logging`

Default: *:debug if `bluetooth_multiroom_sink_debug` is true. otherwise `none`
(Snapcast internal default: *:info)


## Variable interactions with imported roles

### bluetooth
Note that there are several variables described here where `bluetooth_multiroom_foo` corresponds to
a `bluetooth` role variable called `bluetooth_foo`.
When `bluetooth_multiroom` imports `bluetooth` it assigns variables as:
```
  vars:
    bluetooth_foo: "{{ bluetooth_multiroom_foo }}
    bluetooth_bar: "{{ bluetooth_multiroom_bar }}
```
Because of the high predecence of variables assigned at role imports, any default or inventory
defintions of `bluetooth_foo` will be ignored in favour of `bluetooth_multiroom_foo` (see 
[Ansible variable precedence](https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_variables.html#understanding-variable-precedence)).

This is by design and is intended for the case where a single-room instance and a multi-room
instance of Bluetooth may or may not co-exist.
Take the variables `bluetooth_active` and `bluetooth_multiroom_active`, which determine whether
the two instances are active, and both default to `false`:
- setting `bluetooth_active: true` will enable the single-room instance but not the multi-room
  instance
- setting `bluetooth_multiroom_active: true` will enable the multi-room instance but not the
  single-room instance

In some cases, the variable names do not match exactly, notably `bluetooth_multiroom_name` sets
`bluetooth_location`: the `bluetooth` role assumes it is a single-room deployment but the concept
of `location` isn't helpful for a multi-room deployment, so `bluetooth_multiroom_name` is used
instead.

Where `bluetooth` role variables that have no `bluetooth_multiroom` instance, this is for one of
two reasons:
- The variable does not apply to the multi-room instance, e.g., `bluetooth_output_device`
- The same value is valid for both the single- and multi-room instances, e.g.,
  `bluetooth_volume_normalization`

### snapserver
A similar situation arises with variables for the `snapserver` role where there may be multiple
instances of the Snapcast server, for example for Spotify, Mopidy and Bluetooth.
When `bluetooth_multiroom` imports `snapserver` it assigns variables as:
```
  vars:
    snapserver_foo: "{{ bluetooth_multiroom_foo }}
    snapserver_bar: "{{ bluetooth_multiroom_bar }}
```
The most significant `snapserver` role variable is `snapserver_port_offset`.
The Snapcast server uses three TCP ports, which by default are: 1704, 1705 and 1780.
With multiples instances, of `snapserver` all of them bar one must change the ports that are used.
The `snapserver` role uses the variable `snapserver_port_offset` (with a default value of 0) to
apply a single arithmetical offset to the three ports.  A value of `100` would change them to
`1804`, `1805` and `1880` respectively.
Setting `snapserver_port_offset` as a variable directly would apply the same offset to each
instance and defeat the purpose of the offset.
Therefore, the variable `bluetooth_multiroom_snap_port_offset` is used (note the additional string
`_snap` in the name) which is only applied to the `snapserver` instance created by
`bluetooth_multiroom`.

## Handlers

Handlers of a given name will be overrwritten by subsequent definitions and the dynamic definitions
of an include are always the last to be applied.
See [Ansible Handlers: running operations on change](
    https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_handlers.html#handlers-running-operations-on-change)

The including role is responsible for the flushing of handlers if there is a possibility of
including this role again (or including either of its imported roles) and thus re-writing variables
(such as service names) before the end of the play, when handlers are normally flushed.

### Handlers in this role:
- Enable Bluetooth Multiroom FIFO servvice
- Reload systemd


## License

See [License.md](../../License.md)
