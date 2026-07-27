# `raspotify_installer` role (part of Aardsound)

For a fuller description of the operation of the Aardsound project, see the
[Aardsound README](../../README.md).

The `raspotify_installer` role is imported by the `aardsound` role, which is in turn imported by the
`aardsound.yml` playbook.
The paths of the three are related as follows
```
.
└── aardsound                       # Github directory
    ├── aardsound.yml               # aardsound playbook
    └── roles
        ├── aardsound               # aardsound role
        └── raspotify_installer     # raspotify_installer role (this role)
```

This role installs the [raspotify](https://dtcooper.github.io/raspotify/) package from `github`;
this is a Debian/RasPiOS packaging of [librespot](https://github.com/librespot-org/librespot) that
"mostly just works".

The role installs the `raspotify` package and then masks the `raspotify` `systemd` service: this
is to allow Aaardsound to create several custom services based on `raspotify`

## Variables

No variables need to be defined for this role.

## Handlers

This role does not have handlers.


## License

See [License.md](../../License.md)
