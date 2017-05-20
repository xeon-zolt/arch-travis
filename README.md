# arch-travis [![Travis BuildStatus](https://travis-ci.org/mikkeloscar/arch-travis.svg?branch=master)](https://travis-ci.org/mikkeloscar/arch-travis)

arch-travis provides a chroot based Arch Linux build environment for
[Travis-CI][travis-ci] builds. It supports a very simple (and limited)
configuration based on `.travis.yml`.

Example:
```yml
sudo: required

arch:
  repos:
    - papyros=archlinuxfr=http://repo.archlinux.fr/$arch
  packages:
    # pacman packages
    - python
    - perl
    # aur packages
    - go-git
    # packages from archlinuxfr 
    - yaourt
  script:
    - "./build_script.sh"

script:
  - "curl -s https://raw.githubusercontent.com/xeon-zolt/arch-travis/master/arch-travis.sh | bash"
```

`arch.repos` defines a list of custom repositories.

`arch.packages` defines a list of packages (from official repos or AUR) to be
installed before running the build.

`arch.script` defines a list of scripts to run as part of the build. Anything
defined in the `arch.script` list will run from the base of the repository as a
normal user called `travis`. `sudo` is available as well as any packages
installed in the setup. The path of the build dir (or repository base), is
stored in the `TRAVIS_BUILD_DIR` environment variable inside the chroot.

`script` defines the scripts to be run by travis, this is where arch-travis is
initialized.

### Default packages and repositories

By default the following packages are installed and usable from within the
build environment.

* base-devel (group)
* [git](https://www.archlinux.org/packages/extra/x86_64/git/)
* [cower](https://aur.archlinux.org/packages/cower/)
* [pacaur](https://aur.archlinux.org/packages/pacaur/)

The following repositories are enabled by default.

* core
* extra
* community
* multilib

It is possible to use custom respositories by adding them to the `arch.repos`
section of `.travis.yml` using the following format:

```yml
arch:
  repos:
    - repo-name=http://repo.com/path
```

The first repository in the list will be added first in `pacman.conf` and all
custom repositories will be added before the default repositories.

### Limitations/tradeoffs

* Increases build time with about 1-3 min.
* Doesn't work on [travis container-based infrastructure][travis-container] because `sudo` is required.
* Limited configuration.
* Doesn't include `base` group packages. If you need anything
  from `base` just add it to the `arch.packages` list in `.travis.yml`.

## Advanced configuration

Apart from the basic `arch` entry in `.travis.yml` it is also possible to
define some environment variables in order to control the chroot setup.

The following variables are available:

`ARCH_TRAVIS_CHROOT` name of the folder containing the chroot. (default is
`root.x86_64`).

`ARCH_TRAVIS_MIRRORS` Comma separated list of Arch Linux mirrors used by
pacman. See list of available mirrors [here][arch-mirrors]. Omit the
`/$repo/os/$arch` part of the mirrors when defining this variable.

> Note some https mirrors are not supported due to [#4757][travis-issue-4757].

`ARCH_TRAVIS_CLEAN_CHROOT` by default the chroot archive and chroot folder is
left in `$TRAVIS_BUILD_DIR`, if you don't want this, then you can make
arch-travis remove them by enabling `ARCH_TRAVIS_CLEAN_CHROOT`.

`ARCH_TRAVIS_ARCH` Set the default architecture to use in the chroot. Valid
values are `x86_64` (default) and `i686`.

To use, just add the variable to the `env` section of `.travis.yml`.

```yml
env:
  - ARCH_TRAVIS_CHROOT="custom_root" ARCH_TRAVIS_CLEAN_CHROOT=1
```


### clang support

The default compiler available in the chroot is `gcc`, if you want to use
`clang` instead just add the following to `.travis.yml` and arch-travis will
export `CC=clang` in your build:


```yml
language: c

compiler: clang
```

## Projects using arch-travis

* [Event-Linux](https://github.com/xeon-zolt/Event-linux)
