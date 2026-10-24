# google-drive-auto-mount-and-sync

This repo is an installation and configuration guide for automatically mounting
and syncing a google drive once connected to internet via
google-drive-ocamlfuse on ubuntu like linux OSs. I'm assuming ubuntu 22.04,
24.04, and above.

For the full walkthrough (OAuth setup, manual mount test, systemd automount,
mount vs. sync), see
[docs/google_drive_ocamlfuse_setup.md](docs/google_drive_ocamlfuse_setup.md).

## Install google-drive-ocamlfuse

### Option 1: install with `apt`

If the maintainer's PPA works on your system, add it and install:

```bash
sudo add-apt-repository ppa:alessandro-strada/ppa
sudo apt update
sudo apt install google-drive-ocamlfuse
```

### Option 2: download the `.deb` (Ubuntu 22.04, `amd64`)

If `apt install` isn't an option, download the 22.04 package from the
[maintainer's Launchpad directory](https://ppa.launchpadcontent.net/alessandro-strada/ppa/ubuntu/pool/main/g/google-drive-ocamlfuse/)
and install it:

```bash
cd ~/Downloads
PKG='google-drive-ocamlfuse_0.9.0-0ubuntu1~bpo22.04.1_amd64.deb'
wget "https://ppa.launchpadcontent.net/alessandro-strada/ppa/ubuntu/pool/main/g/google-drive-ocamlfuse/$PKG"
sudo apt install "./$PKG"
```

Use the `bpo22.04` package only on 22.04 (not a `bpo24.04` one), and pick an
`arm64` package instead of `amd64` on ARM hardware.

### Verify

```bash
google-drive-ocamlfuse -version
```

Then continue with authorization and automount in
[the setup guide](docs/google_drive_ocamlfuse_setup.md#2-authorize-google-drive-once-per-account).

## Configure GDO credentials

google-drive-ocamlfuse \
  -id "YOUR_CLIENT_ID" \
  -secret "YOUR_CLIENT_SECRET"