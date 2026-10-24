# Google Drive OCamlFUSE: Install and Automount on Ubuntu

**Applies to:** Ubuntu 22.04 (Jammy) and 24.04 (Noble), `amd64` (Intel/AMD
x86-64).  
**Application:** `google-drive-ocamlfuse` (GDO) **0.9.0**.  
**Goal:** Expose Google Drive at `~/GoogleDrive` and mount it automatically at
user login.

## 1. Download and install the correct `.deb`

Check the Ubuntu release and CPU architecture:

```bash
. /etc/os-release; echo "$VERSION_ID"
dpkg --print-architecture
```

For an **`amd64`** machine, select the appropriate package:

| Ubuntu            | Package                                                      |
| ----------------- | ------------------------------------------------------------ |
| **22.04 (Jammy)** | `google-drive-ocamlfuse_0.9.0-0ubuntu1~bpo22.04.1_amd64.deb` |
| **24.04 (Noble)** | `google-drive-ocamlfuse_0.9.0-0ubuntu1~bpo24.04.1_amd64.deb` |

Download directly from the
[maintainer's Launchpad binary directory](https://ppa.launchpadcontent.net/alessandro-strada/ppa/ubuntu/pool/main/g/google-drive-ocamlfuse/):

```bash
cd ~/Downloads

# Ubuntu 22.04: use this assignment
PKG='google-drive-ocamlfuse_0.9.0-0ubuntu1~bpo22.04.1_amd64.deb'

# Ubuntu 24.04: replace the assignment above with this one instead
# PKG='google-drive-ocamlfuse_0.9.0-0ubuntu1~bpo24.04.1_amd64.deb'

wget "https://ppa.launchpadcontent.net/alessandro-strada/ppa/ubuntu/pool/main/g/google-drive-ocamlfuse/$PKG"
sudo apt install "./$PKG"
```

Verify installation (the package status should begin with `ii`):

```bash
dpkg -l google-drive-ocamlfuse
google-drive-ocamlfuse -version
command -v google-drive-ocamlfuse  # usually /usr/bin/google-drive-ocamlfuse
```

**Important:** Do not install the `bpo24.04.1` package on Ubuntu 22.04;
fundamental system-library versions differ. For ARM hardware, select a matching
`arm64` package instead of `amd64`.

## 2. Authorize Google Drive (once per account)

GDO uses **OAuth 2.0** (Google's delegated-access authorization mechanism). In
[Google Cloud Console](https://console.cloud.google.com/), create/select a
project, enable the **Google Drive API**, configure the OAuth consent screen,
and create an **OAuth client ID** of type **Desktop app**. Obtain its client ID
and client secret. Follow the project's
[authorization instructions](https://github.com/astrada/google-drive-ocamlfuse#usage)
for more detail.

Run as your **normal user**, not with `sudo`:

```bash
google-drive-ocamlfuse -id 'YOUR_CLIENT_ID' -secret 'YOUR_CLIENT_SECRET'
```

Complete Google's browser authorization. GDO maintains its local account
configuration/cache under `~/.gdfuse/default/`. **Keep OAuth credentials and
tokens private.**

## 3. Test a manual mount

```bash
mkdir -p ~/GoogleDrive
google-drive-ocamlfuse ~/GoogleDrive
```

GDO 0.9.0 stays in the foreground. In a **second terminal**, confirm:

```bash
findmnt ~/GoogleDrive
ls ~/GoogleDrive
```

To unmount, stop the foreground command (Ctrl+C) and, if necessary, run:

```bash
fusermount3 -u ~/GoogleDrive
```

## 4. Automount on login using a systemd user service

Create the service file:

```bash
mkdir -p ~/.config/systemd/user
nano ~/.config/systemd/user/google-drive.service
```

Paste:

```ini
[Unit]
Description=Google Drive OCamlFUSE mount

[Service]
Type=exec
ExecStartPre=/usr/bin/mkdir -p %h/GoogleDrive
ExecStart=/usr/bin/google-drive-ocamlfuse %h/GoogleDrive
ExecStop=/usr/bin/fusermount3 -u %h/GoogleDrive
Restart=on-failure
RestartSec=15

[Install]
WantedBy=default.target
```

Enable and start the service **without `sudo`**:

```bash
systemctl --user daemon-reload
systemctl --user enable --now google-drive.service
systemctl --user status google-drive.service
findmnt ~/GoogleDrive
```

The service starts with your user session on subsequent logins. Internet
connectivity is needed to access remote data; if a connection is unavailable at
login, you may need to restart the service once online.

Useful commands:

```bash
systemctl --user restart google-drive.service       # remount
systemctl --user stop google-drive.service          # unmount
journalctl --user -u google-drive.service -e        # view errors
systemctl --user disable --now google-drive.service # remove autostart
```

If the mount is busy when stopping, close any terminals or programs accessing
`~/GoogleDrive` and retry.

## 5. Mounting vs. syncing

- **Mount:** A FUSE filesystem makes remote Google Drive files appear under
  `~/GoogleDrive`. File changes made through the mount are communicated to
  Drive; GDO uses caching, and remote changes may take time to appear.
- **Full offline sync:** A separate _local copy_ of all selected files remains
  available without the internet and is reconciled with Drive later. **GDO is
  not a full offline synchronization tool.** For this, consider
  [rclone](https://rclone.org/drive/) with a separate, deliberately configured
  synchronization workflow. Do not assume the GDO cache is a backup.

**Sources:**
[GDO repository/usage instructions](https://github.com/astrada/google-drive-ocamlfuse)
·
[Launchpad `.deb` directory](https://ppa.launchpadcontent.net/alessandro-strada/ppa/ubuntu/pool/main/g/google-drive-ocamlfuse/)
