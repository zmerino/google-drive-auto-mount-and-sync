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

```bash
google-drive-ocamlfuse \
  -id "YOUR_CLIENT_ID" \
  -secret "YOUR_CLIENT_SECRET"
```

## Repository layout

```
google-drive-auto-mount-and-sync/
├── README.md
├── docs/
│   ├── google_drive_ocamlfuse_setup.md     ← install, OAuth, manual mount, automount
│   ├── stow_and_multiple_accounts.md       ← per-account units with Stow (read this for multi-PC setups)
│   └── encrypte_individual_files.md
├── gdrive-default/                         ← stow package: google-drive.service (single account)
├── gdrive-zdm2718/                         ← stow package: gdrive-zdm2718.service
└── gdrive-zmerino2718/                     ← stow package: gdrive-zmerino2718.service
```

## Choosing accounts per machine

Each account's systemd unit is its own [GNU Stow](https://www.gnu.org/software/stow/) package, so
a machine only mounts the accounts you stow:

```bash
stow -v --no-folding -t ~ gdrive-zdm2718                         # machine with one account
stow -v --no-folding -t ~ gdrive-zdm2718 gdrive-zmerino2718      # machine with both
```

Authorize each account first, then enable its unit. Details:
[docs/stow_and_multiple_accounts.md](docs/stow_and_multiple_accounts.md).

This repo is also a submodule of [zmerino/dotfiles](https://github.com/zmerino/dotfiles)
(`modules/gdrive`), whose `bootstrap.sh` stows the accounts listed in that machine's `machine.conf`.