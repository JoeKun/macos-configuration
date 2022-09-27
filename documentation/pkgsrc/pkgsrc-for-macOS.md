# Common packages and configuration for macOS

## Why not use Homebrew?

Although the hegemonic package manager for macOS is unequivocally Homebrew, and despite that project having been very exciting in the early days, it may not be well suited for everyone.

For example, if you need to dedicate `/usr/local` to something other than Homebrew, while it's supposedly supported to install Homebrew in a different location, in reality, all bets are off at that point, and it may be a nightmare to set this up and maintain in the long run. This is all the more true that now, Homebrew really pushes installing packages with bottles, which are pre-compiled and only meant to work well when installed in `/usr/local`.

For those reasons, you might think you just need to use Homebrew in a mode where all packages are compiled locally for the chosen non-standard prefix; but even that is very difficult now that the environment variable `HOMEBREW_BUILD_FROM_SOURCE` is not supported anymore; and the alternative of passing `--build-from-source` to invoke certain `brew` commands is not satisfactory either, since using it will only ensure that the requested package is built from source, while all of its dependencies will likely be installed from bottles.

Beyond that, having packages managed as an unprivileged user never sat well with me.

Some of those reasons are also explained in [this blog post](https://derflounder.wordpress.com/2015/04/23/installing-joyents-pkgsrc-package-manager-on-os-x/).


## Install `pkgsrc`

We'll be using the precompiled `pkgsrc` and `pkgin` distribution maintained by Joyent.

See [this page](https://pkgsrc.joyent.com/install-on-osx/) from Joyent's `pkgsrc` website to see up-to-date and complete installation instructions.

Here's the simpler set of steps we can follow, where we simply skip checking the GPG signature.

```
$ BOOTSTRAP_TAR="bootstrap-macos11-trunk-x86_64-20211207.tar.gz"
$ BOOTSTRAP_SHA="07e323065708223bbac225d556b6aa5921711e0a"
$ curl -O https://pkgsrc.joyent.com/packages/Darwin/bootstrap/${BOOTSTRAP_TAR}
$ echo "${BOOTSTRAP_SHA}  ${BOOTSTRAP_TAR}" > check-shasum
$ shasum -c check-shasum
$ rm -f check-shasum
$ sudo tar -zxpf ${BOOTSTRAP_TAR} -C /
```

Close your Terminal window and open a new one. Then run the following commands:

```
$ sudo -s
# pkgin update && pkgin full-upgrade
# pkgin update && pkgin full-upgrade && pkgin clean
```


## Install packages

### Fetch custom binary packages

Most of the packages we need to install are available as pre-compiled binary packages in Joyent's `pkgsrc` binary repository.

However, we also require a few custom binary packages, that are made available in a separate git repository.

```
# git clone https://github.com/JoeKun/pkgsrc-macos-binary-packages.git
```

### Basic command line utilities

```
# pkgin install fortune par vim colordiff wget
# pkg_add -C /dev/null pkgsrc-macos-binary-packages/packages/figlet-2.2.5nb2.tgz
# pkg_add -C /dev/null pkgsrc-macos-binary-packages/packages/vimpager-2.07-alpha-1.tgz
# pkg_add -C /dev/null pkgsrc-macos-binary-packages/packages/diff-so-fancy-1.2.7nb1.tgz
```

And here are some optional packages you might also want to install.

```
# pkgin install mmv pstree
# pkgin install p7zip pbzip2 unrar
# pkgin install coreutils
```

### Terminal Markdown Viewer

```
# pkgin install py310-docopt py310-expat py310-markdown py310-pygments py310-yaml py310-tabulate
# pkg_add -C /dev/null pkgsrc-macos-binary-packages/packages/py310-mdv3-2.0.2.tgz
# pkgin unkeep py310-docopt py310-expat py310-markdown py310-pygments py310-yaml py310-tabulate
```


## Building custom binary packages

In order to build those binary packages, you need to follow [Joyent's guide for package development](https://github.com/joyent/pkgsrc/wiki/pkgdev:setup).

That said, those instructions are lacking in some ways. Here are a few more things you should do on your build macOS machine.

### Create symbolic link for tools packages

```
$ cd /Volumes/data/packages/Darwin/11.0
$ mkdir trunk
$ cd trunk
$ ln -s ../x86_64 tools
```

### Install additional tools

After installing Xcode from the App Store, there are a couple more things you should install.

 - Command Line Tools for Xcode from [Apple Developer Downloads](https://developer.apple.com/downloads/).
 - [Pandoc](https://www.pandoc.org/).
	 - This is useful to generate `man` pages for `vimpager`, and building it manually takes forever.

### Basic `git` configuration

```
$ git config --global user.name "Foo Bar"
$ git config --global user.email "foo@bar.com"
$ git config --global apply.whitespace nowarn
```

### Prepare `root`'s home directory

```
$ sudo -s

# mkdir /var/root/.ssh
# chmod 700 /var/root/.ssh

# mkdir /var/root/.gnupg
# chmod 700 /var/root/.gnupg
# touch /var/root/.gnupg/pubring.gpg
# chmod 600 /var/root/.gnupg/pubring.gpg
```

### Build main bootstrap package

```
$ sudo -s
# export HOME=/var/root
# cd /Volumes/data/pkgsrc/bootstrap
# export CFLAGS="-pipe -Os"
# ./bootstrap --prefix /opt/pkg --pkgdbdir /opt/pkg/.pkgdb --pkgmandir=share/man --varbase /var --sysconfbase /etc --mk-fragment /Volumes/data/pkgbuild/conf/macos11-trunk-x86_64/mk-fragment.conf --workdir /tmp/pkgsrc-macos11-trunk-x86_64 --make-jobs 2
# cd /opt/pkg
# mkdir .fseventsd
# touch .fseventsd/no_log
# touch .metadata_never_index
# cd /
# tar cvzf /var/root/bootstrap-macos11-trunk-x86_64.tar.gz opt
# chown pbulk:staff /var/root/bootstrap-macos11-trunk-x86_64.tar.gz
# mv /var/root/bootstrap-macos11-trunk-x86_64.tar.gz /Volumes/data/packages/Darwin/bootstrap-pbulk
# cd /Volumes/data/pkgsrc/bootstrap
# ./cleanup
# rm -R -f /opt/pkg /tmp/pkgsrc-macos11-trunk-x86_64
```

### Build tools bootstrap package

```
$ sudo -s
# export HOME=/var/root
# cd /Volumes/data/pkgsrc/bootstrap
# export CFLAGS="-pipe -Os"
# ./bootstrap --prefix /opt/tools --pkgdbdir /opt/tools/.pkgdb --pkgmandir=share/man --sysconfbase /etc --mk-fragment /Volumes/data/pkgbuild/conf/macos11-trunk-tools/mk-fragment.conf --workdir /tmp/pkgsrc-macos11-trunk-tools --make-jobs 2
# cd /opt/tools
# mkdir .fseventsd
# touch .fseventsd/no_log
# touch .metadata_never_index
# cd /
# tar cvzf /var/root/bootstrap-macos11-trunk-tools.tar.gz opt
# chown pbulk:staff /var/root/bootstrap-macos11-trunk-tools.tar.gz
# mv /var/root/bootstrap-macos11-trunk-tools.tar.gz /Volumes/data/packages/Darwin/bootstrap-pbulk
# cd /Volumes/data/pkgsrc/bootstrap
# ./cleanup
# rm -R -f /opt/tools /tmp/pkgsrc-macos11-trunk-tools
```

### Sandbox setup

Create a sandbox environment as `root`:

```
$ sudo -s
# export HOME=/var/root
# run-sandbox macos11-trunk-x86_64
```

Once you've done that, you need to manually mount `/usr/local` in the sandbox environment, in order to be able to use `pandoc`, which was installed previously in `/usr/local`. To do that, open a new tab in your Terminal window, and run the following commands:

```
$ sudo -s
# mount localhost:/usr/local /Volumes/data/chroot/dev-macos11-trunk-x86_64/usr/local
```

Back in the first tab with the sandbox shell, add `/usr/local/bin` to the `PATH`:

```
--<root@pkgsrc>-(/Volumes/data/chroot/dev-macos11-trunk-x86_64)-<~>--
-> export PATH="/usr/local/bin:${PATH}"
```

Then build your custom packages as needed.

Once you're completely finished with the sandbox, go back to the second tab where you mounted `/usr/local`, and unmount it manually:

```
# umount /Volumes/data/chroot/dev-macos11-trunk-x86_64/usr/local
```

You can now logout from that tab and close it.

Then, back in the sandbox tab, you can logout and close it as well.