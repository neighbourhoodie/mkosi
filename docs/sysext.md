# Building system extensions with mkosi

[System extension](https://uapi-group.org/specifications/specs/extension_image/)
images may – dynamically at runtime — extend the base system with an
overlay containing additional files.

To build system extensions with mkosi, we first have to create a base
image on top of which we can build our extension.

To keep things manageable, we'll use mkosi's support for building
multiple images so that we can build our base image and system extension
in one go.

Start by creating a temporary directory with a base configuration file
`mkosi.conf` with some shared settings:

```conf
[Output]
OutputDirectory=mkosi.output
CacheDirectory=mkosi.cache
```

From now on we'll assume all steps are executed inside the temporary
directory.

Now let's continue with the base image definition by writing the
following to `mkosi.images/base/mkosi.conf`:

```conf
[Output]
Format=directory

[Content]
CleanPackageMetadata=no
Packages=systemd
         udev
```

We use the `directory` output format here instead of the `disk` output
so that we can build our extension without needing root privileges.

Now that we have our base image, we can define a sysext that builds on
top of it by writing the following to `mkosi.images/btrfs/mkosi.conf`:

```conf
[Config]
Dependencies=base

[Output]
Format=sysext
Overlay=yes

[Content]
BaseTrees=%O/base
Packages=btrfs-progs
```

`BaseTrees=` point to our base image and `Overlay=yes` instructs mkosi
to only package the files added on top of the base tree.

We can't sign the extension image without a key, so let's generate one
with `mkosi genkey` (or write your own private key and certificate
yourself to `mkosi.key` and `mkosi.crt` respectively). Note that this
key will need to be loaded into your kernel keyring either at build time
or via MOK for systemd to accept the system extension at runtime as
trusted.

Finally, you can build the base image and the extensions by running
`mkosi -f`. You'll find `btrfs.raw` in `mkosi.output` which is the
extension image.

If you want to package up the base image into another format, for
example an initrd, we can do that by adding the following to
`mkosi.images/initrd/mkosi.conf`:

```conf
[Config]
Dependencies=base

[Output]
Format=cpio

[Content]
MakeInitrd=yes
BaseTrees=%O/base
```

If we now run `mkosi -f` again, we'll find `initrd.cpio.zst` in
`mkosi.output` with its accompanying extension still in `btrfs.raw`.
