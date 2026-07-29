# `snapclient` role (part of Aardsound)

For a fuller description of the operation of the Aardsound project, see the
[Aardsound README](../../README.md).

The `snapclient` role is *imported* by the `aardsound` role, which is imported by the
`aardsound.yml` playbook.
The paths of these are related as follows:
```
.
└── aardsound                   # Github directory
    ├── aardsound.yml           # aardsound playbook
    └── roles
        ├── aardsound           # aardsound role
        └── snapclient          # snapclient role (this role)
```

This role installs and configures one or more Snapcast client instances as `systemd` services and
connects them to a single ALSA output PCM.

With multiple clients, this should be a direct mixer (`dmix`) PCM; with only one instance, it may be
a PCM with a hardware (`hw`) or plug-hardware (`plughw`) ALSA plugin connected to a sound card.
If multiple clients connect to the same `hw` or `plughw` PCM, one will succeed and the others
will be blocked.


## Variables

This section describes the variables defined in the file `./defaults/main.yml`, that are intended to
be modified as required by setting values in the Ansible inventory to suit the required Aardsound
configuration.
Internal variables defined in `./vars/main.yml` have a higher variable precedence than
inventory variables and should not be modified (e.g. by setting extra variables or role parameters).

#### snapclient_active = *boolean*
Should the Snapcast client service be installed and configured or removed

Default: `true`


#### snapclient_of = *list of dictionaries*
Each entry corresponds to a Snapcast client to be created.  Three keys are valid, one of which is
mandatory:

- `host` &ndash; the hostname, FQDN or IP address of the Snapcast server to connect to (mandatory)
- `port` &ndash; the port to connect to (default: 1704)
- `name` &ndash; a name to describe the Snapserver service; defaults to the value of `host` if
  `port` equals 1704, otherwise `host`:`port`

Defines the name of the `systemd` service and the name of the user running the service:
- Convert to lower case
- Replace whitespace, colons, underscores and dots with hyphens
- Remove any character that is not a letter in the range 'a-z', a digit or a hyphen
- Truncate at 255 characters for the service name and 32 for the user name

names must be unique unless omitted, `host`:`port` combinations must be unique, user name must be
unique and ports must be in the range 1025-65535.

Default: []  (no clients to be created)


#### snapclient_location = *string*
The place where the audio output can be heard, e.g. 'Kitchen'

Default: `none`

#### snapclient_output_device = *string*
The ALSA PCM to which `snapclient` sends its audio stream

Default: default


#### snapclient_mixer = *boolean*
Use an ALSA hardware mixer?

Default: `false`

#### snapclient_mixer_plugin = *string*
ALSA hardware mixer control as reported by the sound card

Default: hw; ignored if `snapclient_mixer` = `false`

#### snapclient_mixer_card = *string* | *integer* 
The sound card to use for the hardware mixer

Default: 0; ignored if `snapclient_mixer` = `false`

#### snapclient_mixer_control = *string*
The control to use for the hardware mixer

Default: Master; ignored if `snapclient_mixer` = `false`

#### snapclient_rate = *integer*
The audio sampling bit rate

Default: 44100  (not tested with any other value)

#### snapclient_depth = *integer*
The audio sampling depth

Default: 16  (not tested with any other value)

#### snapclient_channels = *integer*
The number of audio sampling channels

Default: 2  (not tested with any other value)

#### snapclient_debug = *boolean*
Enable debugging of the Snapcast client

May be overridden by `snapsclient_logging`

Default: false

#### snapclient_logging = *string*
Set log levels for the Snapcast client

Provides more fine-grained control of debugging than `snapclient_debug`

Default: *:debug if `snapclient_debug` is true, otherwise `none`


## Handlers

Unlike many of the other roles called by the `aardsound` role, this role does not require any
particular care with the flushing of handlers.  That is because it is called once to implement all
of the required Snapcast client instances instead of being included repeatedly.

### Handlers in this role:
- Reload systemd
- Enable and (re)start Snapserver client(s)


## License

See [License.md](../../License.md)

