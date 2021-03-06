+++
title = "Guix Introduction Part 3: Why Guix?"
date = 2021-05-11
tags = ["Guix", "Functional Package Manager", "Reproducibility"]
categories = ["Guix"]
draft = false
author = "Peter Lo"
+++

This is the third part of a brief introduction to the Guix functional
package manager, and how it could be used to manage dependencies of
projects, much like virtual environments for Python, but with much
larger scope.

This time we look at why bother with Guix and how it compares with
some other dependency management alternatives such as: system and
language package managers; including all dependencies as in docker;
another functional package manager Nix.

Although there are a lot of package managers at both the language
level and operating system level, and tools such as docker to help
tackle the dependency problem, Guix still has merits when compared to
these, as we now discuss.


## Use operating system package managers {#use-operating-system-package-managers}

-   e.g. apt, yum, pacman
-   these package managers resolve dependencies for you, some allow pinning particular versions of package
-   sometimes language specific packages is also packaged


### downsides: {#downsides}

-   these package managers mostly operate by _mutating_ the system state when adding or removing packages, so may not be easy to revert to previous state in case installing/upgrading some packages causes problem
    -   if the process is interrupted, the system might be in an inconsistent state
    -   installing/upgrading package may cause some dependencies to be updated, which might break other packages
    -   subsequently removing that package does not necessarily revert the updated dependencies
-   since installing/upgrading package may cause dependencies to be updated, which may cause conflicts if some packages need older versions of the dependencies
    -   although this is not common, this could be painful when it happens
    -   this is problematic often when you want newer version of some package, but older version of another package, and they (or their chain of dependencies) somehow have version conflicts
