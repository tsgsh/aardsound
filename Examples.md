# Example Configurations

## Introduction &ndash; Single Source with different HiFi Devices

```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Raspberry Pi ("Wallace")                        ┃
┃ ┌─────────┐                                     ┃
┃ │ spotify │   ┌─────────────┐   ┌────────────┐  ┃      ┏━━━━━━━━━━━┓      ┏━━━━━━━━━━┓
┃ │   or    ├───┤ ALSA device ├───┤ Sound card ├──╂──────┨ Amplifier ┠──────┨ Speakers ┃
┃ │ mopidy  │   └─────────────┘   └────────────┘  ┃      ┗━━━━━━━━━━━┛      ┗━━━━━━━━━━┛
┃ └─────────┘                                     ┃ 
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```
*Figure 1*

The **ALSA** device is the software representation of the output device on the sound card.
By default, Aardsound will select something like `hw:IQaudIODAC,0` where:
- `hw` is an **ALSA** plugin (a very simple one that just passes data to the sound card).
  Other plugins are available but for now we will just use `hw`.
- `IQaudIODAC` is the name that the sound card declares to **ALSA**, possibly with a suffix like
  `_2`   assigned by **ALSA** if there is more than one of the same type.
  "IQaudIODAC" is the name  declared the [IQaudio DAC+ HAT](
  https://shop.pimoroni.com/products/pi-dac).
- `0` is the index number of the output device on the sound card; most sound cards have the
  playback device as 0.

**ALSA** refers to `hw:IQaudIODAC,0` as a **PCM**, which refers to Pulse Code Modulation, but is
used more generally by **ALSA** to refer to the combination of a plugin, a card and a device that
will perform digital audio processing.

The traditional amplifier and passive speakers can be replaced by active speakers with an amplifier
built into one of the pair:
```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Raspberry Pi ("Wallace")                        ┃
┃ ┌─────────┐                                     ┃  
┃ │ spotify │   ┌─────────────┐   ┌────────────┐  ┃      ┏━━━━━━━━━━━━━━━━━┓
┃ │   or    ├───┤ ALSA device ├───┤ Sound card ├──╂──────┨ Active speakers ┃
┃ │ mopidy  │   └─────────────┘   └────────────┘  ┃      ┗━━━━━━━━━━━━━━━━━┛
┃ └─────────┘                                     ┃ 
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```
*Figure 2*

It is also possible to have an amplifier built into the sound card, for example
```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Raspberry Pi ("Wallace")        ┌──────────────────────────────┐  ┃  
┃ ┌─────────┐                     │ Integrated DAC & Amplfier    │  ┃  
┃ │ spotify │   ┌─────────────┐   │ ┌────────────┐ ┌───────────┐ │  ┃      ┏━━━━━━━━━━┓
┃ │   or    ├───┤ ALSA device ├───┼─┤ Sound card ├─┤ Amplifier ├─┼──╂──────┨ Speakers ┃
┃ │ mopidy  │   └─────────────┘   │ └────────────┘ └───────────┘ │  ┃      ┗━━━━━━━━━━┛
┃ └─────────┘                     └──────────────────────────────┘  ┃ 
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```
*Figure 3*

For example, the IQaudio DAC+ mentioned under the first diagram could be replaced by the
[IQaudio DigiAMP+](https://shop.pimoroni.com/products/pi-digiamp), which needs an external power
supply insteead of the Raspberry Pi's USB supply to power the Pi, the soundcard and the speakers.
Despite changing the hardware model, this still appears to **ALSA** as an `IQaudIODAC` card.

## Simple Room Setup

In other words, as far as **ALSA** is concerned (and, therefore, **Spotify** or **Mopidy**), the
HiFi equipment isn't significant and the important part of the Figures 1-3 in the previous section
is simply:
```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Raspberry Pi ("Wallace")          ┃
┃ ┌─────────┐                       ┃
┃ │ spotify │   ┌─────────────────┐ ┃
┃ │   or    ├───┤ hw:IQaudIODAC,0 │ ┃
┃ │ mopidy  │   └─────────────────┘ ┃
┃ └─────────┘                       ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```
*Figure 4*

Obviously, your sound card is likely to have a different **ALSA** card name, particularly given that
my IQaudIO cards are discontinued (don't worry, they have been replaced by equivalents sold under
the RaspberryPi brand).
However, Aardsound will automatically detect the first non-HDMI, non-Headphone (and non-Loopback)
sound card, so you don't need to know the name of your sound card or configure it in **Ansible**.

The **Ansible** inventory needed to deploy **Spotify** with the default settings is as simple as:
```YAML
aardsound:
  hosts:
    wallace:
  vars:
    aardsound_spotify: true
```
The **Mopidy** equivalent configuration is
```YAML
aardsound:
  hosts:
    wallace:
  vars:
    aardsound_mopidy: true
```
However, configuration of **Mopidy** requires the definition of a **Mopidy Local** data store
and/or some **Mopidy** extensions.
The **Ansible**  inventory variables needed to perform this additional **Mopidy** configuration
are given in the [main README](./README.md).
The same will apply, naturally, to the **Mopidy** configurations in the other examples that follow.


## **Spotify** and **Mopidy** together
If we decided to replace "Spotify or Mopidy" in the diagram with "Spotify and Mopidy" then one of 
the two wouldn't work.
One would succeed in connecting to the `hw:IQAudIODAC,0` PCM and would prevent the other from
accessing it; to solve this **ALSA** provides the direct mixer (**dmix**) plugin.
To use **dmix** we need to define a PCM that uses it (actually we also need to define a second 
PCM of type `dmix` to connect to the output PCM, but we'll ignore that detail: it's handled
quietly in the `dmixer` **Ansible** role).

Both **Spotify** and **Mopidy** are configured to use the `dmix` plugin PCM as their **ALSA**
playback device.
The PCM that Aardsound creates for this is called `aardmixer` (by default).
Now we have:
```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Raspberry Pi ("Wallace")                                   ┃
┃ ┌─────────┐                                                ┃
┃ │ spotify ├───┐                                            ┃
┃ └─────────┘   │   ┌────────────────┐   ┌─────────────────┐ ┃
┃               ├───│ plug:aardmixer ├───┤ hw:IQaudIODAC,0 │ ┃
┃ ┌─────────┐   │   └────────────────┘   └─────────────────┘ ┃
┃ │ mopidy  ├───┘                                            ┃
┃ └─────────┘                                                ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```
*Figure 5*
If more than one audio source is configured, Aaardsound automatically adds the `aardmixer` PCM
into the configuration.
All we need to configure in **Ansible** is:
```YAML
aardsound:
  hosts:
    wallace:
  vars:
    aardsound_spotify: true
    aardsound_mopidy: true
```

## Multi-room Audio
Let's play **Mopidy** across two (or more) rooms (there is a subtlety with **Spotify**, which we
will come to soon).
To do this, we introduce[Snapcast](https://github.com/snapcast/snapcast), which consists of a
server, and a streaming client: `snapclient` and `snapserver`.
There are also separate control clients (e.g., `snapdroid`) that control the volume level of each
streaming client.
Aardsound does *not* deploy control clients.
We'll use 'client' to refer to a 'streaming client' for brevity.

The client connects to the server and sends whatever the server from its input source to the client's playback device, with a volume level set by a control client.
We need to send the output from **Mopidy** to the input stream of the **Snapcast** server.  A common
stream supported by both is a *FIFO* -(First In/First Out): a special type of UNIX (and therefore
Linux) file that copies its input to its output.
So we can use Aardsound to set up:
```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓     ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Raspberry Pi ("Wallace")           ┃     ┃ Raspberry Pi ("Gromit")           ┃
┃ ┌────────┐ ┌──────┐ ┌────────────┐ ┃     ┃ ┌────────────┐ ┌────────────────┐ ┃
┃ │ mopidy ├─┤ FIFO ├─┤ snapserver ├─╂──┬──╂─│ snapclient ├─┤ hw:IQaudioDC,0 │ ┃
┃ └────────┘ └──────┘ └────────────┘ ┃  │  ┃ └────────────┘ └────────────────┘ ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛  │  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
                                        │  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
                                        │  ┃ Raspberry Pi ("Feathers")         ┃
                                        │  ┃ ┌────────────┐ ┌────────────────┐ ┃
                                        └──╂─│ snapclient ├─┤ hw:IQaudioDC,0 │ ┃
                                           ┃ └────────────┘ └────────────────┘ ┃
                                           ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```
*Figure 6*

We can have as many Snapclients as we have rooms (there may be an upper limit but my house isn't
big enough to test it).
The **Ansible** inventory needed for this is a little more complex, because now we have three
hosts doing two different things.
We can add a little simplification with an **Ansible** group for `Gromit` and `Feathers`.
We will keep the "aard" prefix for the group name to avoid clashes with an existing **Ansible**
groups.
```YAML
aardsound:
  hosts:
    wallace:
      vars:
        aardsound_mopidy_multiroom: true
  children:
    aardclient:
aardclient:
  hosts:
    gromit:
    feathers:
  vars:
    aardsound_snapclient: true
    snapclient_of:
    - name: Wallace Mopidy
      host: wallace
      port: 11704
```
`aardclient` must be a child of `aardsound` or the `aardsound.yml` playbook will ignore it and any
hosts that are not defined in the `aardsound` group.
There are three new variables
- `aardsound_mopidy_multiroom` has replaced `aardsound_multiroom` but now it applies specifically
  to `wallace` not to the whole `aardsound` group
- `aardsound_snapclient` applies to the `aardclient` group and therefore to `gromit` and `feathers`;
  it declares that both should run `snapclient`
- `snapclient_of` tells the hosts in the `aardclient` group where to find the **Snapcast** server:
  - `name` is an optional name that is used to say what this **Snapcast** client is connecting to
  - `host` is the name of the host running `snapserver`
  - `port` is set to `11704` instead of the default `1704` used by **Snapcast**.
    This is because, by default, **Aardsound** uses `1704` for **Spotify** multi-room and
    `11704` for **Mopidy** multi-room.
    
  If you are familiar with YAML, you may notice that `snapclient_of` is a list (with one entry) of
  dictionaries.  The reason will become clear in later examples.

Setting `aardsound_mopidy_multiroom` to `true` creates the instance of `mopidy` on `wallace` and
configures it to output to the FIFO.
It also creates the FIFO and configures the instance of `snapserver` to consume the content of the
FIFO.

You may wonder "why does `snapclient_of` not begin with `aardsound_`?"  The reason is as follows:
- Aaardsound is implemented as a set of 12 [**Ansible** roles](
  https://docs.ansible.com/projects/ansible/latest/playbook_guide/playbooks_reuse_roles.html)
- The main role is called `aardsound`; it uses the other roles as needed
- Good **Ansible** practice is for variables used by roles to begin with the role name and an
  underscore, i.e., `aardsound_` for variables used by the `aardsound` role.
- Variables like `aardsound_snapclient: true` declare that the `snapclient` role needs to be
  included by `aardsound`
- `snapclient_of` is only used by the `snapclient` role: it doesn't need the `aardsound_` prefix

Note that there are other variables that use the `aardsound_` prefix to simplify setting variables
for multiple roles.
For example, `aardsound_location` sets the value of both `spotify_location` and `mopidy_location`;
this allows you to configure the room that the Raspberry Pi is in for both **Spotify** and
**Mopidy** in one variable.

The **Snapcast** server can run on a Debian VM if that's easier for you, or you can do effectively
the same setup with just two Raspberry Pis by making one a client of itself.
```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓     ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Raspberry Pi ("Wallace")           ┃     ┃ Raspberry Pi ("Gromit")           ┃
┃ ┌────────┐ ┌──────┐ ┌────────────┐ ┃     ┃ ┌────────────┐ ┌────────────────┐ ┃
┃ │ mopidy ├─┤ FIFO ├─┤ snapserver ├─╂─────╂─│ snapclient ├─┤ hw:IQaudioDC,0 │ ┃
┃ └────────┘ └──────┘ └─────┬──────┘ ┃     ┃ └────────────┘ └────────────────┘ ┃
┃       ┌───────────────────┘        ┃     ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
┃ ┌─────┴──────┐  ┌────────────────┐ ┃
┃ │ snapclient ├──┤ hw:IQaudioDC,0 │ ┃
┃ └────────────┘  └────────────────┘ ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```
*Figure 7*

This has a slightly simpler inventory: we no longer need the `aardclient` group.
```YAML
aardsound:
  hosts:
    wallace:
      vars:
        aardsound_mopidy_multiroom: true
    gromit:
  vars:
    aardsound_snapclient: true
    snapclient_of:
    - name: Wallace Mopidy
      host: wallace
      port: 11704
```

### Multi-room **Spotify**
There is a small technical difference with Spotify: although it used to work with a FIFO backend,
the behaviour has changed.
**Spotify** Connect now uses the positioning of the playback device to determine whether to keep
sending data and **Snapcast** doesn't support this.
This results in **Spotify Connect** skipping to the next track as soon as it attempts playback.
The [solution](https://gist.github.com/itskevinb/59630c0a57148ab63cb6325ae6e26da9) to this problem
(thanks to [itskevinb](https://gist.github.com/itskevinb)) is to insert something in front of the
FIFO used by the **Snapcast** server that will provide this information to **Spotify**.
This can be done by inserting an FFMPEG program whos role is to "bridge" the **Spotify Connect**
output to the **Snapcast** FIFO.
In order for **Spotify** to output to FFMPEG, which can listen to **ALSA** devices but not to the
`librespot` program that supports **Spotify Connect**, it is necessary to insert an **ALSA**
loopback device that copies its input to its output.
This uses the **ALSA** `snd-aloop` module.

The **Spotify** versions of the two diagrams above, look like this:
```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓     ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Raspberry Pi ("Wallace")               ┃     ┃ Raspberry Pi ("Gromit")           ┃
┃ ┌─────────┐    ┌──────┐ ┌────────────┐ ┃     ┃ ┌────────────┐ ┌────────────────┐ ┃
┃ │ spotify │    │ FIFO ├─┤ snapserver ├─╂──┬──╂─│ snapclient ├─┤ hw:IQaudioDC,0 │ ┃
┃ └────┬────┘    └──┬───┘ └────────────┘ ┃  │  ┃ └────────────┘ └────────────────┘ ┃
┃ ┌────┴──────┐ ┌───┴────┐               ┃  │  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
┃ │ snd-aloop ├─┤ FFMPEG │               ┃  │  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ └───────────┘ └────────┘               ┃  │  ┃ Raspberry Pi ("Feathers")         ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛  │  ┃ ┌────────────┐ ┌────────────────┐ ┃
                                            └──╂─│ snapclient ├─┤ hw:IQaudioDC,0 │ ┃
                                               ┃ └────────────┘ └────────────────┘ ┃
                                               ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```
*Figure 8*

The inventory for this is similar to the one for **Mopidy** in Figure 6, but we no longer need to
declare a port on the `snapclient_of` variable, because multi-room **Spotify** uses the default
value of `1704`.
```YAML
aardsound:
  hosts:
    wallace:
      vars:
        aardsound_spotify_multiroom: true
  children:
    aardclient:
aardclient:
  hosts:
    gromit:
    feathers:
  vars:
    aardsound_snapclient: true
    snapclient_of:
    - name: Wallace Spotify
      host: wallace
```
Setting `aardsound_spotpidy_multiroom` to `true` creates the instance of `Spotify` on `wallace` and
configures it to output to the **ALSA** loopback PCM.
It also creates the loopback PCM and the FFMPEG instance that connects to the output of the loopback
and the FIFO that receives the output from FFMPEG and it configures the instance of `snapserver` to
consume the content of the FIFO.

Unsurprisingly, the version where Wallace is a server and client looks like this
```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓     ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Raspberry Pi ("Wallace")               ┃     ┃ Raspberry Pi ("Gromit")           ┃
┃ ┌─────────┐    ┌──────┐ ┌────────────┐ ┃     ┃ ┌────────────┐ ┌────────────────┐ ┃
┃ │ spotify │    │ FIFO ├─┤ snapserver ├─╂──┬──╂─│ snapclient ├─┤ hw:IQaudioDC,0 │ ┃
┃ └────┬────┘    └──┬───┘ └─────┬──────┘ ┃  │  ┃ └────────────┘ └────────────────┘ ┃
┃ ┌────┴──────┐ ┌───┴────┐      │        ┃  │  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
┃ │ snd-aloop ├─┤ FFMPEG │      │        ┃  │  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ └───────────┘ └────────┘      │        ┃  │  ┃ Raspberry Pi ("Feathers")         ┃
┃       ┌───────────────────────┘        ┃  │  ┃ ┌────────────┐ ┌────────────────┐ ┃
┃ ┌─────┴──────┐      ┌────────────────┐ ┃  └──╂─│ snapclient ├─┤ hw:IQaudioDC,0 │ ┃
┃ │ snapclient ├──────┤ hw:IQaudioDC,0 │ ┃     ┃ └────────────┘ └────────────────┘ ┃
┃ └────────────┘      └────────────────┘ ┃     ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```
*Figure 9*
```YAML
aardsound:
  hosts:
    wallace:
      vars:
        aardsound_spotify_multiroom: true
    gromit:
  vars:
    aardsound_snapclient: true
    snapclient_of:
    - name: Wallace Spotify
      host: wallace
```

## Multi-room or Single-room Audio
How do we handle the case where we might want multi-room audio sometimes, but several separate
single-room instances of **Mopidy** at other times.

### **Mopidy**
This uses different instances of **Mopidy** for the single- and multi-room cases: one **Mopidy** for
each HiFi-connected Raspberry Pi and one additional one for multi-room.
You can even mix the two of them, if you turn the multi-room volume down to zero (using a Snapserver
control client) in rooms where the single-room **Mopidy** is playing.
We need to reintroduce `dmix` for this to work
```
                                           ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
                                           ┃ Raspberry Pi ("Gromit")               ┃
                                           ┃ ┌────────┐ ┌────────────────┐         ┃
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓     ┃ │ mopidy ├─┤ plug:aardmixer │         ┃
┃ Raspberry Pi ("Wallace")           ┃     ┃ └────────┘ └───┬─┬──────────┘         ┃
┃ ┌────────┐ ┌──────┐ ┌────────────┐ ┃     ┃ ┌────────────┐ │ │ ┌────────────────┐ ┃
┃ │ mopidy ├─┤ FIFO ├─┤ snapserver ├─╂──┬──╂─│ snapclient ├─┘ └─┤ hw:IQaudioDC,0 │ ┃
┃ └────────┘ └──────┘ └────────────┘ ┃  │  ┃ └────────────┘     └────────────────┘ ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛  │  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
                                        │  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
                                        │  ┃ Raspberry Pi ("Feathers")             ┃
                                        │  ┃ ┌────────┐ ┌────────────────┐         ┃
                                        │  ┃ │ mopidy ├─┤ plug:aardmixer │         ┃
                                        │  ┃ └────────┘ └───┬─┬──────────┘         ┃
                                        │  ┃ ┌────────────┐ │ │ ┌────────────────┐ ┃
                                        └──╂─│ snapclient ├─┘ └─┤ hw:IQaudioDC,0 │ ┃
                                           ┃ └────────────┘     └────────────────┘ ┃
                                           ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```
*Figure 10*

The inventory for this is similar to that for Figure 6 but also has `aardsound_mopidy: true` which
adds the local **Mopidy** instance to the hosts `aardclient` child group.
Because there are now two audio sources for those two hosts, Aaardsound creates the `aardmixer` PCM
to allow both of them to access the sound card.
```YAML
aardsound:
  hosts:
    wallace:
      vars:
        aardsound_mopidy_multiroom: true
  children:
    aardclient:
aardclient:
  hosts:
    gromit:
      vars:
        aardsound_location: Lounge
    feathers:
      vars:
        aardsound_location: Kitchen
  vars:
    aardsound_mopidy: true
    aardsound_snapclient: true
    snapclient_of:
    - name: Wallace Mopidy
      host: wallace
      port: 11704
```

This inventory file introduces a new variable `aardsound_location`.
We now have three instances in total, but which is which and more importantly, where are they.
The location is included in the default **Mopidy** zeroconf description of the service so that
clients that find the three services can distinguish the location of each.
A different variable is used for the multi-room instance: `aardsound_multiroom_name`;
it has a default value of `Multiroom`, so there is no need to change that in the inventory.

Of course, it is possible to combine the **Snapcast** client and server, like this:
```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓     ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Raspberry Pi ("Wallace")             ┃     ┃ Raspberry Pi ("Gromit")              ┃
┃ ┌────────┐  ┌──────┐  ┌────────────┐ ┃     ┃ ┌────────────┐ ┌────────────────┐    ┃
┃ │ mopidy ├──┤ FIFO ├──┤ snapserver ├─╂─────╂─┤ snapclient ├─┤ plug:aardmixer │    ┃
┃ └────────┘  └──────┘  └─────┬──────┘ ┃     ┃ └────────────┘ └────┬──────┬────┘    ┃
┃       ┌─────────────────────┘        ┃     ┃     ┌───────────────┘      │         ┃
┃ ┌─────┴──────┐ ┌────────────────┐    ┃     ┃ ┌───┴────┐        ┌────────┴───────┐ ┃
┃ │ snapclient ├─┤ plug:aardmixer │    ┃     ┃ │ mopidy │        │ hw:IQaudioDC,0 │ ┃
┃ └────────────┘ └────┬──────┬────┘    ┃     ┃ └────────┘        └────────────────┘ ┃
┃     ┌───────────────┘      │         ┃     ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
┃ ┌───┴────┐        ┌────────┴───────┐ ┃
┃ │ mopidy │        │ hw:IQaudioDC,0 │ ┃
┃ └────────┘        └────────────────┘ ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```
*Figure 11*

Again, just a the `aardsound_location:` and `aardsound_mopidy:` lines are added to the inventory
from Figure 7.
```YAML
aardsound:
  hosts:
    wallace:
      vars:
        aardsound_location: Kitchen
        aardsound_mopidy_multiroom: true
    gromit:
      vars:
        aardsound_location: Lounge
  vars:
    aardsound_mopidy: true
    aardsound_snapclient: true
    snapclient_of:
    - name: Wallace Mopidy
      host: wallace
      port: 11704
```

### **Spotify** Multi-room
The **Spotify** equivalents are more complex because of additional steps in the server pipeline:
```
                                               ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
                                               ┃ Raspberry Pi ("Gromit")               ┃
                                               ┃ ┌─────────┐ ┌────────────────┐        ┃
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓     ┃ │ spotify ├─┤ plug:aardmixer │        ┃
┃ Raspberry Pi ("Wallace")               ┃     ┃ └─────────┘ └──┬─┬───────────┘        ┃
┃ ┌─────────┐    ┌──────┐ ┌────────────┐ ┃     ┃ ┌────────────┐ │ │ ┌────────────────┐ ┃
┃ │ spotify │    │ FIFO ├─┤ snapserver ├─╂──┬──╂─│ snapclient ├─┘ └─┤ hw:IQaudioDC,0 │ ┃
┃ └────┬────┘    └──┬───┘ └────────────┘ ┃  │  ┃ └────────────┘     └────────────────┘ ┃
┃ ┌────┴──────┐ ┌───┴────┐               ┃  │  ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
┃ │ snd-aloop ├─┤ FFMPEG │               ┃  │  ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ └───────────┘ └────────┘               ┃  │  ┃ Raspberry Pi ("Feathers")             ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛  │  ┃ ┌─────────┐ ┌────────────────┐        ┃
                                            │  ┃ │ spotify ├─┤ plug:aardmixer │        ┃
                                            │  ┃ └─────────┘ └──┬─┬───────────┘        ┃
                                            │  ┃ ┌────────────┐ │ │ ┌────────────────┐ ┃
                                            └──╂─│ snapclient ├─┘ └─┤ hw:IQaudioDC,0 │ ┃
                                               ┃ └────────────┘     └────────────────┘ ┃
                                               ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```
*Figure 12*

The extra pipeline complexity in managed by Aaardsound and the inventory is the same as for Figure 8
with one additional line to enable the "local" **Spotify** instance.
```YAML
aardsound:
  hosts:
    wallace:
      vars:
        aardsound_spotify_multiroom: true
  children:
    aardclient:
aardclient:
  hosts:
    gromit:
      vars:
        aardsound_location: Lounge
    feathers:
      vars:
        aardsound_location: Kitchen
  vars:
    aardsound_spotify: true
    aardsound_snapclient: true
    snapclient_of:
    - name: Wallace Spotify
      host: wallace
```
The same applies to the alternative setup without Feathers.
```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓     ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Raspberry Pi ("Wallace")               ┃     ┃ Raspberry Pi ("Gromit")              ┃
┃ ┌─────────┐    ┌──────┐ ┌────────────┐ ┃     ┃ ┌────────────┐ ┌────────────────┐    ┃
┃ │ spotify │    │ FIFO ├─┤ snapserver ├─╂─────╂─│ snapclient ├─┤ plug:aardmixer │    ┃
┃ └────┬────┘    └──┬───┘ └─────┬──────┘ ┃     ┃ └────────────┘ └────┬──────┬────┘    ┃
┃ ┌────┴──────┐ ┌───┴────┐      │        ┃     ┃      ┌──────────────┘      │         ┃
┃ │ snd-aloop ├─┤ FFMPEG │      │        ┃     ┃ ┌────┴────┐       ┌────────┴───────┐ ┃
┃ └───────────┘ └────────┘      │        ┃     ┃ │ spotify │       │ hw:IQaudioDC,0 │ ┃
┃       ┌───────────────────────┘        ┃     ┃ └─────────┘       └────────────────┘ ┃
┃ ┌─────┴──────┐ ┌────────────────┐      ┃     ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
┃ │ snapclient ├─┤ plug:aardmixer │      ┃
┃ └────────────┘ └────┬───────┬───┘      ┃
┃      ┌──────────────┘       │          ┃
┃ ┌────┴────┐         ┌───────┴────────┐ ┃
┃ │ spotify │         │ hw:IQaudioDC,0 │ ┃
┃ └─────────┘         └────────────────┘ ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```
*Figure 13*

```YAML
aardsound:
  hosts:
    wallace:
      vars:
        aardsound_location: Kitchen
        aardsound_spotify_multiroom: true
    gromit:
      vars:
        aardsound_location: Lounge
  vars:
    aardsound_spotify: true
    aardsound_snapclient: true
    snapclient_of:
    - name: Wallace Spotify
      host: wallace
```

### Multi-room **Spotify** and **Mopidy**
Combining multi-room **Spotify** with multi-room requires two instances of **Snapcast** per
Raspberry Pi.
Rather than mix the **Spotify** and **Mopidy** streams on the source side, they are mixed on the
client side.
This is more flexible: for instance, the **Spotify** and **Mopidy** sources could be on different
servers and even in different subnets.
**Spotify** and **Mopidy** sharing a multi-room server with two clients on each of two Raspberry
Pis that can also play either **Spotify** or **Mopidy** local looks like this (for simplicity,
only one of the clients is shown in detail)
```
                                                ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
                                                ┃ Raspberry Pi ("Feathers")             ┃
                                               ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓┃
                                               ┃ Raspberry Pi ("Gromit")               ┃┃
                                               ┃ ┌─────────┐        ┌────────────────┐ ┃┃
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓     ┃ │ spotify ├─────┬──┤ plug:aardmixer │ ┃┃
┃ Raspberry Pi ("Wallace")               ┃     ┃ └─────────┘     │  └────────┬───────┘ ┃┃
┃ ┌─────────┐    ┌──────┐ ┌────────────┐ ┃     ┃ ┌────────────┐  │           │         ┃┃
┃ │ spotify │    │ FIFO ├─┤ snapserver ├─╂─────╂─│ snapclient ├──┤           │         ┃┃
┃ └────┬────┘    └──┬───┘ └────────────┘ ┃     ┃ └────────────┘  │           │         ┃┃
┃ ┌────┴──────┐ ┌───┴────┐               ┃     ┃ ┌────────┐      │  ┌────────┴───────┐ ┃┃
┃ │ snd-aloop ├─┤ FFMPEG │               ┃     ┃ │ mopidy ├──────┤  │ hw:IQaudioDC,0 │ ┃┃
┃ └───────────┘ └────────┘               ┃     ┃ └────────┘      │  └────────────────┘ ┃┃
┃ ┌────────┐     ┌──────┐ ┌────────────┐ ┃     ┃ ┌────────────┐  │                     ┃┃
┃ │ mopidy ├─────┤ FIFO ├─┤ snapserver ├─╂─────╂─│ snapclient ├──┘                     ┃┛
┃ └────────┘     └──────┘ └────────────┘ ┃     ┃ └────────────┘                        ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛     ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```
*Figure 14*

The inventory for this is the inventory from Figures 10 and 12 combined.
This is why the `snapclient_of` variable is a list of dictionaries, because there are two
instances to define on each host.
```YAML
aardsound:
  hosts:
    wallace:
      vars:
        aardsound_mopidy_multiroom: true
        aardsound_spotify_multiroom: true
  children:
    aardclient:
aardclient:
  hosts:
    gromit:
      vars:
        aardsound_location: Lounge
    feathers:
      vars:
        aardsound_location: Kitchen
  vars:
    aardsound_mopidy: true
    aardsound_spotify: true
    aardsound_snapclient: true
    snapclient_of:
    - name: Wallace Mopidy
      host: wallace
      port: 11704
    - name: Wallace Spotify
      host: wallace
```

Finally, it is possibe to run single- and multi-room **Spotify** and **Mopidy** all together to
save a Raspberry Pi.
```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Raspberry Pi ("Wallace")               ┃
┃ ┌─────────┐         ┌────────────────┐ ┃
┃ │ spotify ├──────┐  │ hw:IQaudioDC,0 │ ┃
┃ └─────────┘      │  └───────┬────────┘ ┃
┃ ┌────────┐       │  ┌───────┴────────┐ ┃
┃ │ mopidy ├───────┴──┤ plug:aardmixer │ ┃
┃ └────────┘          └─────────┬──────┘ ┃     ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃              ┌────────────────┤        ┃     ┃ Raspberry Pi ("Gromit")               ┃
┃       ┌──────┴─────┐    ┌─────┴──────┐ ┃     ┃ ┌─────────┐        ┌────────────────┐ ┃
┃       │ snapclient ├─┐  │ snapclient │ ┃     ┃ │ spotify ├─────┬──┤ plug:aardmixer │ ┃
┃       └────────────┘ │  └─────┬──────┘ ┃     ┃ └─────────┘     │  └────────┬───────┘ ┃
┃ ┌────────┐ ┌──────┐  │  ┌─────┴──────┐ ┃     ┃ ┌────────────┐  │           │         ┃
┃ │ mopidy ├─┤ FIFO ├─────┤ snapserver ├─╂─────╂─│ snapclient ├──┤           │         ┃
┃ └────────┘ └──────┘  │  └────────────┘ ┃     ┃ └────────────┘  │           │         ┃
┃                      └────────┐        ┃     ┃                 │           │         ┃
┃ ┌─────────┐    ┌──────┐ ┌─────┴──────┐ ┃     ┃ ┌────────────┐  │  ┌────────┴───────┐ ┃
┃ │ spotify │    │ FIFO ├─┤ snapserver ├─╂─────╂─│ snapclient ├──┤  │ hw:IQaudioDC,0 │ ┃
┃ └────┬────┘    └──┬───┘ └────────────┘ ┃     ┃ └────────────┘  │  └────────────────┘ ┃
┃ ┌────┴─────┐ ┌────┴───┐                ┃     ┃ ┌────────┐      │                     ┃
┃ │ Loopback ├─┤ FFMPEG │                ┃     ┃ │ mopidy ├──────┘                     ┃
┃ └──────────┘ └────────┘                ┃     ┃ └────────┘                            ┃
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛     ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛

```
*Figure 15*

Unsurprisingly, the inventory for this is a combination of those for Figures 11 and 13
```YAML
aardsound:
  hosts:
    wallace:
      vars:
        aardsound_location: Kitchen
        aardsound_mopidy_multiroom: true
        aardsound_spotify_multiroom: true
    gromit:
      vars:
        aardsound_location: Lounge
  vars:
    aardsound_mopidy: true
    aardsound_spotify: true
    aardsound_snapclient: true
    snapclient_of:
    - name: Wallace Mopidy
      host: wallace
      port: 11704
    - name: Wallace Spotify
      host: wallace
```
Rememner that in all of these cases, it is straightforward to add more hosts to any group and to
add additional groups with different variables if you want differing configurations on different
Raspberry Pis.

## My Setup
The audio equipment I use is by no-means "audiophile", but the IQaudIO DACs and their Raspberry
Pi-branded replacements provide excellent sound quality.  The DAC+ is capable of 24-bit 192kHz
digital audio, compared with CD-quality (also used by **Spotify Connect**) 16-bit 44.1kHz.
In the lounge, I have a 30 year-old budget-but-good-quality amplifier and matching speakers.
For the other two rooms, I have budget active speakers that sound fine for background music and, in
the garden, some more expensive exterior passive speakers.

My setup corresponds to Figure 14, so that I don't have to take a room offline to reconfigure
multi-room audio but I use a Debian virual machine as the **Snapcast** server instead of a
Raspberry Pi.
There are are four Raspberry Pi's connected to the **Snapcast** server, all supporting single-
and multi-room **Spotify** and **Mopidy**.
I also have a test setup that can be configured to match any of the diagrams above.

My **Ansible** inventory is more complex than the examples shown above.
- There are child groups under the `aardsound` parent group: `aardserver`, `aardclient`,
  `aardtestserver` and `aardtestclient`
- Inventory variables for each group are in the `/etc/ansible/group_vars/<group>` folder with
  variables for each role in a file named `<role>.yml`.
  For example, all of the `aardsound` role variables for the `aardsound` group are in the file
  `/etc/ansible/group_vars/aardsound/aardsound.yml`, whereas the `spotify` variables for the 
  `aardclient` group are in `/etc/ansible/group_vars/aardlcient/spotify.yml`.


The Raspberry Pi models and the audio equipment they drive are:

| Name | Role | Hardware | HAT | Output to |
| ---- | ---- | -------- | --------- | --------- |
| [Wallace](https://en.wikipedia.org/wiki/List_of_Wallace_%26_Gromit_characters#Wallace) | Kitchen | Pi 4B | IQaudIO DAC+ | Edifier R1000T4 active bookehself speakers
| [Gromit](https://en.wikipedia.org/wiki/List_of_Wallace_%26_Gromit_characters#Gromit) | Lounge | Pi 4B | IQaudIO DAC+ | Denon PMA255UK amp, Mission M70 speakers
| [Feathers](https://en.wikipedia.org/wiki/List_of_Wallace_%26_Gromit_characters#Feathers_McGraw) | Dining Room | Pi 4B | HiFiBerry DAC+ | Edifier R1000T4 active bookehself speakers
| [Shaun](https://en.wikipedia.org/wiki/List_of_Wallace_%26_Gromit_characters#Shaun_the_Sheep) | Garden | Pi 4B | IQAudIO DigiAMP+ | Polk Audio Atrium 4 exterior speakers
| [Wendolene](https://en.wikipedia.org/wiki/List_of_Wallace_%26_Gromit_characters#Wendolene_Ramsbottom) | Server | VM | N/A | N/A
| [Cooker](https://en.wikipedia.org/wiki/List_of_Wallace_%26_Gromit_characters#The_Cooker) | Test | Pi 3B+ | IQaudIO DAC+ | Logitech Z120 active desktop speakers
| [Piella](https://en.wikipedia.org/wiki/List_of_Wallace_%26_Gromit_characters#Piella) | Test | Pi Zero 2W | none | Behringer UCA202 USB DAC, headphones
| [Morph](https://en.wikipedia.org/wiki/Morph_(TV_series)) | Test Server | Pi 3A+ | none | N/A

All of these run RasPiOS 13 "Trixie" or, for Wendolene, Debian 13 "Trixie".
- "HAT" means [Hardware Attached on Top](https://github.com/raspberrypi/hats/tree/master)
- "Pi 4B" means a Raspberry Pi 4 Model B Rev 1.5 with 2GiB of RAM
- "Pi 3B+" means a Raspberry Pi 3 Model B+ Rev 1.3 with 1GiB of RAM
- "Pi 3A+" means a Raspberry Pi 3 Model A+ Rev 1.0 with 512MiB of RAM
- "Pi Zero 2W means a Raspberry Pi Zero 2W with 512MiB of RAM

In each case, the HAT uses RCA audio connections, except for Shaun, which is connected to the
speakers in the garden via two pairs of 25m exterior-grade speaker wires.
Wallace, Cooker and Piella are connected via Wifi; everything else via Ethernet cables.

Both WiFi and Ethernet-connected devices are connected to the same VLAN and subnet;
this is important because **Spotify Connect** relies on ZeroConf, which is normally limited to one
subnet.

I am also running DNS (BIND9) and DHCP (ISC Kea) services on my network, which means that each of
the devices has a fixed hostname and IP address.

