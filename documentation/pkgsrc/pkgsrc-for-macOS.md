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
$ BOOTSTRAP_TAR="bootstrap-trunk-x86_64-20171103.tar.gz"
$ BOOTSTRAP_SHA="d7bee3a08e6e07818ff445f042c469333c96ac22"
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
# pkg_add -C /dev/null pkgsrc-macos-binary-packages/packages/vimpager-2.06nb1.tgz
# pkg_add -C /dev/null pkgsrc-macos-binary-packages/packages/diff-so-fancy-1.2.7nb1.tgz
```

And here are some optional packages you might also want to install.

```
# pkgin install mmv pstree
# pkgin install p7zip pbzip2 unrar
```

### Terminal Markdown Viewer

```
# pkgin install py27-docopt py27-markdown py27-yaml
# pkg_add -C /dev/null pkgsrc-macos-binary-packages/packages/py27-mdv-1.6.3.tgz
# pkgin unkeep py27-docopt py27-markdown py27-yaml
```