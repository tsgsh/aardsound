# `mopidy_installer` role (part of Aardsound)

For a fuller description of the operation of the Aardsound project, see the
[Aardsound README](../../README.md).

The `mopidy_installer` role is imported by the `aardsound` role, which is in turn imported by the
`aardsound.yml` playbook.
The paths of the three are related as follows
```
.
└── aardsound                   # Github directory
    ├── aardsound.yml           # aardsound playbook
    └── roles
        ├── aardsound           # aardsound role
        └── mopidy_installer    # mopidy_installer role (this role)
```

This role installs [Mopidy](https://mopidy.com/) &ndash; an extensible music server written in
Python &ndash; and its bundled extensions that are supplied with RasPiOS/Debian and, optionally,
additional extensions via PIP or even from Github.

## Variables

This section describes the variables defined in the file `./defaults/main.yml`, that are intended to
be modified as required by setting values in the Ansible inventory to suit the required Aardsound
configuration.
Internal variables defined in `./vars/main.yml` have a higher variable precedence than
inventory variables and should not be modified (e.g. by setting extra variables or role parameters).

#### mopidy_installer_use_venv = *boolean*

Should Mopidy be installed in a Python Virtual Enviromnment

Default: 
`true` if the host
- the server has more than 512MiB of RAM; or
- is *not* running a desktop environment; or
- has any defined (i.e., non-bundled) extensions that require installtion via PIP
otherwise: `false`
#### mopidy_installer_venv = *path*
The path to the virtual environment

Default: `/usr/local/mopidy/venv`

#### mopidy_installer_nfs = *host*:*path*
An NFS volume containing music files for the bundled `mopidy-local` extension

An entry will be created in `/etc/fstab` to mount this volume if required

Default: `none`

#### mopidy_installer_nfs_options = *string*
Options for the NFS mmount

Default: bg,ro

#### mopidy_installer_library = *path*
The NFS mount point

Default: `/mnt/mopidy` if `mopidy_installer_nfs` is defined, otherwise `none`

#### mopidy_installer_extensions = *list*
A list of Mopidy extensions to be installed using PIP.

Default: `[]`

#### mopidy_installer_sudoers = *list*
A list of privileged commands to be added to `/etc/sudoers.d` to support Mopidy extensions

These are created as
```
%mopdiy ALL=NOPASSWD: <item>
```
in the file `/etc/sudoers.d/50-mopidy`, where `<`*`item`*`>` is an entry in the list


Default: `[]`

#### mopidy_installer_install = *list of dictionaries*
A list of commands to run to perform Mopidy extension installation tasks.

Each entry is a dictionary with one key/value pair.
The key is the user that will run the command (e.g. `root` or `mopidy`) and the value is a shell
command to be run as follows:
```
set -o pipefail
<command>
```
where `<`*`command`*`>` is the value of the dictionry entry.


Default: `[]`


## Handlers

This role has one handler:
- Reload systemd


## License

See [License.md](../../License.md)
