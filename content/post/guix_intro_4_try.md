+++
title = "Guix Introduction Part 4: Try Guix"
date = 2021-05-12
tags = ["Guix", "Functional Package Manager", "Reproducibility"]
categories = ["Guix"]
draft = false
author = "Peter Lo"
+++

This is the fourth part of a brief introduction to the Guix functional
package manager, and how it could be used to manage dependencies of
projects, much like virtual environments for Python, but with much
larger scope.

This time we look at how to install Guix, either in a physical or
virtual machine, either as a complete GNU/Linux distribution, or just
as a package manager on top of another Linux distribution. We also
list some commonly used Guix commands


## Get Guix {#get-guix}

Refer to [System Installation](https://guix.gnu.org/manual/en/guix.html#System-Installation) for details on getting Guix running
(either the Guix system, or just the Guix package manager), we briefly
summarize the main ways below. At the time of writing, the Guix
version is 1.2.0.


### Run Guix system in qemu virtual machine with pre-made image {#run-guix-system-in-qemu-virtual-machine-with-pre-made-image}

Running Guix in a virtual machine is probably the easiest way to
try it out. [Qemu](https://www.qemu.org/) is a generic and open source emulator and virtualizer
that can be used to run a pre-made image.

Refer to [Running Guix in a Virtual Machine](https://guix.gnu.org/manual/en/guix.html#Running-Guix-in-a-VM) for more details. At the
time of writing, the Guix version of the image is 1.2.0 for x86\_64
architecture. We summarize the steps below:

1.  install qemu following the instructions at [https://www.qemu.org/download/](https://www.qemu.org/download/)
2.  download the [guix-system-vm-image-1.2.0.x86\_64-linux.xz](https://ftp.gnu.org/gnu/guix/guix-system-vm-image-1.2.0.x86%5F64-linux.xz) compressed image
3.  decompress to get `guix-system-vm-image-1.2.0.x86_64-linux`

    ```shell
    xz -d guix-system-vm-image-1.2.0.x86_64-linux.xz
    ```
4.  invoke qemu on the image with

    ```shell
    qemu-system-x86_64 \
       -nic user,model=virtio-net-pci \
       -enable-kvm -m 1024 \
       -device virtio-blk,drive=myhd \
       -drive if=none,file=./guix-system-vm-image-1.2.0.x86_64-linux,id=myhd
    ```
5.  you can change arguments such as `-m` as appropriate to set the
    amount of memory desired. Note that you may need to tweak the
    arguments of invoking qemu a bit, depending on your host system.
    E.g. in MacOS Mojave 10.14.6, I need to use the following:

    ```shell
    qemu-system-x86_64 \
       -nic user,model=virtio-net-pci \
       -m 1024 \
       -device virtio-blk,drive=myhd \
       -drive if=none,file=./guix-system-vm-image-1.2.0.x86_64-linux,id=myhd \
       -accel hvf
    ```
6.  wait for a while, then you will be in the default xfce desktop of
    a Guix system, i.e. a GNU/Linux distribution built around
    Guix. But note that by default, not many packages are installed,
    e.g. even a web browser is not installed by default. You may
    install them as needed, e.g. you may install `icecat` (the GNU
    version of Firefox browser) by:

    ```shell
    guix package -i icecat
    ```
7.  optional: since the command for invoking qemu can be very long,
    you may put it into a shell script to more conveniently invoke the
    VM
8.  optional: you may use other front-ends (some are GUI) to more
    conveniently manage qemu VM, see
    <https://wiki.archlinux.org/index.php/Libvirt#Client>


### Install the Guix system, the GNU/Linux distribution built on the Guix package manager {#install-the-guix-system-the-gnu-linux-distribution-built-on-the-guix-package-manager}

You can also install a Guix system in either a virtual or physical machine.

-   **Get the Guix system install ISO image**

    Refer to [USB Stick and DVD Installation](https://guix.gnu.org/manual/en/guix.html#USB-Stick-and-DVD-Installation)

    1.  download the compressed 64-bit ISO from <https://ftp.gnu.org/gnu/guix/guix-system-install-1.2.0.x86%5F64-linux.iso.xz>
        -   you may instead get the compressed 32-bit ISO from <https://ftp.gnu.org/gnu/guix/guix-system-install-1.2.0.i686-linux.iso.xz>
    2.  optionally verify the authenticity of the image:
        -   get the sig file for 64-bit ISO and verify (change `x86_64-linux` to `i686-linux` if you download the 32-bit ISO)

            ```shell
            wget https://ftp.gnu.org/gnu/guix/guix-system-install-1.2.0.x86_64-linux.iso.xz.sig
            gpg --verify guix-system-install-1.2.0.x86_64-linux.iso.xz.sig
            ```
        -   if then above command fails, then import the gpg public key first and then re-run the above verification:

            ```shell
            wget https://sv.gnu.org/people/viewgpg.php?user_id=15145 \
                  -qO - | gpg --import -
            ```
    3.  decompress the image with `xz -d` command to get `guix-system-install-1.2.0.x86_64-linux.iso`

        ```shell
        xz -d guix-system-install-1.2.0.x86_64-linux.iso.xz
        ```

-   **Install in a fresh qemu virtual machine**

    Refer to [Installing Guix in a Virtual Machine](https://guix.gnu.org/manual/en/guix.html#Installing-Guix-in-a-VM)

    1.  install qemu following the instructions at [https://www.qemu.org/download/](https://www.qemu.org/download/)
    2.  create a fresh disk image in qcow2 format, where you can choose an
        appropriate size, e.g. 50G (the created image file will initially
        be much smaller, but will grow in size when it is filled up)

        ```shell
        qemu-img create -f qcow2 guix-system.img 50G
        ```
    3.  boot the installation ISO image in the VM:

        ```shell
        qemu-system-x86_64 -m 1024 -smp 1 -enable-kvm \
          -nic user,model=virtio-net-pci -boot menu=on,order=d \
          -drive file=guix-system.img \
          -drive media=cdrom,file=guix-system-install-1.2.0.system.iso
        ```
    4.  note that you may need to tweak the arguments to
        `qemu-system-x86_64` depending on your host system, e.g. you may
        need to replace `-enable-kvm` with `-accel hvf` in MacOS. The
        `-enable-kvm` is optional anyway, but will significantly improve
        performance.
    5.  note the few things mentioned in <https://guix.gnu.org/manual/en/guix.html#Preparing-for-Installation>
        -   The graphical installer is available on TTY1.
        -   You can obtain root shells on TTYs 3 to 6 by hitting ctrl-alt-f3, ctrl-alt-f4, etc.
        -   TTY2 shows the documentation and you can reach it with ctrl-alt-f2.
        -   Documentation is browsable using the Info reader commands (see [Stand-alone GNU Info](https://www.gnu.org/software/texinfo/manual/info-stnd/info-stnd.html#Top)).
        -   The installation system runs the GPM mouse daemon, which
            allows you to select text with the left mouse button and to
            paste it with the middle button.
    6.  for simplicity, choose [Guided Graphical Installation](https://guix.gnu.org/manual/en/guix.html#Guided-Graphical-Installation) to finish the installation
    7.  after the installation, you can get into guix with the default
        xfce desktop whenever you invoke the VM
    8.  optional: since the command for invoking qemu can be very long,
        you may put it into a shell script to more conveniently invoke the
        VM
    9.  optional: you may use other front-ends (some are GUI) to more
        conveniently manage qemu VM, see
        <https://wiki.archlinux.org/index.php/Libvirt#Client>

-   **Install in other virtual machine**

    You can also install Guix system using the ISO in other types of
    virtual machines such as:

    -   [VirtualBox](https://www.virtualbox.org/) (free)
    -   [Parallels](https://www.parallels.com/) (proprietary)
    -   [VmWare](https://www.vmware.com/) (proprietary)
    -   and many others, see <https://en.wikipedia.org/wiki/Comparison%5Fof%5Fplatform%5Fvirtualization%5Fsoftware>

    Similar to installing in qemu VM:

    -   note the few things mentioned in <https://guix.gnu.org/manual/en/guix.html#Preparing-for-Installation>
        -   The graphical installer is available on TTY1.
        -   You can obtain root shells on TTYs 3 to 6 by hitting ctrl-alt-f3, ctrl-alt-f4, etc.
        -   TTY2 shows the documentation and you can reach it with ctrl-alt-f2.
        -   Documentation is browsable using the Info reader commands (see [Stand-alone GNU Info](https://www.gnu.org/software/texinfo/manual/info-stnd/info-stnd.html#Top)).
        -   The installation system runs the GPM mouse daemon, which allows you to select text with the left mouse button and to paste it with the middle button.
    -   for simplicity, choose [Guided Graphical Installation](https://guix.gnu.org/manual/en/guix.html#Guided-Graphical-Installation) to finish the installation
    -   after the installation, you can get into guix with the default
        xfce desktop whenever you invoke the VM

-   **Install on physical machine**

    Installing Guix system on physical machine is similar to installing in
    virtual machine, except that you need to transfer the ISO image to
    either a USB stick or a DVD. It is recommended that you try to install
    in a virtual machine first to get familiar with the process, see the
    preceding subsection.

    Refer to <https://guix.gnu.org/manual/en/guix.html#USB-Stick-and-DVD-Installation>

    -   **Copying to a USB Stick**
        1.  Insert a USB stick of 1 GiB or more into your machine
        2.  if on a Linux machine,
            -   determine its device name, e.g. `/dev/sdb` or `/dev/sdc`. Note
                that it is **VERY** important to determine the correct device name,
                otherwise you may mistakenly wipe out another device.
            -   assuming that the USB stick is known as `/dev/sdX`, copy the image
                with:

                ```shell
                # access to /dev/sdX usually requires root privilege
                # also, /dev/sdX should NOT be mount
                sudo dd if=guix-system-install-1.2.0.x86_64-linux.iso of=/dev/sdX status=progress
                sudo sync
                ```
        3.  if on a different OS or you prefer to use a GUI, you can try these programs:
            -   <https://www.tipard.com/resource/burn-iso-to-usb.html>
            -   <https://techrrival.com/best-usb-bootable-softwares/>

    -   alternative: **Burning on a DVD**
        1.  Insert a blank DVD into your machine
        2.  if on a Linux machine,
            -   determine its device name. Note that it is **VERY** important to
                determine the correct device name, otherwise you may mistakenly
                wipe out another device.
            -   assuming that the DVD drive is known as `/dev/srX`, copy the image
                with:

                ```shell
                # access to /dev/srX usually requires root privilege
                growisofs -dvd-compat -Z /dev/srX=guix-system-install-1.2.0.x86_64-linux.iso
                ```
        3.  if on a different OS or you prefer to use a GUI, you can try these programs:
            -   <https://en.wikipedia.org/wiki/List%5Fof%5FISO%5Fimage%5Fsoftware>
            -   <https://dvdcreator.wondershare.com/dvd-tips/best-free-iso-burner.html>

    -   **Installation**

        Similar to installing in qemu VM:

        -   note the few things mentioned in <https://guix.gnu.org/manual/en/guix.html#Preparing-for-Installation>
            -   The graphical installer is available on TTY1.
            -   You can obtain root shells on TTYs 3 to 6 by hitting ctrl-alt-f3, ctrl-alt-f4, etc.
            -   TTY2 shows the documentation and you can reach it with ctrl-alt-f2.
            -   Documentation is browsable using the Info reader commands (see [Stand-alone GNU Info](https://www.gnu.org/software/texinfo/manual/info-stnd/info-stnd.html#Top)).
            -   The installation system runs the GPM mouse daemon, which
                allows you to select text with the left mouse button and to
                paste it with the middle button.
        -   for simplicity, choose [Guided Graphical Installation](https://guix.gnu.org/manual/en/guix.html#Guided-Graphical-Installation) to finish the installation
        -   after the installation, you can get into guix with the default
            xfce desktop


### Install the guix package manager in existing GNU/Linux distribution (possibly in a VM) {#install-the-guix-package-manager-in-existing-gnu-linux-distribution--possibly-in-a-vm}

Instead of installing a Guix system, you may choose to only install
guix package manager on top of a GNU/Linux distribution (named
"foreign distro in Guix documentation", which does not need to be a Guix
system, e.g. Debian, Ubuntu, Fedora, ArchLinux, etc).

Refer to [Binary Installation](https://guix.gnu.org/manual/en/guix.html#Binary-Installation)

-   the easiest way is to use the shell installer script:
    1.  download and execute the shell installer script as **root**

        ```shell
        cd /tmp
        wget https://git.savannah.gnu.org/cgit/guix.git/plain/etc/guix-install.sh
        chmod +x guix-install.sh
        ./guix-install.sh
        ```

        -   you may be asked to import gpg key, if so, just follow the instructions, then re-run the installer script
    2.  after that, check [Application Setup](https://guix.gnu.org/manual/en/guix.html#Application-Setup) for extra configuration you might need
-   alternatively, you may choose to install the guix package from the
    package manager of your distribution, if there is one, e.g.:
    -   ArchLinux: <https://wiki.archlinux.org/index.php/Guix#AUR%5FPackage%5FInstallation>
    -   Slackware: <https://slackbuilds.org/repository/14.1/system/guix/>
    -   Fedora: <https://copr.fedorainfracloud.org/coprs/lantw44/guix/>


## Brief summary of commonly used commands in Guix {#brief-summary-of-commonly-used-commands-in-guix}

Here we list some common commands and usage of Guix, see [Getting Started](https://guix.gnu.org/manual/en/guix.html#Getting-Started) of the Guix manual for more details.


### Basic usage of Guix {#basic-usage-of-guix}

-   **Help on Guix commands**
    -   Help on the main usage, to see the available commands

        ```shell
        guix --help
        ```

    -   Help on each command, e.g. the `guix package` command

        ```shell
        guix package --help
        ```
-   **Search packages**

    ```shell
    guix search keywords-here
    # guix package -s keywords-here
    ```
-   **Show information of package(s)**

    ```shell
    guix show package1 package2
    ```
-   **Update channel(s)**

    To update the channels so that all the package definitions are
    updated, but the packages themselves are not automatically
    updated.

    ```shell
    guix pull
    ```
-   **Install package(s)**
    -   You can install one or more packages in one transaction

        ```shell
        guix install package1 package2
        ```

    -   An alternative way

        ```shell
        guix package -i package1 package2
        # guix package --install package1 package2
        ```
-   **Remove package(s)**
    -   `guix remove`

        ```shell
        guix remove package1 package2
        ```

    -   An alternative way

        ```shell
        guix package -r package1 package2
        # guix package --remove package1 package2
        ```
-   **Upgrade package(s)**
    -   `guix upgrade`

        ```shell
        guix upgrade package1 package2
        ```

    -   An alternative way

        ```shell
        guix package -u package1 package2
        # guix package --upgrade package1 package2
        ```
-   **Install/Remove/Upgrade package(s) in one transaction**

    You can also install/remove/upgrade package(s) in a single
    transaction, simply list the package(s) with suitable options of
    `guix package`, `-i` for installing, `-r` for removing, `-u` for
    upgrading:

    ```shell
    guix package -i package1 -r package2 -u package3
    ```
-   **List installed packages**

    ```shell
    guix package -I
    # guix package --list-installed
    ```
-   **Rollback to the previous generation**

    ```shell
    guix package --roll-back
    ```
-   **List generations**

    ```shell
    guix package -l
    # guix package --list-generations
    ```
-   **Switch generations**

    -   `PATTERN` can be a generation number or `+n` to go forward n
        generations; or `-n` to go backward n generations.

    <!--listend-->

    ```shell
    guix package -S PATTERN
    # guix package --switch-generation=PATTERN
    ```

    -   E.g. to go forward one generation

    <!--listend-->

    ```shell
    guix package -S +1
    # guix package --switch-generation=+1
    ```

    -   E.g. to go backward one generation, similar to `--roll-back`,
        but if the specified generation does not exist, the current
        generation will not change

    <!--listend-->

    ```shell
    guix package -S -1
    # guix package --switch-generation=-1
    ```


### Using manifest file to specify a list of package(s) {#using-manifest-file-to-specify-a-list-of-package--s}

-   A manifest file is a Scheme file that evaluates to a list of guix
    _packages_ (as Scheme object).
-   A convenient way is to use the `specifications->manifest`
    function which takes a list of package names as strings and
    returns a list of the corresponding package objects.
-   It can be used in a few contexts such as `guix package` to
    install packages, or `guix environment` to spawn a shell with
    specified packages (see below), through the `-m` or `--manifest`
    option.
-   The advantage of a manifest file is that the list of packages can
    be easily version controlled (e.g. with `git`).

-   An example of manifest file (let's say `pkgs.scm`) is:

<!--listend-->

```scheme
(specifications->manifest
 '(
   ;; R
   "r"
   "r-yaml"
   "r-xgboost"
   "r-tidymodels"
   "r-tidyverse"
   "r-survminer"
   ))
```

-   **Install the list of package(s) in a manifest file**

    Note that the `-m` can be repeated multiple times with different
    manifest files, and the lists will be concatenated and all will
    be installed.

    ```shell
    guix package -m pkgs.scm
    ```

    Note that a manifest file by itself only specifies the list of
    packages by name, but not the exact versions. In order to
    reproduce the exact versions of the list of packages, a manifest
    file can be combined with `guix time-machine` as described below.


### Using Guix profiles {#using-guix-profiles}

Default guix actions (install/upgrade/remove) operate on the
default _profile_ of each user (it is a symbolic link, default
location is at `$HOME/.guix-profile`), but each user can create
multiple profiles, activate the profiles as needed in similar ways
to Python _virtual environments_, and specify the profile to
operate on with each action through the `-p` (or `--profile`)
option (see [Invoking guix package](https://guix.gnu.org/manual/en/guix.html#Invoking-guix-package)):

-   **Create profile**

    The profile name is basically symbolic link(s) to the real
    contents of the profile, and we can put it basically anywhere,
    e.g. `~/my-ds-profile`. So it is up to you to organize the
    profiles in a sensible way. It is also up to you to assign the
    roles of different profiles, e.g. one profile for each project;
    or different profiles for different domains of packages (e.g. one
    profile for data analysis, one profile for entertainment, one
    profile for graphics editing, etc).

    A profile will be created when you perform guix package actions
    specifying a profile, e.g. install `r` and `r-xgboost` in the
    profile `~/my-ds-profile`, which need not exist before this
    action:

    ```shell
    guix package -i r r-xgboost -p ~/my-ds-profile
    ```

    Note that the created profile is not automatically activated, so
    you retain the maximum flexibility to activate as needed, or
    arrange to activate chosen profiles in each shell, see below.

-   **Install or upgrade or remove packages in a profile**

    -   Any usual `guix package` actions can be performed on a chosen
        profile by using the `-p` (or `--profile`) option, e.g. to
        install another package `r-yaml` to the profile
        `~/my-ds-profile` just created:

    <!--listend-->

    ```shell
    guix package -i r-yaml -p ~/my-ds-profile
    ```

    -   E.g. install packages using a manifest file in a profile:

    <!--listend-->

    ```shell
    guix package -m the_manifest_file -p path_to_profile
    ```
-   **List profiles**

    ```shell
    guix package --list-profiles
    ```

-   **Activate a profile**

    Activating a profile amounts to modifying and exporting some
    environment variables (e.g. `PATH`) in the _current shell_, which
    is conveniently done by sourcing the `etc/profile` file under the
    profile. The recommended way to activate a profile,
    e.g. `~/my-ds-profile`, is to:

    ```shell
    GUIX_PROFILE="~/my-ds-profile" ; . "$GUIX_PROFILE"/etc/profile
    ```

    Note that you can activate multiple profiles in the same shell as
    appropriate. But there is no effective way to _deactivate_ a
    profile in the same shell, other than exiting the shell. The
    upside is that the global environment is not "polluted", and you
    can start a new shell to activate a profile, then exit the shell
    afterwards.

    Also, you can automatically activate any profile in each spawn
    shell by adding the activation to your shell initialization,
    e.g. `.bash_profile` or `.bashrc`.

-   **Delete a profile**

    Somehow Guix does not provide a convenient command to delete a
    profile. Deleting a profile involves removing the profile
    symbolic link and its siblings that point to specific
    generations. E.g. to delete the `~/my-ds-profile` profile created
    above:

    ```shell
    rm ~/my-profile ~/my-profile-*-link
    ```

-   **Export current or chosen profile as a manifest file**

    You can export the list of packages of a profile (if not
    specified, will use the default profile) to a manifest
    file. E.g. to export the default profile:

    ```shell
    guix package --export-manifest > pkgs1.scm
    ```

    E.g. to export the above `~/my-ds-profile` profile:

    ```shell
    guix package --export-manifest -p ~/my-ds-profile > pkgs2.scm
    ```


### Using Guix environment {#using-guix-environment}

The `guix environment` command allows you to spawn a new shell with
certain packages (or their dependencies) accessible (by modifying some
environment variables, e.g. `PATH`), _without_ changing your other
profiles; and you can optionally either just spawn such a shell, or
execute a command inside this shell. Essentially a temporary profile
is created (for the set of packages wanted), so that you do not
"pollute" your other environment. The needed packages will be
automatically built if they are not yet in the cached store. So `guix
   environment` is a very convenient way to reproduce a set of packages
in a clean way. For example, this can be used to manage per-project
dependencies, as we will see in a future demo post of this series.

Note that when in a shell spawn by `guix environment`, the environment
variable `GUIX_ENVIRONMENT` will have the value of the path to the
temporary profile for the environment, and you can detect it to change
the shell prompt, e.g. add this to your .bashrc:

```shell
if [ -n "$GUIX_ENVIRONMENT" ]
then
  export PS1="\u@\h \w [dev]\$ "
fi

```

-   **Spawn a shell with dependencies of some packages**
    E.g. to get the dependencies of `emacs` and `vim`:

    ```shell
    # guix environment pkg1 pkg2 ...
    guix environment emacs vim
    ```

    Can also use `-m` (possibly multiple times) to specify manifest file(s):

    ```shell
    # guix environment pkg1 pkg2 ... -m pkgs1.scm -m pkgs2.scm ...
    guix environment kmahjongg -m pkgs.scm
    ```

    Also check out the `--load` option for loading a file for package.

-   **Spawn a shell with some packages themselves**

    Use the `--ad-hoc` option to get the packages themselves.
    E.g. to get the game `kmahjongg` itself:

    ```shell
    # guix environment --ad-hoc pkg1 pkg2 ...
    guix environment --ad-hoc kmahjongg
    ```

    Can also use `-m` (possibly multiple times) to specify manifest file(s):

    ```shell
    # guix environment --ad-hoc pkg1 pkg2 ... -m pkgs1.scm -m pkgs2.scm ...
    guix environment --ad-hoc kmahjongg -m pkgs.scm
    ```

    Also check out the `--load` option for loading a file for package.

-   **Spawn a shell and directly execute some commands in it**

    To run commands in the spawn shell, put the command after `--` at the end.
    E.g. to get `R` and start it

    ```shell
    # guix environment --ad-hoc pkg1 ... -- pkg1
    guix environment --ad-hoc r -- R
    ```

-   **Spawn a shell where original environment variables are unset**

    Use the `--pure` option, e.g. to get R

    ```shell
    # show all environment variables currently defined
    env
    # guix environment --pure --ad-hoc pkg1 ... -- pkg1
    guix environment --pure --ad-hoc r
    # then in the new shell, the PATH is reduced to only those for the
    # package, and even basic shell utilities are not in the PATH unless
    # they are added as one of the packages.
    echo $PATH
    ```

    You may use the `--preserve` to control the environment variables
    to keep, check `guix environment --help` for more details.

-   **Spawn a shell to run things in an isolated environment**

    For further isolation, you can use the `--container` option to run in
    a container where only the `/gnu/store` and (by default) the current
    directory are mounted. You may check `guix environment --help` for
    other relevant options, e.g. `--expose`, `--network`, `--share`,
    `--no-cwd`, `--user`.

    E.g. to run R in a container:

    ```shell
    # guix environment --ad-hoc pkg1 ... -- pkg1
    guix environment --container --ad-hoc r -- R
    ```

    But seems this feature is not very mature yet. When I tried the
    above command (on guix (GNU Guix)
    510e24f973a918391d8122fd6ad515c0567bf23e),  it gave this error:

    ```text
    guix environment: error: mount: mount "/home/peter" on "/tmp/guix-directory.QDXQmx//home/peter": Invalid argument
    ```


### Using Guix time-machine {#using-guix-time-machine}

The `guix time-machine` can be used to execute other Guix commands (e.g. `guix package`, or `guix environment`) at specified Guix revisions (exact commits of the channels, which can be produced by `guix describe`)

-   **Install package in a particular revision**

    E.g. to install `emacs` at the revision in the file `channels.scm`

    ```shell
    guix time-machine -C channels.scm -- package -i emacs
    ```

    The `channels.scm` could be obtained with `guix describe`, e.g.

    ```shell
    guix describe -f channels > channels.scm
    ```

    The `channels.scm` will look something like (exact content of course depends on your channels) this:

    ```scheme
    (list (channel
            (name 'nonguix)
            (url "https://gitlab.com/nonguix/nonguix")
            (commit
              "05fad6e54d497b7427ab7ea488210c7d43c3f676")
            (introduction
              (make-channel-introduction
                "897c1a470da759236cc11798f4e0a5f7d4d59fbc"
                (openpgp-fingerprint
                  "2A39 3FFF 68F4 EF7A 3D29  12AF 6F51 20A0 22FB B2D5"))))
          (channel
            (name 'guix)
            (url "https://git.sjtu.edu.cn/sjtug/guix.git")
            (commit
              "8a452e156e37568fdd59df169acb4f2092aeb3ac")
            (introduction
              (make-channel-introduction
                "9edb3f66fd807b096b48283debdcddccfea34bad"
                (openpgp-fingerprint
                  "BBB0 2DDF 2CEA F6A8 0D1D  E643 A2A0 6DF2 A33A 54FA")))))
    ```

    Note that it is possible to hand-edit the file to the commit desired.

-   **Use time-machine with guix environment**

    Assuming there is a manifest file `pkgs.scm` containing the needed packages, e.g.

    ```scheme
    (specifications->manifest
     '(
       ;; R
       "r"
       "r-yaml"
       "r-xgboost"
       "r-tidymodels"
       "r-tidyverse"
       "r-survminer"
       ))
    ```

    Then you can open a shell with these packages at the revisions specified in a `channels.scm`

    ```shell
    guix time-machine -C channels.scm -- environment --ad-hoc -m pkgs.scm
    # in this new shell where the packages are visible, you can run anything as needed
    ```

    And you can directly run things in the new shell, e.g. to run R:

<!--listend-->

```shell
guix time-machine -C channels.scm -- environment --ad-hoc -m pkgs.scm -- R
```


### Use Guix to create docker image {#use-guix-to-create-docker-image}

Guix can produce a docker image of a set of packages through the `guix
   pack` command, see [Invoking guix pack](https://guix.gnu.org/manual/en/html%5Fnode/Invoking-guix-pack.html#Invoking-guix-pack) and `guix pack --help` for more
description. In fact, `guix pack` can produce other formats, see `guix
   pack --list-formats` for the possible formats.

-   **Produce a docker image of a list of packages**

    The set of packages can be specified as a list when calling `guix pack`.
    E.g. to get `R` and `xgboost`:

    ```shell
    # guix pack -f docker -S /bin=bin pkg1 pkg2 ...
    guix pack -f docker -S /bin=bin r r-xgboost
    ```

    The `-S` option creates a symbolic link to file or directory
    under the profile (which is otherwise a path with long hash),
    provided for convenience. In the above example, a symbolic link will
    be created at `/bin` pointing to the `bin` directory in the
    profile (which will be something like
    `/gnu/store/...-profile/bin`), then the user can start `R` with
    `/bin/R`.

    The path to the created tar.gz (something like
    `/gnu/store/...-docker-pack.tar.gz` with a long hash) file is
    output to stdout of the `guix pack` call, so we can get it like:

    ```shell
    PACKLOC=$(guix pack -f docker -S /bin=bin r r-xgboost)
    # then can easily copy it elsewhere
    ```

    Then the file produced is a docker image that can be loaded into
    docker:

    ```shell
    # assume the file name is shortened to docker-pack.tar.gz
    docker load < docker-pack.tar.gz
    # docker will show the image tag after loading
    # the tag of the image is generated from the set of packages
    docker run -ti r-r-xgboost /bin/R
    ```

-   **Produce a docker image of packages from a manifest file**

    The set of packages can also be specified by a manifest file,
    similar to `guix package`, e.g. if we have a manifest file
    `pkgs.scm`:

    ```shell
    guix pack -f docker -S /bin=bin -m pkgs.scm
    ```

    NOTE: the documentation specifically mention that the set of
    packages can be specified by either a list of package names as
    arguments to `guix pack`, or through a manifest file, but NOT
    BOTH. It is strange that there is such a limitation, hopefully it
    will be changed in the future.

-   **Also specify an entry point command**

    The `--entry-point` option can specify a command to run when the
    docker image is started, e.g. to run guile:

    ```shell
    # seems the command is relative to the profile path
    guix pack -f docker --entry-point=bin/guile guile
    ```

    Then the produced image (say renamed to `pack.tar.gz`), can be
    loaded into docker and run as follows and `guile` will start
    automatically:

    ```shell
    docker load -i pack.tar.gz
    #docker run image-id
    # assume the image id is "guile"
    docker run guile
    ```

-   **Also use time-machine to pin package versions**

    The `guix time-machine` can be used, together with a manifest
    file, to easily produce a docker image with the set of desired
    packages pinned to versions at a certain time point.

    E.g. suppose we have the channel file `channels.scm`, and
    manifest file `pkgs.scm`:

    ```shell
    guix time-machine -C channels.scm -- pack -f docker -S /bin=bin -m pkgs.scm
    ```

    The above have been tested with `guix (GNU Guix)
         ff1c7e40252337cb2c992c2d41e0ed4cefe56df0` (installed on top of
    Debian 10 in a qemu VM), and the produced docker images have been
    tested on MacOS with Docker version 19.03.8, build afacb8b.


## What's next? {#what-s-next}

In this part we looked at various ways of installing Guix to try it
out, and showed some commonly used Guix commands. Next time we show a
little demo of managing per-project dependency with Guix, through the
`guix time-machine` and `guix environment` commands.
