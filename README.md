# Packer Plugin systemd-nspawn
This plugin can build and provision systemd-nspawn containers. It can import or
clone an existing base image, or generate one from scratch using debootstrap.

## Installation

### Using pre-built releases

#### Using the `packer init` command

Starting from version 1.7, Packer supports a new `packer init` command allowing
automatic installation of Packer plugins. Read the
[Packer documentation](https://www.packer.io/docs/commands/init) for more information.

To install this plugin, copy and paste this code into your Packer configuration .
Then, run [`packer init`](https://www.packer.io/docs/commands/init).

```hcl
packer {
  required_plugins {
    nspawn = {
      version = ">= 1.2.4"
      source  = "github.com/emansom/nspawn"
    }
  }
}
```


#### Manual installation

You can find pre-built binary releases of the plugin [here](https://github.com/emansom/packer-plugin-nspawn/releases).
Once you have downloaded the latest archive corresponding to your target OS,
uncompress it to retrieve the plugin binary file corresponding to your platform.
To install the plugin, please follow the Packer documentation on
[installing a plugin](https://www.packer.io/docs/extending/plugins/#installing-plugins).


### From Sources

If you prefer to build the plugin from sources, clone the GitHub repository
locally and run the command `go build` from the root
directory. Upon successful compilation, a `packer-plugin-ansible` plugin
binary file can be found in the root directory.
To install the compiled plugin, please follow the official Packer documentation
on [installing a plugin](https://www.packer.io/docs/extending/plugins/#installing-plugins).


### Configuration

For more information on how to configure the plugin, please read the
documentation located in the [`docs/`](docs) directory.


## Contributing

* If you think you've found a bug in the code or you have a question regarding
  the usage of this software, please reach out to us by opening an issue in
  this GitHub repository.
* Contributions to this project are welcome: if you want to add a feature or a
  fix a bug, please do so by opening a Pull Request in this GitHub repository.
  In case of feature contribution, we kindly ask you to open an issue to
  discuss it beforehand.

## Setup

Prerequisites:
- `debootstrap` to generate a minimal viable chroot image
- `golang-go` to build this plugin from source
- `libglib2.0-bin` to monitor container status with `gdbus`
- `packer` (the Debian package recommends docker, you don't need that)
- `systemd-container` systemd-nspawn and related tools
- (optional) `packer-provisioner-apt` for installing deb packages
- (optional) `zstd` for creating and importing .tar.zst images

In most cases, you'll want your container to be able to connect to the network
to provision itself, for that you need to enable systemd-networkd and
systemd-resolved:

```
systemctl enable systemd-networkd.service
systemctl start systemd-networkd.service
systemctl enable systemd-resolved.service
systemctl start systemd-resolved.service
```

The recommended way to install deb packages when provisioning images is
[packer-provisioner-apt](https://git.sr.ht/~angdraug/packer-provisioner-apt).
When installing that plugin from source, symlink it into your working directory
so that Packer can find it:

```
cd ..
git clone https://git.sr.ht/~angdraug/packer-provisioner-apt
cd packer-provisioner-apt
go build
ln -s $(pwd)/packer-provisioner-apt ../packer-builder-nspawn
```

The included example `nspawn.pkr.hcl` uses zstd to compress the image tarball,
it is several times faster than gzip at the same or better compression ratio.
You don't need zstd if you use a different method for archiving and delivering
your images.

For compatibility with the Debian package of Packer that is built with a newer
version of [ugorji-go-codec](https://github.com/ugorji/go) than the one pinned
in Packer source, this plugin's `go.mod` includes a replace line to import a
similarly patched version of Packer source. To use this plugin with Packer
built from unpatched upstream source, comment out that replace line.

## Security

In Debian, unprivileged user namespaces are disabled by default and have to be
enabled for nspawn's `-U` option to have any effect:

```
echo kernel.unprivileged_userns_clone=1 > /etc/sysctl.d/nspawn.conf
systemctl restart systemd-sysctl.service
```

See discussion of this kernel feature and its security implications in
[Debian Wiki](https://wiki.debian.org/nspawn#Host_Preparation) and
[Linux Weekly News](https://lwn.net/Articles/673597/).

This builder will work with and without private user namespaces. If you want an
image built on a system with userns enabled to be usable on systems with userns
disabled, use the method offered in
[systemd-nspawn(1)](https://www.freedesktop.org/software/systemd/man/systemd-nspawn.html#-U)
to reset chroot file ownership before archiving the image:

```
systemd-nspawn ... --private-users=0 --private-users-chown
```

## Copying

Copyright (c) 2020  Dmitry Borodaenko <angdraug@debian.org>

This program is free software. You can distribute/modify this program under
the terms of the GNU General Public License version 3 or later, or under
the terms of the Mozilla Public License, v. 2.0, at your discretion.

This Source Code Form is subject to the terms of the Mozilla Public
License, v. 2.0. If a copy of the MPL was not distributed with this
file, You can obtain one at http://mozilla.org/MPL/2.0/.

This Source Code Form is not "Incompatible With Secondary Licenses",
as defined by the Mozilla Public License, v. 2.0.
