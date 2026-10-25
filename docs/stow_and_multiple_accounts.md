# Systemd units as GNU Stow packages (one per account)

The systemd user units live in this repo as [GNU Stow](https://www.gnu.org/software/stow/)
packages, one per Google account. A machine mounts exactly the accounts whose packages you stow,
so "PC A mounts one drive, PC B mounts both" is just a different list of packages.

## Packages

| Package | Unit | Mount point | Notes |
| --- | --- | --- | --- |
| `gdrive-zdm2718` | `gdrive-zdm2718.service` | `~/zdm2718_gdrive` | account label `zdm2718` |
| `gdrive-zmerino2718` | `gdrive-zmerino2718.service` | `~/zmerino2718_gdrive` | account label `zmerino2718` |
| `gdrive-default` | `google-drive.service` | `~/zdm2718` | single-account unit from the [setup guide](google_drive_ocamlfuse_setup.md); don't combine it with the labeled units for the same account |

The labeled units use `%h` (systemd's home directory specifier), so they work for any username.

## Stow them

Run from the repo root and pass `-t ~`. Use **`--no-folding`**: without it, Stow may link the
whole `~/.config/systemd/user` directory into this repo, and `systemctl --user enable` would then
write its `*.wants/` symlinks into the repo.

```bash
cd ~/google-drive-auto-mount-and-sync            # or modules/gdrive inside the dotfiles repo

stow -nv --no-folding -t ~ gdrive-zdm2718        # dry run first
stow -v  --no-folding -t ~ gdrive-zdm2718        # PC A: one account
stow -v  --no-folding -t ~ gdrive-zdm2718 gdrive-zmerino2718   # PC B: both
```

If `~/.config/systemd/user/<unit>.service` already exists as a real file, Stow refuses to
overwrite it. Move it aside and run Stow again.

## Authorize each account, then enable

Authorize every account once **before** enabling its unit (the units use `Restart=always`, so an
unauthorized account would retry in a loop):

```bash
google-drive-ocamlfuse -label zdm2718 -id 'YOUR_CLIENT_ID' -secret 'YOUR_CLIENT_SECRET'
google-drive-ocamlfuse -label zmerino2718 -id 'YOUR_CLIENT_ID' -secret 'YOUR_CLIENT_SECRET'   # PC B only
```

Then:

```bash
systemctl --user daemon-reload
systemctl --user enable --now gdrive-zdm2718.service
systemctl --user enable --now gdrive-zmerino2718.service    # PC B only
findmnt -t fuse | grep gdrive
```

Tokens live in `~/.gdfuse/<label>/` and are not part of this repo.

## Add or remove an account

```bash
# add: copy a package and rename the label
cp -r gdrive-zdm2718 gdrive-NEWLABEL
mv gdrive-NEWLABEL/.config/systemd/user/gdrive-zdm2718.service \
   gdrive-NEWLABEL/.config/systemd/user/gdrive-NEWLABEL.service
sed -i 's/zdm2718/NEWLABEL/g' gdrive-NEWLABEL/.config/systemd/user/gdrive-NEWLABEL.service

# remove from one machine
systemctl --user disable --now gdrive-zmerino2718.service
stow -D -v --no-folding -t ~ gdrive-zmerino2718
```
