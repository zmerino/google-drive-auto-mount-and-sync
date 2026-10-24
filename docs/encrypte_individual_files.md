# Encrypte Credentials

For google-drive-ocamlfuse to mount credentials mount
be validated using OAuth 2.0. These credentials, such as ID and client seceret
might be stored on local disk and should be encrypted.

Use gocryptfs for file level AES-256-GC encryption:

### Install gocryptfs

First build-without-openssl (if desired),
```bash
$ git clone https://github.com/rfjakob/gocryptfs.git
$ cd gocryptfs
$ ./build-without-openssl.bash
```
Then install into user local bin dir,
```bash
sudo install -m 755 gocryptfs /usr/local/bin/gocryptfs
```

### Setup encrypted file and interaction file dirs,
```bash
mkdir -p ~/formetoknow.encrypted
mkdir -p ~/formetoknow

chmod 700 ~/formetoknow.encrypted ~/formetoknow
```

Initialize encrpyted dir,
`gocryptfs - init ~/formetoknow.encrypted`

Unlock encrypted dir,
`gocryptfs ~/formetoknow.encrypted/ ~/formetoknow`

Unmount unencrypted data dir,
`fusermount -u ~/formetoknow`
