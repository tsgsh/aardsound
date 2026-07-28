# `dmixer` role (part of Aardsound)

For a fuller description of the operation of the Aardsound project, see the
[Aardsound README](../../README.md).

The `dmixer` role is imported by the `aardsound` role, which is in turn imported by the
`aardsound.yml` playbook.
The paths of the three are related as follows
```
.
└── aardsound               # Github directory
    ├── aardsound.yml       # aardsound playbook
    └── roles
        ├── aardsound       # aardsound role
        └── dmixer          # dmixer role (this role)
```

This role installs (or removes) an ALSA direct mixer (`dmix`) device if there are (or are not)
multiple audio inputs sharing an ALSA output device.

Two entries are create in `/etc/asound.conf`
- A plugin that is used by multiple upstream audio services as an output device
- a PCM of type **dmix** that is slaved to the plugin and routes audio output to a single downstream
  ALSA device


## Variables

This section describes the variables defined in the file `./defaults/main.yml`, that are intended to
be modified as required by setting values in the Ansible inventory to suit the required Aardsound
configuration.
Internal variables defined in `./vars/main.yml` have a higher variable precedence than
inventory variables and should not be modified (e.g. by setting extra variables or role parameters).

#### dmixer_active = true | false
Should the direct mixer plugins be defined in `/etc/asound.conf`

Default: `true`

#### dmixer_card = *string* | *integer*
Card for the `ctl` entry in the configuration, must correrspond to `dmixer_output_device`

Default: 0

#### dmixer_plugin = *string*
Name of the plugin that that accepts multiple sources (as `plug:`*`dmixer_plugin`*)

Default: dmixer

#### dmixer_dmix_device = *string*
Name of the dmix type PCM that is a slave to *`dmixer_plugin`*

Default: dmix

#### dmixer_output_device = *string*
Output PCM that is a slave to *`dmixer_dmix_device`*

Default: default

#### dmixer_rate = *integer*
Sampling rate for *`dmixer_output_device`*

Default: 44100

#### dmixer_period_time = *integer*
Period time for *`dmixer_output_device`* in μs

A period time that is a multiple of 10,000μs or 10ms will avoid rounding errors with either 44.1kHz
streams (Spotify and Bluetooth A2DP defaults) or any streams with bitrates in whole kHz: 44.1kHz
generates 441 audio frames in 10ms and 48kHz generates 480 frames.
Setting the period time that does not contain a whole number of frames can cause significant
synchronization drift.

See [https://github.com/arkq/bluez-alsa/blob/master/doc/bluealsa-aplay.1.rst#dmix].

Default: 20000

#### dmixer_periods = *integer*
The number of sample periods that will be buffered,

Default: 4

#### dmixer_ipc_key = 0-65535
IPC key for *`dmixer_output_device`*; arbitrary but not random.
Used to distinguish this IPC instance from others

Default: 54321

#### dmixer_channels = *integer*
Channels for *`dmixer_output_device`*

Default: 2; not tested with other values


## Handlers

This role does not have handlers.


## License

See [License.md](../../License.md)

