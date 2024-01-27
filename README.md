# omadapi

Omada on Raspberry Pi

## Whats this?
A fork of [pi-gen](https://github.com/RPi-Distro/pi-gen/) to build a custom image for Omada on raspberry pi (~4)

Features:
* [MongoDB 4.4.26](https://github.com/GeoffWilliams/mongodb-raspberrypi-binaries/releases/tag/v4.4.26)
* OpenJDK 17
* jsvc 1.3.4
* Plug and play - just burn the image ssh in on ethernet (look for IP address on router)
* Omada is already installed and set to start automatically
* Access via ssh

## Recommended hardware

* Raspberry Pi 4 Model B 1GB edition
* If you have more memory adjust `-Xmx` in `/opt/tplink/EAPController/bin/control.sh`. Dont forget to leave memory for the system!

# After installation

You MUST login to Raspberry Pi via ssh and change the password on all newly flashed SD cards (`passwd` command):

* Username: `omada`
* Password: `omada`
* Hostname `omadapi` - Your router may register this if you run something good like OpenWrt otherwise check what IP address router allocated.


## Omada
When booted, Omada will be available at:
* http: [http://omadapi:8088](http://omadapi:8088) (redirects to TLS port)
* https: [https://omadapi:8043](https://omadapi:8043)

Where `omadapi` is the hostname or IP address of the pi. TLS certificate is self-signed so you have to click-through the browser security warning.

You will be prompted to setup a user.

## Start/stop Omada

```shell
/etc/init.d/tpeap start
/etc/init.d/tpeap stop
```

## Logs?

In `/opt/tplink/EAPController/logs/`

## Start/stop mongodb

Dont - its controlled automatically by Omada

## Upgrades

### Option 1 (swap)

Backup settings:

settings -> maintenance -> backup -> click export, a file will be prepared and then it downloads.

Shut down the pi, flash a new image on an additional SD card and then restore from backup when the new image boots. If there are problems just swap back to the old SD card.


### Option 2 (in-place)

Follow the vendor instructions to update the Omada debian package. Omadapi is just a regular Linux system so vendor upgrade path should work. After upgrade, these changes are required to re-apply `omadapi` settings:

**Post upgrade steps**

* Replace `/opt/tplink/EAPController/bin/control.sh` with contents of https://github.com/GeoffWilliams/omadapi/blob/omadapi/stageomada/10-omada/files/control.sh (and check yourself for any inc
* Ensure vendored `mongod` is a symlink to system `mongod`:

```shell
mv /opt/tplink/EAPController/bin/mongod /opt/tplink/EAPController/bin/mongod.orig
ln -s /usr/local/bin/mongod /opt/tplink/EAPController/bin/mongod`
```

...Reboot.

This process is riskier since your operating on a running device... if upgrade breaks for some reason now you have degraded network _and_ a broken controller. Make sure you have a backup before starting.

## Testing

What testing have you done?

* Boot to login screen
* Login, add 2 access points
* Perform backup
* Restore backup
* Mesh wifi with 2 access points
* Firmware update access points
* 33 hour+ uptime (rebooted after loss of internet connectivity to get device firmware update notification)

## Building the image

`omdapi`` is A fork of [pi-gen](https://github.com/RPi-Distro/pi-gen/) to build a custom image for omada on raspberry pi (~4) so can be updated for newer Raspbery Pi OS releases by rebasing.

To build the image yourself:

1. Read the [pi-gen docs](./docs/pi-gen.md) to setup your build environment
2. Clone the repo
3. Switch to branch `omada`
4. Adjust (or disable...) proxy configuration in `config`. It seems necessary to build with an apt proxy to prevent timeouts
4. Run `build-docker.sh`
5. Burn the `full` image that the script generates with [Balena Etcher](https://etcher.balena.io/) or similar, then put SD card in pi and power on

## Status

* Larger deployments untested, please report successes/failures
* From tp-link? Please feel free to make some raspbery pi image for the community based on this!
* Interested to help? Please open a ticket...

## Acknowledgements
* Lots of good infos on the [Omada Raspbery Pi forum thread](https://community.tp-link.com/en/business/forum/topic/528450)
* `themattman` for providing a [guide to setting up old versions of MongoDB on Raspberry Pi](https://github.com/themattman/mongodb-raspberrypi-binaries)
* [pi-gen](https://github.com/RPi-Distro/pi-gen/) - entire build system