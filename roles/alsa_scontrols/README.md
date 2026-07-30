# `alsa_scontrols` role (part of Aardsound)

For a fuller description of the operation of the Aardsound project, see the
[Aardsound README](../../README.md).

The `alsa_scontrols` role is *imported* by the `aardsound` role, which is imported by the
`aardsound.yml` playbook.
The paths of these are related as follows:
```
.
└── aardsound                   # Github directory
    ├── aardsound.yml           # aardsound playbook
    └── roles
        ├── aardsound           # aardsound role
        └── alsa_scontrols      # alsa_scontrols role (this role)
```

This role configures one or more ALSA "simple controls" for the sound card used for
Aardsound audio output.
The controls would typically be specific to a specific hardware type and/or a group of hosts.

For example, the [IQaudio DigiAMP+](https://shop.pimoroni.com/products/pi-digiamp) has two mixer controls
called "Analogue" and "Analogue Playback Boost": both 
[should be set to 100%](https://www.manualslib.com/manual/1172395/Iqaudio-Pi-DacPlus.html?page=26&term=Boost&selected=1#manual).

To do this, set
```
alsa_scontrols_controls:
  'Analogue': '100%'
  'Analogue Playback Boost': '100%'
```
as an inventory variable for each host using this card.

**Notes**:

- The variable that defines the ALSA simple controls is actually `alsa_scontrols_controls` but
  for brevity and convenience, it is configured to default to the value of `alsa_scontrols` or an
  empty dictionary if the latter is undefined
- The role is idempotent: it checks the current controls for the card and only sets controls that
  are currently unset or need to be modified
- There is no mechanism to remove controls via this role; controls can be given a null value
- No checks are performed on the validity of any controls before an attempt is made to set them
- Restoration of the state of the controls across boots is managed by the `systemd` service
 `alsa-restore.service`


## Variables

This section describes the variables defined in the file `./defaults/main.yml`, that are intended to
be modified as required by setting values in the Ansible inventory to suit the required Aardsound
configuration.
Internal variables defined in `./vars/main.yml` have a higher variable precedence than
inventory variables and should not be modified (e.g. by setting extra variables or role parameters).

#### alsa_scontrols_card = *string* | *integer* 
The card to which the controls are to be applied.

Default: 0

#### alsa_scontrols_controls = *dictionary*
A dictionary of controls to be set with `amixer sset`
- The key is the name of the control
- The value is the value that the control is to be set to

If the dictionary is empty, no controls will be defined.

Default: the value of `alsa_scontrols` if defined, otherwise `{}`


## Handlers

This role does not have handlers.


## License

See [License.md](../../License.md)

