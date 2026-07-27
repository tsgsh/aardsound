# `snapserver` role (part of Aardsound)

For a fuller description of the operation of the Aardsound project, see the
[Aardsound README](../../README.md).

The `snapserver` role is *imported* by the `spotify_multiroom` and the `mopidy_multiroom` roles,
which are *included* by Enable and (re)start Snapserver client(s) `aardsound role`.
The `aardsound` role, is imported by the `aardsound.yml` playbook.
The paths of these are related as follows:
```
.
└── aardsound                   # Github directory
    ├── aardsound.yml           # aardsound playbook
    └── roles
        ├── aardsound           # aardsound role
        ├── mopidy_multiroom    # mopidy_multiroom role
        ├── snapserver          # snapserver role (this role)
        └── spotify_multiroom   # spotify_multiroom role
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

#### snapserver_active = *boolean*
Should the Snapcast server service be installed and configured or removed

Default: `true`

#### snapserver_service = *string*
The name of the `systemd` service running mpidy.
Also sets the name of the user that owns the service: this is `_snapserver` if the service is named
`snapserver` (the defaults for the Snapcast package), otherwise the user name is set to the name of
the service.

Default: snapserver

#### snapserver_description = *string*

Used to set the `systemd` unit description and the comment (GECOS) for the user

Default: Snapcast server

#### snapserver_codec = *codec*[:*options*]
Where *codec* = flac | ogg | opus | pcm

and *options* are specific to the selected codec

Default: flac

#### snapserver_rate = *integer*
The audio sampling bit rate

Default: 44100  (not tested with any other value)

#### snapserver_depth = *integer*
The audio sampling depth

Default: 16  (not tested with any other value)

#### snapserver_channels = *integer*
The number of audio sampling channels

Default: 2  (not tested with any other value)

#### snapserver_port_offset = -699 to -2, 2 to 63755
Aritmetic offset to the default ports used by Snapcast (1704, 1705, 1780) for this instance.

Note that the values above are the ones that will not cause any direct clash with this instance
of the Snapcast server only.
No checking is done on the input value.

Overridden by `snapserver_http_port`, `snapserver_tcp_port` and/or `snapserver_stream_port`

Default: 0

#### snapserver_http_port = *1025-65535*
Explicilty set the HTTP port for control clients and HTTP streaming

Default: 1780 plus the value of `snapserver_port_offset`

#### snapserver_tcp_port = *1025-65535*
Explicitly set the port used for the Snapcast JSON-RPC control service

Default: 1705 plus the value of `snapserver_port_offset`

#### snapserver_stream_port = *1025-65535*
Explicitly set the port used for the Snapcast streaming service

Default: 1704 plus the value of `snapserver_port_offset`

#### snapserver_fifo = *path*
The name of the FIFO used to feed input to the Snapcast server.

Note that on RasPiOS Trixie, `/tmp` is a RAM-based `tmpfs` file system (unless modified).
It is not, therefore, optimal to place the FIFO in the `/run/<`*`uid`*`>` directory (where *`uid`*, is
the UID of the user defined by `snapserver_service` as noted above); this would also be a `tmpfs`
filesystem.

Placing the FIFO in `/run` could potentially lead to memory starvation, because `/tmp` can
consume up to half of the available RAM, and `/run` usage would be in addition to that.

Default: /tmp/snapfifo

#### snapserver_fifo_service = *string*
The name of a `systemd` "oneshot" service that will create the FIFO at boot time.

Default: `none` (no server to be created)

#### snapserver_debug = *boolean*
Enable debugging of the Snapcast server.
May be overridden by `snapserver_logging`.

Default: false

#### snapserver_logging = *string*
Set log levels for the Snapcast server.
Provides more fine-grained control of debugging than `snapserver_debug`

Default: *:debug if `snapserver_debug` is true, otherwise `none` (Snapcast internal default: *:info)

#### snapserver_extra_groups = *list*
A list of groups that the user running the Snapcast server (as defined by 
`snapcast_service`) should belong to, in addition to `_snapserver` (the
primary group) and `audio`.
Defines entries in `/etc/group` and the `SupplementaryGroup=` statement in the `systemd` unit file.

Default: []


## Handlers

Handlers of a given name will be overrwritten by subsequent definitions and the dynamic definitions
of an include are always the last to be applied.
See [Ansible Handlers: running operations on change](
    https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_handlers.html#handlers-running-operations-on-change)

The including role is responsible for the flushing of handlers if there is a possibility of
including this role again (or including either of its imported roles) and thus re-writing variables
(such as service names) before the end of the play, when handlers are normally flushed.

### Handlers in this role:
- Enable and (re)start Snapcast server servvice


## License

See [License.md](../../License.md)