-   since the packages are installed to some shared location, usually root privilege is needed to install/upgrade/remove packages, e.g. through `sudo`
    -   this is not much of a problem on desktop Linux system because it is often used by one person
    -   it is more of a problem on servers with multiple users, either we trust all users and give them root privilege (through `sudo`), or we reserve the root privilege to a small number of users (e.g. the system admins)
    -   and the possibility of possible breakage and conflicts when upgrading packages only increases with the number of users
    -   therefore, on such systems, some users may not get the packages he/she wants, either it is too new or too old, because the updating schedule is often set by the system admins
    -   package managers such as [Homebrew](https://brew.sh/) try to remedy this short-coming by installing packages under the user's home directory, and therefore each user can manage his/her own packages without root privilege
        -   although this reduces possible conflicts with other users, there are still possible conflicts among the packages


### upsides of Guix: {#upsides-of-guix}

-   Guix can be used just like the usual system package manager, although there are better ways of using it for specific needs
-   every install/remove/upgrade packages action (which may involve multiple packages) is one _transaction_, and causes a _generation_ to be created
    -   Guix can detect and remedy interrupt of the action to maintain transactional behavior, i.e. either it succeeds or fails as a whole, so that the system would not be in a half-completed inconsistent state.
        -   if an action is interrupted, it has not succeeded, and can be safely repeated
        -   but in case an action involves multiple packages, a completely downloaded or built package need not be re-built or re-downloaded when the action is repeated, since they would be cached (in a _store_ where only the Guix daemon can modify) and identified by hash.
    -   a _generation_ is essentially a record of the set of specific packages and their dependencies, so it is easy to revert to previous generation
        -   all the packages and dependencies referenced by a generation would be kept in the system
        -   so normally removing packages in Guix simply results in a generation without those packages, the packages themselves are still in the system cache, this is similar in spirit to git where a commit "deletes" some files.
        -   reverting to other generation involves only updating some symbolic links, so is quick
        -   the user can safely try out different versions of some packages, knowing that it is easy to revert to previous known good state if the user dislike the version for whatever reason (e.g. bugs, different UI, missing features, etc)
        -   the user can optionally delete older generations, and do a _garbage collection_ to really delete any unreferenced (directly or indirectly) packages to free up disk space
-   each package in Guix literally has its dependencies hard-coded using absolute paths to the dependency in the _store_ (as much as possible)
    -   each package in Guix is cached in `/gnu/store` with a path with some sort of hash to identify the exact version of the package
    -   therefore an updated package may also have updated dependencies, but older versions of the package or other packages still refer to their previous versions of dependencies fixed at built time
    -   therefore there is no fear that updating a package will break another package just because they share some dependencies with conflicting versions
    -   also, two packages with conflicting dependencies can coexist in Guix because each can have their own versions of dependencies
-   Guix allows each user to manage his/her own packages without root privilege
    -   the _store_ of installed packages in Guix is managed by a dedicated daemon, so is essentially read-only to the users, so identical packages can be safely shared
    -   each user has his/her symbolic links under the home directory to profiles containing sets of packages, so any install/upgrade/remove actions can be performed by the user without root privilege
    -   and as mentioned above, because of hard-coded dependencies for each package, there will not be package conflicts among users nor for the same user


## Use language specific package manager such as pip, packrat, npm, etc {#use-language-specific-package-manager-such-as-pip-packrat-npm-etc}

-   many programming languages have their own package manager, because the system package manager may not have these language-specific packages, and having a language specific one would be more uniform across different operating systems or Linux distributions
-   e.g.
    -   pip for Python
    -   `install.packages()` for R
    -   npm for Javascript
    -   RubyGems for Ruby
    -   Cabal for Haskell
-   for better management of possibly different packages for different projects, there are either some sort of _virtual environment_, or some kind of _lock files_ to pin the versions of set of packages for each project, e.g.
    -   [pyenv](https://github.com/pyenv/pyenv#simple-python-version-management-pyenv), [virtualenv](https://virtualenv.pypa.io/en/stable/), [anaconda](https://docs.continuum.io/anaconda/packages/pkg-docs/) for Python, see <https://stackoverflow.com/a/39928067> for a brief comparison
    -   [packrat](https://rstudio.github.io/packrat/) or [renv](https://rstudio.github.io/renv/) for R
    -   rubygems, npm and cabal have lock files


### downsides: {#downsides}

-   these language-specific package managers naturally only handle packages for one programming language
    -   if a project uses only one programming language, e.g. Python, then either one of the virtual environment manger may be sufficient
    -   but if the projects in the same team use multiple programming languages, e.g. both Python and R for data science projects, then the users would need to be familiar with multiple package managers
-   these package managers may not help with system-level dependencies, especially when pre-built binary package is not available (e.g. R packages under Linux) and the package needs to be built locally
-   some dependencies are not managed by these virtual environments
    -   e.g. packrat, being an R library, does not help manage the version of R itself
        -   although this is often OK because R is usually backward compatible, but sometimes there could be issues, e.g. see <https://github.com/rstudio/packrat/issues/327>
    -   in contrast, virtual environments in Python can also manage different versions of Python, because there are bigger differences between versions of Python
-   the virtual environments are often setup per-project, but identical packages (and dependencies) may be duplicated instead of shared, taking up more disk space than necessary (unless the file system had built-in support for deduplication)
    -   e.g. packrat for R install a copy of the needed packages for each project
    -   in contrast, renv for R has a global shared cache of packages, so that identical packages can be shared for different projects, see <https://cloud.r-project.org/web/packages/renv/vignettes/renv.html>


### upsides of Guix: {#upsides-of-guix}

-   Guix has system-level libraries, applications, language specific packages all at the same level, and can be managed in the same way.
    -   e.g. `r-tidyverse` is the Guix package for the R [tidyverse](https://www.tidyverse.org/) package, which depends on many other R packages, all of which can also be managed by Guix
    -   e.g. `python-numpy` is the Guix package for the Python [numpy](https://numpy.org/) package, which depends on `gfortran@7.5.0` (version 7.5.0), `lapack@3.9.0`, `openblas@0.3.9`, `python-cython@0.29.21` and `python-pytest@5.3.5`
    -   e.g. `r-xml` is the Guix package for the R [xml](https://cran.microsoft.com/web/packages/XML/index.html) package, which (as of this writing, at version 3.99-0.5) depends on libxml2@2.9.10, pkg-config@0.29.2 and zlib@1.2.11, but these libraries are managed by Guix in the same way as any other dependencies
    -   the philosophy of Guix is really to manage as many dependencies as sensible, e.g. `emacs-projectile` is the Guix package for the [projectile](https://docs.projectile.mx/projectile/index.html) package of the [GNU Emacs](https://www.gnu.org/software/emacs/) text editor
    -   Guix can also manage R itself as a package, so the R version can also be managed just as any other packages in your project
    -   therefore, Guix can handle dependencies across multiple programming languages, and mixing with system level dependencies
-   all Guix packages are put in `/gnu/store`, with a path having the name and a hash, e.g. `/gnu/store/9naz5xl42amla3ph860yxxqrk9420nvr-r-tidyverse-1.3.0` for the `r-tidyverse` currently on my system
    -   this store is only modifiable by the Guix daemon, and are read-only for normal users
    -   by virtue of the nice properties of the hash, this path serves as a unique identity of package, even if they have the same version number
        -   e.g. currently on my system, I find three Python 3.8.2 packages with different hashes, which are probably dependencies of other packages, and are built with slightly different settings:
            -   `/gnu/store/09a5iq080g9b641jyl363dr5jkkvnhcn-python-3.8.2`
            -   `/gnu/store/jxx8fr78jrcvpid5aplmkplbm1dk6czs-python-3.8.2`
            -   `/gnu/store/q9rm8h9imazsq2c4qiv2yjpvlvliywqb-python-3.8.2`
    -   therefore, the exact same package (as identified using the path) can be shared, while different versions (even with the same version number) can coexist
    -   also, when installing packages, if the exact package is also in the store, it need not be downloaded/built again
-   Guix can manage per-project dependencies, similar to a virtual environment or a per-project lockfile
    -   a list of packages can be recorded in a _manifest_ file, which is a plain text file that can be easily version-controlled
    -   e.g. a manifest file for some R packages may look like this (this is in fact [Scheme](https://en.wikipedia.org/wiki/Scheme%5F(programming%5Flanguage)) code, because Guix is implemented as a _domain specific language_ in [Guile](https://www.gnu.org/software/guile/) implementation of Scheme):

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
    -   the manifest file is declarative, and simple enough that it can be easily maintained even by manually editing as needed
    -   note that the manifest file only contains the names of the packages, but not the explicit versions
        -   so by itself, it cannot pin-point the exact versions of the packages (and their dependencies)
        -   each (specific version) of package is described by a package definition
            -   e.g. the package definition for `r-xgboost` (version 1.2.0.1) is:

                ```scheme
                (define-public r-xgboost
                  (package
                    (name "r-xgboost")
                    (version "1.2.0.1")
                    (source
                     (origin
                       (method url-fetch)
                       (uri (cran-uri "xgboost" version))
                       (sha256
                        (base32
                         "16hpvv2hwdzcyg90z7c1g5d2hj011qk8mivy4l2nqd2g7rkjwis4"))))
                    (build-system r-build-system)
                    (propagated-inputs
                     `(("r-data-table" ,r-data-table)
                       ("r-magrittr" ,r-magrittr)
                       ("r-matrix" ,r-matrix)
                       ("r-stringi" ,r-stringi)))
                    (native-inputs
                     `(("r-knitr" ,r-knitr)))
                    (home-page "https://github.com/dmlc/xgboost")
                    (synopsis "Extreme gradient boosting")
                    (description
                     "This package provides an R interface to Extreme Gradient Boosting, which
                is an efficient implementation of the gradient boosting framework from Chen
                and Guestrin (2016).  The package includes efficient linear model solver and
                tree learning algorithms.  The package can automatically do parallel
                computation on a single machine.  It supports various objective functions,
                including regression, classification and ranking.  The package is made to be
                extensible, so that users are also allowed to define their own objectives
                easily.")
                    (license license:asl2.0)))
                ```
            -   we see that it contains information on:
                -   where to fetch the source files: `(uri (cran-uri "xgboost" version))`
                -   the sha256 hash of source file: the `(sha256 (base32 ...))` part to aid reproducibility
                -   the way to build the package: `r-build-system` is used here for a typical R package
                -   the built-time dependency: ``(native-inputs `(("r-knitr" ,r-knitr)))``
                -   the run-time dependencies: `(propagated-inputs ...)` or `(inputs ...)` (see <https://guix.gnu.org/manual/en/guix.html#package-Reference> for details of the difference between the two)
                -   other auxiliary information which are not really essential, but nice to have
            -   the set of all these package definitions is managed with git repository
                -   they are managed as _channel_, which is git repository with some meta-data
                -   there is an official repository at <https://git.savannah.gnu.org/git/guix.git>
                -   user can create their own repository for their own modification or private packages that they would not like to share with outsiders
                -   the particular commit(s) of the channel(s) in your system _determine_ which exact versions of the packages when installed, reproducibly
            -   the current commit(s) of channel(s) can be exported to a plain text file by Guix, e.g.

                ```scheme
                (list (channel
                        (name 'guix)
                        (url "https://git.sjtu.edu.cn/sjtug/guix.git")
                        (commit
                          "2283baae907f4f38a8299d47ba4c0b2b49222883")
                        (introduction
                          (make-channel-introduction
                            "9edb3f66fd807b096b48283debdcddccfea34bad"
                            (openpgp-fingerprint
                              "BBB0 2DDF 2CEA F6A8 0D1D  E643 A2A0 6DF2 A33A 54FA")))))
                ```
            -   Guix also provides _time-machine_ to conveniently use particular commit(s) of channel(s) (in the form as exported above) for any Guix actions, see [Invoking guix time-machine](https://guix.gnu.org/manual/en/guix.html#Invoking-guix-time%5F002dmachine)
            -   therefore, by keeping two easily version-controlled plain text files (the manifest file for packages and the exported channel description), a set of particular versions of packages can be recorded reproducibly
    -   there are a few ways to use the manifest file (and can be used together with channel description through time-machine for maximum benefits), in fact a manifest file is only a convenient way to specify a list of packages, so can be used for different commands of Guix:
        -   install the specified packages in one transaction (on the default profile of the user):
            -   this is inconvenient for managing different sets of packages for different projects, because the actions are applied to the default profile of the user
            -   there are better ways for managing per-project packages
        -   a better way is to install the specified packages in a separate profile for the project:
            -   each user can create multiple profiles in addition to the default one, and each profile can have a separate set of packages and its own sequence of generations
            -   each profile can be activated as needed, much like a virtual environment in Python
            -   the Guix install/upgrade/remove actions can be applied to specified profile, see the `-p` option of [Invoking-guix-package](https://guix.gnu.org/manual/en/guix.html#Invoking-guix-package) for details
            -   but the downside is that when the set of packages in the manifest file is changed, the user need to remember to reapply the manifest file to the chosen profile to update the set of packages
            -   if the packages for each project are rarely changed, using profiles can be a reasonable way of managing per-project packages, but there is still a better way
        -   spawns a new shell with the specified packages accessible in the `PATH`, by using `guix environment`:
            -   `guix environment` is a powerful and flexible command in Guix
                -   we can choose to have the packages themselves be accessible in the shell, by placing the list of packages or using the `-m` option for manifest file after the `--ad-hoc` option
                -   or we can choose to have the direct dependencies of the packages be accessible, or both
                    -   having the dependencies be accessible is useful for developing a package, because we may need the dependencies for testing the building of the package and tweaking as needed
                -   we can either have an interactive shell with the needed packages accessible,
                -   or we can directly execute a command in the new shell by placing the command (and the arguments) after a `--` at the end of the list of packages
                -   can choose to have only the specified packages be accessible in the new shell by using the `--pure` option, the default is to augment current `PATH`
                -   can even choose to run the command inside an isolated container, which uses the isolation capability of the Linux kernel in a similar way to docker containers
                -   of course this can be combined with `guix time-machine`
                -   `guix environment` is basically creating a temporary profile, so would not "pollute" or clutter the default profile when the shell exits or the specified command ends
                -   the specified packages will be downloaded/built if they are not already in the cache
                -   see [Invoking-guix-environment](https://guix.gnu.org/manual/en/guix.html#Invoking-guix-environment) for more details of other useful options
            -   therefore, we can spawn a new shell with needed packages each time we work on a project, we won't forget to reapply the manifest file
            -   e.g. to work on a project, suppose in a R project directory
                -   there is a manifest file `pkgs.scm` specifying the packages (including R itself),
                -   and there is an exported channel description file `channels.scm` containing the commits of the channels
                -   when we want to work on the project, we can type in the shell

                    ```shell
                    guix time-machine -C channels.scm -- environment --ad-hoc -m pkgs.scm
                    # then in the new shell, start R
                    R
                    ```
                -   or you can do it in one line:

                    ```shell
                    guix time-machine -C channels.scm -- environment --ad-hoc -m pkgs.scm -- R
                    ```
                -   the first time running this may take a while if the packages are not yet downloaded/built in your system, but subsequent runs should be much quicker because the packages are already cached
                -   moreover, these `pkgs.scm` and `channels.scm` files can be committed to version control system (e.g. git) together with other project files
                    -   they are project dependencies, which are logically part of the project, so in my opinion should also be committed
                    -   then everyone working on the project would be using the same versions of packages for the same commit
                    -   if these two files are changed, the correct versions of the needed packages would be prepared by Guix the next time you work on the project
                -   of course, typing this long line each time is tiresome, so we can make an alias in your shell (e.g. by adding this to your `.bashrc` or `.bash_profile`) to reduce some typing, e.g.

                    ```shell
                    alias work="guix time-machine -C channels.scm -- environment --ad-hoc -m pkgs.scm"
                    ```

                    then you can just type

                    ```shell
                    work -- R
                    ```
                -   working with Python project, or any other language is similar, just make sure you have the needed packages in the `pkgs.scm` file.
                -   one thing that I have not yet investigated deeply enough:
                    -   of course, we usually use some development tools such as IDE or text editor that are usually personal preferences, so they should not be put into `pkgs.scm`, but should better be installed in the default user profile
                    -   e.g. for R, common choices are [RStudio](https://rstudio.com/), [Emacs ESS](https://ess.r-project.org/) with [ESS](https://ess.r-project.org/), [Vim](https://www.vim.org/), [VSCode](https://code.visualstudio.com/)
                    -   you would need some way to make these tools see and use the R in the newly spawned shell, which may need some tweaking, but I have not yet spent the time to investigate this
                    -   we will explore this further in a future post in this series.
            -   e.g. another use is in running batch jobs, e.g. a [Jenkins](https://www.jenkins.io/) job:
                -   suppose we have the same `pkgs.scm` and `channels.scm` in the project root
                -   also suppose we want to batch run the R script `myscript.R`
                -   assuming we have the same alias as above, we can run the script as (or config the Jenkins job to run this)

                    ```shell
                    work -- Rscript myscript.R
                    ```
                -   of course you can run any command by putting them after `--`, and they would be run in the newly spawned shell
                -   e.g. to run another shell script containing many commands

                    ```shell
                    work -- sh myscript.sh
                    ```
                -   again, using Guix ensures that the correct versions of the needed packages are there when needed, this is especially convenient when you need to run code on multiple machines, e.g. on multiple Jenkins nodes, and saves you the trouble of manually managing the dependencies on multiple machines for multiple projects.
            -   it should be clear that Guix can provide at least the same virtual environment like functionality, if not more useful and convenient


## Avoid dependency hell by including all dependencies {#avoid-dependency-hell-by-including-all-dependencies}

-   including all dependencies is also a valid way to avoid dependency hell
-   this has been common practice in Windows or MacOS for many years
-   in Linux, similar strategy has become more common in recent years, e.g (among others).
    -   [flatpak](https://flatpak.org/) provides bundled dependencies and sandboxing. This is mainly for application distribution
    -   [AppImage](https://appimage.org/) aims at providing a universal format for application that can run in different Linux distributions, with dependencies included. This is mainly for application distribution
    -   [docker](https://www.docker.com/) provided bundled dependencies in an _docker image_, and can run programs in an isolated (optionally for network and file system access) _container_. This is commonly used for deploying application and preparing a consistent development environment.
        -   docker images are often built using a _dockerfile_ which is a text file with imperative directives
        -   docker images are often built on top of another base image which already has included a lot of needed parts, so that by choosing (or first building) a suitable base image, images can often be built with a relatively simple dockerfile.
        -   once a docker image is built, the same running environment can be easily reproduced by starting another container using the same image
        -   docker images are organized in layers, where identical layers can be shared between images
            -   roughly, each directive in the dockerfile creates a layer
-   I will focus more on docker, as I do not have much experience with either flatpak or AppImage, but I think the comments apply similarly.


### downsides: {#downsides}

-   although once the package or docker image is built, the user should have little trouble with dependencies, these tools do not provide much help with building the package or docker image
-   in order to _build_ a docker image _reproducibly_ through dockerfile, great care must be exercised:
    -   a dockerfile often starts from a base image (e.g. from [docker hub](https://hub.docker.com/)) by a name and a tag, but the image itself may have been updated inbetween builds
        -   if the base image is something like `ubuntu:latest` which is meant to be the most updated version, it is normal that different builds using the same dockerfile would see different base image
    -   most dockerfiles would then update the package manager repository of the base image, e.g. `apt-get update` for Debian based distributions
        -   this is usually done to get security updates of the packages
        -   therefore, at different times, the exact versions of the packages may be different
    -   then it is usual to install some package through the distribution package manager of the base image, or some language specific package manager
        -   if the package versions are not carefully pinned, different versions may get installed at different times
    -   if some packages are built from git repository, then a particular commit need to be used, otherwise, the latest version at the time of the build will be used
    -   basically, dockerfile itself does not provide much help for reproducible build, it is up to the writer of the dockerfile to ensure as much reproducibility as possible
-   since the layers are organized linearly, the potential sharing is limited unless the dockerfiles of the different images are carefully organized to have as much in common as possible from the top
-   if some packages are built in building the docker image, some files or packages that are only needed in build time may be left over in the final image, if not carefully removed, making the image larger than necessary
    -   see for example [Multi-stage builds \\#1: Smaller images for compiled code](https://pythonspeed.com/articles/smaller-python-docker-images/) and [Smaller Docker Image using Multi-Stage Build](https://medium.com/the-artificial-impostor/smaller-docker-image-using-multi-stage-build-cb462e349968) for using multi-stage build to reduce docker image size
-   of course, sometimes the intention is simply to build the latest version of some packages, without reproducibility requirements, then dockerfile is sufficient


### upsides of Guix: {#upsides-of-guix}

-   strong reproducibility guarantee
    -   package definition contains the hash of source file for package building
    -   isolated environment is used for package building, to make the build process as deterministic as possible
    -   therefore need only use the same channel description file and Guix time machine to get the exact set of package definitions
    -   together with a manifest file of desired packages, the desired set of packages and their dependencies can be exactly reproduced easily
-   Guix can be used to produce docker image
    -   Guix can pack a set of packages and their dependencies (and nothing more, e.g. no leftover files only for building some packages) into various different formats, e.g. docker image, tarball and squashfs image, see the `-f` option in [Invoking guix pack](https://guix.gnu.org/manual/en/guix.html#Invoking-guix-pack) for details
        -   Guix can also build a vm-image, disk-image or docker image through `guix system vm-image`, `guix system disk-image` or `guix system docker-image` respectively, see [Invoking guix system](https://guix.gnu.org/manual/en/html%5Fnode/Invoking-guix-system.html#Invoking-guix-system) for details
    -   of course, this can be combined with Guix time machine through a channel file, and to use manifest files for the set of packages wanted
    -   therefore, we can use Guix to produce docker image instead of using dockerfile to enjoy better reproducibility, and continue to use the surrounding infrastructure built around docker images, e.g. [Kubernetes](https://kubernetes.io/)
    -   therefore, Guix can be used in a complementary way to docker, if you do not wish to completely replace docker with Guix
-   Guix also allows running programs in container:
    -   by using the isolation capability of the Linux kernel, Guix environment allows running programs in container basically in the same way as docker
    -   the user can also control the sharing of current working directory and the network
    -   see the `--container` or `-C` option and other related options in [Invoking guix environment](https://guix.gnu.org/manual/en/guix.html#Invoking-guix-environment) for details, but note that it requires a Linux kernel at least as new as version 3.19
-   more fine grained sharing
    -   in Guix, each exact version of each package is identified by a hash (together with package name and version number), and identical packages are cached in the store and easily shared, either directly needed, or indirectly needed as a dependency of another package
    -   this sharing is automatic, so does not need careful arrangement of the user, and much more fine-grained than simple layer sharing in docker image
    -   there has been discussion among Guix developers to extend the sharing to individual files, so that identical files in different packages can also be shared


## Use another functional package manager such as Nix {#use-another-functional-package-manager-such-as-nix}

-   [Nix](https://en.wikipedia.org/wiki/Nix%5Fpackage%5Fmanager) originated from Dolstra, E.'s PhD research [The Purely Functional Software Deployment Model](https://nixos.org/~eelco/pubs/phd-thesis.pdf), and refers to two related things:
    -   Nix is a dynamically typed, functional programming language with lazy evaluation, designed for implementing the Nix functional package manager
    -   Nix is a functional package manager implemented using the Nix programming language
        -   Nix can be used on Linux and MacOS
        -   Guix is inspired by Nix
        -   basically Guix is a re-implementation of the Nix package manager using Guile instead of the Nix programming language, and in the process can correct some crufts and add some improvements
-   [NixOS](https://nixos.org/) is a Linux distribution based on the Nix package manager, in the same way that Guix system is a Linux distribution based on the Guix package manager
-   Nix is also a fine choice of functional package manager


### upsides of Nix / downsides of Guix: {#upsides-of-nix-downsides-of-guix}

-   while Guix is only available in Linux, Nix is available in both Linux and MacOS. However, the supporting and testing of packages in MacOS may not be as good as in Linux
-   since Nix was developed (first around 2004) much earlier than Guix (first release in 2013, see [GNU Guix Releases](https://en.wikipedia.org/wiki/GNU%5FGuix#Releases)), Nix has more packages


### downsides of Nix / upsides of Guix: {#downsides-of-nix-upsides-of-guix}

-   the Nix package manager is implemented in the Nix programming language, which is a very specialized and niche programming language because it is not used in other contexts or purposes
    -   on the other hand, Guix is implemented in Guile, which is a dialect of Scheme. Scheme is a general purpose programming language useful in other contexts
    -   of course, for simple use, the users of Nix need not know much about the underlying implementation language, just as Guix users need not know Guile for simple uses
    -   but for more complicated uses, some knowledge of Nix (respectively Guile in the case of Guix) is needed, in which case learning a general purpose language may be considered more useful than learning a niche language, and the tools normally used for Scheme (e.g. editor support, REPL, debuggers) can be used for Guix hacking
    -   also, the data structures available in the Nix language is more limited, whereas Guix can use distinct data structures for different kinds of objects
        -   e.g. in Nix, each package is represented as a _function_ that takes inputs from a large mapping; but in Guix a package is represented as a first-class object in Scheme, forming an explicit dependency graphs, which easily allows different kinds of processing on the dependencies (e.g. substitute a particular package with a modified one; or plotting the dependency graph for visualization)
-   Guix was developed much later than Nix, and therefore has a chance to improve on some early decisions of Nix after some years of real world usage experience
    -   e.g. the use of richer Scheme data structures in Guix for different kinds of objects
    -   e.g. the rough equivalent to `guix environment` in Nix is [nix-shell](https://nixos.wiki/wiki/Development%5Fenvironment%5Fwith%5Fnix-shell), which is originally intended to provide a shell where the build-time dependencies of a package is available for development and testing
        -   if you want to have some packages available in the new shell, you can define a dummy package with the desired packages as build-time dependencies
        -   in contrast, `guix environment` allows specifying the list packages of which their dependencies are wanted, or the list of packages which themselves are wanted (the `--ad-hoc` option), or a mixture of the two
            -   apparently `nix-shell` now also allows this: <https://nixos.org/manual/nix/unstable/command-ref/nix-shell.html>
        -   also, to my knowledge `nix-shell` does not have the convenient time machine as `guix time-machine` to pin-point the set of packages at particular commit of channels, but similar effect could be achieved by some Nix constructs in the `shell.nix` or `default.nix` file manually
            -   see [Pinning Nixpkgs](https://nixos.wiki/wiki/FAQ/Pinning%5FNixpkgs) for examples of pinning versions of packages
-   Nix relies more heavily on the shell for various build steps and occasionally causes trouble due to the escaping and substitutions in the shell, while Guix relies mostly on Scheme, so does not have issues with escaping

<!--listend-->

-   see the following for more comparison between Nix and Guix:
    -   <https://news.ycombinator.com/item?id=16490027>
    -   <https://sandervanderburg.blogspot.com/2012/11/on-nix-and-gnu-guix.html>
    -   <https://www.reddit.com/r/GUIX/comments/hxcq7d/guix%5Fvs%5Fnix/>
-   in short, both Nix and Guix are fine choices of functional package manager, and share a lot of similarities


## What's next? {#what-s-next}

In this part we compared Guix with other solutions, and showed that
Guix has various advantages. Next time we show how you can try out
Guix by various ways of installing Guix, either in a physical or
virtual machine, either as a complete GNU/Linux distribution, or just
as a package manager on top of another Linux distribution.
