### Version 0.3.1
28 August 2026
- Correct issues in bluetooth_adapter default
- Correct bluetooth device class advertisement
- Remove capability to set spotify and mopidy IP addresses and ports using aardsound role variables 
  it was unnecessary, unlikely to be used and did not work correctly setting defaults for mopidy-mpd
  or mopidy-http addresses causing both extensions to fail.

### Version 0.3.0
13 August 2026
- Enable multi-room Bluetooth audio sources
- Fixes to Bluetooth service
- Fix Mopidy NFS mount systemd dependencies
- Clean up of systemd descriptions
- Clean up of Ansible task and variable names
- Minor bug fixes

#### Notes
The development of multi-room bluetooth went down a few blind alleys.
- The first of these was attempting to support single- and multi-room bluetooth instances together:
  this doesn't work.
- The second of these was a consequence of the first: trying to use different
  service names for the multi-room bluetooth daemons: renaming the bluealsa daemon was problematic.
- The third was trying to remove the selected bluetooth adapter from Pipewire to make Bluetooth
  more reliable with a desktop distribution: this caused login issues.
  It's probably better to use a lite RasPiOS distribution if you want to use Bluetooth (either the
  single- or multi-room variant).
  I did not ever diagnose what caused one particular system to not play Bluetooth when it was
  ostensibly identical to another that did (possibly a race condition between the `bluealsa` daemon
  and the Pipewire daemon for the desktop user).

### Version 0.2.0
04 August 2026
- Enable single-room Bluetooth audio sources
- Add role validation via meta/main.yml

### Version 0.1.1
27 July 2026
- Corrections to README.md and Examples.md

### Version 0.1.0
27 July 2026
- Initial Github Release