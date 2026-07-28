# `bluetooth` role (part of Aardsound)

For a fuller description of the operation of the Aardsound project, see the
[Aardsound README](../../README.md).

The `bluetooth` role is either
- *Included* by the `aardsound` role; or
- *Imported* by the `bluetooth_multiroom` role, which is in turn *included* by the `aardsound`
  role.

The `aardsound` role, is imported by the `aardsound.yml` playbook.
The paths of these are related as follows:
```
.
└── aardsound                     # Github directory
    ├── aardsound.yml             # aardsound playbook
    └── roles
        ├── aardsound             # aardsound role
        ├── bluetooth             # bluetooth role (this role)
        └── bluetooth_multiroom   # bluetooth_multiroom role
```
The use of `include_role` is important for the sequencing of handler definitions: see the
[**Handlers**](#Handlers) section below.
 
This role configures (or removes) a Bluetooth agent that supports connections from Bluetooth clients
to use the device as a loudspeaker.

[`bt-agent`](https://manpages.debian.org/trixie/bluez-tools/bt-agent.1.en.html).

### Details
The Bluetooth service uses the following components:
- The system-wide Bluetooth service (`bluetoothd`) &ndash; configuration changes are made to the
  configuration file `/etc/bluetoothd/main.conf`
- The Bluez-ALSA Buletooth audio backend service (either `bluealsa` or `bluealsad`): no changes are
  made to the configuration
- `bluealsa` ALSA plugins &ndash; these are added to `/etc/asound.conf`
- A custom `systemd` service which:
  - Uses the `bt-adapter` utility to
    - set the Bluetooth adapter Alias to something recognisable
    - disable the discoverability and pairing mode timeouts
    - make the adapter discoverable
    - put the adapter in pairing mode
  - Runs the `bt-agent` service as daemon, optionally with a PIN file that allows devices to connect


## Variables

This section describes the variables defined in the file `./defaults/main.yml`, that are intended to
be modified as required by setting values in the Ansible inventory to suit the required Aardsound
configuration.
Internal variables defined in `./vars/main.yml` have a higher variable precedence than
inventory variables and should not be modified (e.g. by setting extra variables or role parameters).

#### bluetooth_active = *boolean*
Should the `bt-agent` service be installed and configured or removed

Note that running the role with `bluetooth_active=false` does not remove modifications to
`/etc/bluetoothd.conf` nor changes to the state or any Bluetooth adapters.

Default: `true`

#### bluetooth_service = *string*
The name of the `systemd` service running the `bt-agent`.
Also sets the name of the user that owns the service.

Default: bt-agent

#### bluetooth_location = *string*
The place where the audio output can be heard, e.g. 'Kitchen'

Used to set the friendly name of the adapter to the capitalised hostname of the server followed by
the location in parentheses.
If not defined then just the capitalised hostname of the server is used as a name.

Default: `none`

#### bluetooth_adapter = *mac_address* | hci0 | hci1 | ...

Select the the bluetooth adapter to use

The adapter will be unblocked using `rfkill` if it is soft-blocked

Default: depends upon which adapters are soft-blocked and whether there are one or more external
adapters; in order of importance:

- unblocked adapters are preferred over soft-blocked adapters (hard-blocked adapters are ignored)
- Higher indexed devices are preferred, e.g, USB adapters over the onboard Bluetooth adapter

The rationale for this is as follows: firstly, if you have soft-blocked adapters, that should be
resepected; secondly if you have installed an external bluetooth adapter, then you probably want
to use it; and thirdly, depending upon the Pi model and your usage of Aardsound, WiFi and Bluetooth
traffic may cause dropouts with each other due to sharing the same hardware

#### bluetooth_codec = SBC | MP3 | aptX | aptX-HD | FastStream | LDAC | Opus

The case-sensitive name of the A2DP codec to use

These are the supported A2DP source codecs for BlueAlsa v4.3.1 in RasPiOS Trixie 
(`bluealsa --help`)

Which codec is best for you is an impossible question to answer in a README file!

Default: SBC (supported by all bluetooth devices, it is also a fallback if the specified codec is
not supported by the client)

##### Notes:
- AAC is an Apple-proprietary and is not compiled into the RasPiOS `bluealsa` binary
  (presumably for licensing reasons)
- On an Android device, it may be necessary to enable 
  ["Developer Settings"](https://www.android.com/intl/en_uk/articles/enable-android-developer-settings/)
  to get access to some codecs such as LDAC .

#### bluetooth_output_device = *string*
The ALSA PCM to which `bluetooth` sends its audio streams

Default: default

<!-- #### bluetooth_volume_control = cubic | fixed | linear | log
Bluetooth volume control scale type

Default: cubic

#### bluetooth_initial_volume = 0-100
`bt-agent` initial volume as a percentage from 0 to 100

Default: 50
 -->
#### bluetooth_extra_groups = *list*
A list of groups that the user running the `bt-agent` (as defined by 
[`bluetooth_service`](#bluetooth_service--string)) should belong to, in addition to `bluetooth` (the primary
group) and `audio`

Default: []

#### bluetooth_start_after_services = *list*
A list of systemd services that you want to start before the `bt-agent` service.
Added to the `systemd` unit definition as an `After=` statement for each entry.
Each entry must be a valid `systemd` service of the creation of the `bt-agent` systemd unit will fail.
Do not include the `.service` suffix.

Default: `[]`

<!-- #### bluetooth_verbose
Whether to enable verbose logging in `librespot`
Logs are seent to the `systemd` journal.

Default: `false`
 -->

## Handlers

Handlers of a given name will be overrwritten by subsequent definitions and the dynamic definitions
of an include are always the last to be applied.
See [Ansible Handlers: running operations on change](
    https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_handlers.html#handlers-running-operations-on-change)

The including role is responsible for the flushing of handlers if there is a possibility of
including this role again and re-writing variables (such as service names) before the end of the
play, when handlers are normally flushed.

### Handlers in this role:
- Restart bluetoothd service
- Enable and (re)start bt-agent service
- Reload systemd


## License

See [License.md](../../License.md)
