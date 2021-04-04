+++
title = "Guix to manage project dependency"
date = 2021-01-02
tags = ["Guix", "Functional Package Manager", "Reproducibility"]
categories = ["Guix"]
draft = true
author = "Peter Lo"
+++

This is a brief introduction to the Guix functional package manager,
and how it could be used to manage dependencies of projects, much like
virtual environments for Python, but with much larger scope.


## What is Guix? {#what-is-guix}


### Guix: functional package manager {#guix-functional-package-manager}

Guix has the following characteristics:

-   [Guix](https://guix.gnu.org/) is in fact a package building system, being a good package manager is a side effect
    -   inspired by the [Nix](https://nixos.org/) functional package manager
        -   which is in turn partly inspired by functional programming, and by [Gentoo](https://wiki.gentoo.org/wiki/Main_Page)'s package management
    -   to make the package building be a _pure function_ and therefore more easily reproduced
-   designed to properly address the "dependency hell" problem
-   aim at reproducible builds
    -   the _exact_ same set of packages could be reproduced at a later time or on a different machine (of the same architecture), by just using two small text files
-   allows atomic installation/uninstallation/upgrade of packages
    -   each action is a **transaction**, i.e. either it succeeds as a whole, or does not succeeds, there would not be half installed packages
-   allows easy rollback of install/uninstall actions
    -   each transaction results in new generation
    -   can easily rollback to previous generations
    -   so there is little fear of accidentally installing/uninstalling/upgrading the wrong packages
-   allows different versions of the "same" package to coexist and used by different pakcages, without conflict
-   encourages declarative package management
    -   a set of packages can be specified in a **manifest file**, and can be installed in one transaction
-   currently only works on GNU/Linux
    -   can be installed on any GNU/Linux distribution such as Debian, Ubuntu, Arch, etc
    -   can coexist with the existing package manager of the distribution
-   allows each user to manage his/her own packages
    -   without root privilege
    -   without interferring other users
-   each user can have multiple **profiles** of packages
    -   each profile has its own list of generations, and can be rolled back separately
-   allows easy creation of isolated environments with designated packages
-   **Guix system** is a GNU/Linux distribution built on top of the Guix package manager
    -   use a config file to declaratively specify the whole system, e.g. the system services, user accounts, etc

At this point, you may not think these characteristics are special if
you have not experienced the pains brought by dependency
management. Hopefully after reading this post you will have better
appreciation of why a package manager is useful and why Guix is a good
one due to these good characteristics.


### Basic concepts of packages and dependency {#basic-concepts-of-packages-and-dependency}

We first list out some concepts related to packages and dependency:

-   **package**:
    -   loosely speakinig, a package is a collection of functionality that we are interested in, e.g.
        -   a library of functions, e.g. the [tidyverse](https://www.tidyverse.org/) R package
        -   some applications, e.g. the [firefox](https://www.mozilla.org/en-US/firefox/browsers/) web browser
        -   command line programs, e.g. the [ripgrep](https://github.com/BurntSushi/ripgrep) command for fast grep search on contents of files
        -   or just a set of documentation files
-   **package building**:
    -   the final files in a package are often produced through a _building_ process from some sort of _source_ files
    -   e.g.
        -   `ripgrep` is written in the [Rust programming language](https://www.rust-lang.org/), and would need a Rust compiler to produce the program
        -   some pdf documentations may be produced through the [Latex](https://www.latex-project.org/) typesetting system
        -   an R package needs to be built, see <https://bookdown.org/rdpeng/RProgDA/building-r-packages.html>
    -   some packages are very flexible, and allow specifying different options when building, resulting in functionally slightly different packages
        -   e.g. some functions may be left out if it is considered not useful
        -   e.g. some function could be provided by different **dependent packages**
    -   building script:
        -   most packages have some building process (e.g. a script or a makefile)
    -   building environment:
        -   some packages will automatically detect its environment to change building options
        -   e.g. if some other package is absent, some functionality will be left out, but the building will still succeed
        -   therefore, if the building environment is not properly isolated, the "same" building script may still result in different built package, depending on what other packages are present in the system
        -   this is similar to programming where the output of a "function" depends not only on its inputs, but also on other global variables
-   **package version**:
    -   strictly speaking, package version is just a name attached to a package, and the name could theoretically be arbitrary, although most package authors follow some conventions
    -   e.g. a lot of packages follow the [Semantic versioning](https://semver.org/), briefly
        -   a version number consists of three numbers, in the format of "MAJOR.MINOR.PATCH", e.g. "1.0.4", where each should be increment as follows
            -   MAJOR: when you make incompatible API changes, where MINOR and PATCH could be reset to 0, e.g. from "1.2.3" to "2.0.0"
            -   MINOR: when you add functionality in a backwards compatible manner, where PATCH could be reset to 0, e.g. from "1.2.3" to "1.3.0"
            -   PATCH: when you make backwards compatible bug fixes, e.g. from "1.2.3" to "1.2.4"
    -   NOTE:
        -   normally, different versions will have some differences, although they may be small, or supposed to be _backward compatible_.
        -   even for the same vesion number, the package may be functionally different if built differently, as mentioned above.
-   **dependency**:
    -   loosely speaking, if a package A needs another package B to provide some functionality, then A _depends_ on B, i.e. B is a _dependency_ of A.
    -   **direct/indirect dependency**:
        -   often a package will list out its needed dependencies (possibly also the range of allowed versions of each dependency), either formally in some fixed format, or informally as free text in some readme
        -   **direct dependency**: the dependencies listed for a package
        -   **indirect dependency**: not direct dependency, but are (direct or indirect) dependencies of the direct dependencies
        -   Why the distinction of direct and indirect dependency? Both are needed to fully capture the dependencies.
            -   the distinction is useful mainly when the dependency updates
            -   if package A directly depends on package B, presumably the developers of A knows which functionality of B is needed
            -   if B is updated to B1, then the developers of A need only check whether the needed functionality is still provided by B1, and act accordinly, rather than checking each of the dependencies of B and B1 to see which are still needed.
    -   **build-time/run-time dependency**:
        -   **build-time dependency**: dependency needed for building a package
            -   e.g. a particular version `gcc` for compiling a program
            -   e,g, **statically linked libraries**, i.e. those compiled into the program, so are needed at build-time
            -   a build-time dependency may or may not be needed when the program is later run
        -   **run-time dependency**: dependency needed for using the package, e.g. running the application
            -   e.g. the **dynamically linked libraries**, i.e. the libraries will be loaded only when the program is run
            -   nowadays, most programs use mostly dynamically linked libraries
            -   NOTE: a dependency can be both build-time and run-time dependency
    -   **optional dependency**: dependency that can be omitted for the package to build or run, but some functionality may be missing
        -   e.g. [inkscape](http://www.inkscape.org/) can be built without the optional dependency `potrace`, just without bitmap tracing functionality.
        -   in most **package manager** which mainly distribute binary packages, often most optional dependencies would be included to provide the most funcitonality
-   **dependency hell**:
    -   roughly speaking, [dependency hell](https://en.wikipedia.org/wiki/Dependency_hell) refers to the problems caused by the dependency on specific versions of some packages.
    -   dependency hell takes a few forms:
        -   too many or long chains of dependencies:
            -   this is only a problem if the dependencies have to be hunted down manually, which could become tedious very quickly
            -   most package managers solve this by installing the dependencies when a package is installed
        -   conflicting dependencies:
            -   in many package manager (and default in dynamic library in Linux), minor versions are considered backward compatible, and for each package of the same major version, only the newest minor version is kept/used
            -   if both package A and B depend on a package C, but A and B needs different minor versions of C to work correctly, then A and B have conflicts
            -   this may happen if B is updated to B1 causing C to be updated to C1, therefore causing A to break, even if the older versions of the 3 packages previously coexisted and worked correctly.

                {{<figure src="/ox-hugo/dep_before_update.png" caption="Dependency before updating B">}}

                {{<figure src="/ox-hugo/dep_after_update.png" caption="Dependency after updating B">}}

            -   in this case, it is clear that if we just let A to use the old C, and the new B1 to use the new C1, then A can work as before, and B can still be updated to B1.

                {{<figure src="/ox-hugo/dep_ideal_after_update.png" caption="Ideal dependency after updating B">}}

-   ways that code of package A can break if a dependency B updates to a supposingly _backward compatible_ minor version:
    -   although most of the time updating a minor version does not cause problem, they might still cause breakage
    -   e.g. suppose A depends on a function in B, there could be a few cases:
        -   the function interface remains unchanged or adds optional parameters, but the implementation is changed:
            -   A may rely on undocumented behavior of the function, which has changed in the new implementation, although the documented interface is still the same.
                -   e.g. the old implementation may sort the output as a side effect, but not promised in the function interface, and A may have relied on the sorted order
            -   the new implementation may have buggy edge case, causing A to break
            -   the new implementation may expose a buggy edge case in A, causing A to break
-   **reproducible build**
    -   it is desirable to have the _exact same versions_ of dependencies between testing and production systems, and preferably also for the development environment
    -   it is therefore desirable to _reproduce_ the exact same set of packages on a different machine (of the same architecture) and/or at a different time
    -   this could be achieved in two main ways:
        -   record the set of versions of the (pre-built) packages, and reinstall when needed
            -   e.g. python virtual environment mostly follow this paradigm
        -   record the set of versions of the packages, and rebuild then when needed
            -   this is similar to the previous one, with the difference that the package can be built from scratch if needed
            -   e.g. Guix can rebuild package(s) through the set of package definitions with explicit dependencies information
                -   Guix can simply download the pre-built package (called _substitute_ in Guix) when available
        -   record the set of built packages and just copy them as a whole when needed
            -   e.g. building a docker image to contain all needed packages
    -   isolated building environment can help with reproduciblity
        -   only the explicitly listed dependencies are visible in building, so that the building script will not depend on other packages unknowingly
-   reproducibility raises the question of _sameness_ of packages
    -   the package name with version number _would be_ sufficient if each version if always built the same way with the same versions of dependencies
        -   this is the strategy adopted by most package managers
    -   a better way is to use the package name together with some kind of **hash**
        -   not necessarily the hash _of_ the package itself, as we will see in later sections
        -   but different contents of a package should produce different hashes
-   **hash**:
    -   basically a (very large) integer calculated through a **hash function** on some input, e.g. a file
    -   the calculated integer is in some fixed range, often written as a long hexadecimal string such as "730e109bd7a8a32b1cb9d9a09aa2325d2430587ddbc0c38bad911525"
    -   the input however often has not length limit
    -   e.g. [sha-256](https://en.wikipedia.org/wiki/SHA-2), [md5](https://en.wikipedia.org/wiki/MD5)
    -   desired properties of a good **hash function**:
        -   the same input always produce the same hash, i.e. it is a _pure function_ in the mathematical sense
            -   i.e. if two inputs produce different hashes, they _must be_ different
        -   it is one-way
            -   there is not efficient way to recover the original content just from the hash, other than trying all possible input to find those that give the same hash
        -   even a slight change in the input causes drastically different hash
            -   useful for identifying corruption or tampering of files
        -   _low_ collision, i.e. different inputs _should_ produce different hashes
            -   it is impossible to have _no_ collision unless the set of possible inputs is less than the set of possible outputs
    -   note that one hash is enough to represent a web of connected things:
        -   e.g.
            -   if you have a few (ordered) inputs, you hash each of them, and write the hashes to a file, then you hash this file
            -   this is still a deterministic hashing process
            -   if any of the input is changed, its calculated hash will _most probably_ be different, so this file of hashes will be different, and consequently the final hash will be different
            -   each of the input itself could contain hashes of more inputs recursively, so a web of things could be represented as one hash
            -   this technique is also used in [git](https://git-scm.com/) version control system to link the commits together
            -   the same technique could be used to hash the (direct) dependencies of a package, and therefore one hash could represent all the direct and indirect dependencies of a package


## Why bother with Guix? There are other ways to handle dependency {#why-bother-with-guix-there-are-other-ways-to-handle-dependency}

Although there are a lot of package managers at both the language level and operating system level, and tools such as docker to help tackle the dependency problem, Guix still has merits when compared to these, as we now discuss.


### Use opearting system package managers {#use-opearting-system-package-managers}

-   e.g. apt, yum, pacman
-   these package managers resolve dependencies for you, some allows pinning particular versions of package
-   sometimes language specific packages is also packaged
-   downsides:
    -   these package managers mostly operate by _mutating_ the system state when adding or removing packages, so may not be easy to revert to previous state in case installing/upgrading some packages causes problem
        -   if the process is interrupted, the system might be in an inconsistent state
        -   installing/upgrading package may cause some dependencies to be updated, which might break other packages
        -   subsequently removing that package does not necessarily revert the updated dependencies
    -   since installing/upgrading package may cause dependencies to be updated, which may cause conflicts if some packages need older versions of the dependencies
        -   although this is not common, this could be painful when happens
        -   this is problematic often when you want newer version of some package, but older version of another package, and they (or their chain of dependencies) somehow have version conflicts
    -   since the packages are installed to some shared location, usually root privilege is needed to install/upgrade/remove packages, e.g. through `sudo`
        -   this is not much of a problem on desktop Linux system because it is often used by one person
        -   it is more of a problem on servers with multiple users, either we trust all users and give them root privilege (through `sudo`), or we reserve the root privilege to a small number of users (e.g. the system admins)
        -   and the possibility of possible breakage and conflict when upgrading packages only increases with the number of users
        -   therefore, on such systems, some users may not get the packages he/she wants, either it is too new or too old, because the updating schedule is often set by the system admins
        -   package managers such as [Homebrew](https://brew.sh/) try to remedy this short-coming by installing packages under the user's home directory, and therefore each user can manage his/her own packages without root privilege
            -   although this reduces possible conflicts with other users, there are still possible conflicts among the packages
-   upsides of Guix:
    -   Guix can be used just like the usual system package manager, although there are better ways of using it for specific needs
    -   every install/remove/upgrade packages action (which may involve multiple packages) is one _transaction_, and causes a _generation_ to be created
        -   Guix can detect and remedy interrupt of the action to maintain transactional behavior, i.e. either it succeeds or fails as a whole, so that the system would not be in a half-completed inconsistent state.
            -   if an action is interrupted, it has not succeeded, and can be safely repeated
            -   but in case an action involves multiple packages, a completely downloaded or built package need not be re-built or re-downloaded when the action is repeated, since they would be cached (in a _store_ where only the Guix daemon can modify) and identified by hash.
        -   a _generation_ is essentially a record of the set of specific packages and their dependencies, so it is easy to revert to previous generation
            -   all the packages and dependencies referenced by a generation would be kept in the system
            -   so normally removing packages in Guix simply results in a generation without those packages, the packages themselves are still in the system cache, this is similar in spirit to git where a commit "deletes" some files.
            -   reverting to other generation involves only updating some symbolic links, so are quick
            -   the user safely try out different versions of some packages, knowing that it is easy to revert to previous known good state if the user dislike the version for whatever reason (e.g. bugs, different UI, missing features, etc)
            -   the user can optionally delete older generations, and do a _garbage collection_ to really delete any unreferenced (directly or indirectly) packages to free up disk space
    -   each package in Guix literally has its dependencies hard-coded using absolute paths to the dependeny in the _store_
        -   each package in Guix is cached in a _store_ with a path with some sort of hash to identify the exact version of the package
        -   therefore an updated package may also have updated dependencies, but older versions of the package or other packages still refer to their previous versions of dependencies fixed at built time
        -   therefore there is no fear that updating a package will break another package just because they share some dependencies with conflicting versions
        -   also, two packages with conflicting dependencies can coexist in Guix because each can have their own versions of dependencies
    -   Guix allows each user to manage his/her own packages without root privilege
        -   the _store_ of installed packages in Guix is managed by a dedicated daemon, so are essentially read-only to the users, so identical packages can be safely shared
        -   each user has his/her symbolic links under the home directory to profiles containing sets of packages, so any install/upgrade/remove actions can be performed by the user without root privilege
        -   and as mentioned above, because of hard-coded dependencies for each package, there will not be package conflicts among users nor for the same user


### Use language specific package manager such as pip, packrat, npm, etc {#use-language-specific-package-manager-such-as-pip-packrat-npm-etc}

-   many programming languages have their own package manager, because the system package manager may not have these language-specific packages, and having a language specific one would be more uniform across different operating systems or Linux distributions
-   e.g.
    -   pip for Python
    -   `install.packages()` for R
    -   npm for Javascript
    -   RubyGems for Ruby
    -   Cabal for Haskell
-   for better management of possibly difference packages for different projects, there are either some sort of _virtual environment_, or some kind of _lock files_ to pin-point the versions of set of packages for each project, e.g.
    -   [pyenv](https://github.com/pyenv/pyenv#simple-python-version-management-pyenv), [virtualenv](https://virtualenv.pypa.io/en/stable/), [anaconda](https://docs.continuum.io/anaconda/packages/pkg-docs/) for Python, see <https://stackoverflow.com/a/39928067> for a brief comparison
    -   [packrat](https://rstudio.github.io/packrat/) or [renv](https://rstudio.github.io/renv/) for R
    -   rubygems, npm and cabal have lock files
-   downsides:
    -   these language-specific package managers naturally only handles packages for on programming language
        -   if a project uses only one programming language, e.g. Python, then either one of the virtual environment manger may be sufficient
        -   but if the projects in the same team use multiple programming languages, e.g. both Python and R for data science projects, then the users would need to be familiar with multiple package managers
    -   these package managers may not help with system-level dependencies, especially when pre-built binary package is not available (e.g. R packages under Linux) and the package needs to be built
    -   some dependencies are not managed by these virtual environments
        -   e.g. packrat, being an R library, does not help manage the version of R itself
            -   although this is often ok because R is usually backward compatible, but sometimes there could be issues, e.g. see <https://github.com/rstudio/packrat/issues/327>
        -   in contrast, virutal environments in Python can also manage different versions of Python, because there are bigger differences between versions of Python
    -   the virtual environments are often setup per-project, but identical packages (and dependencies) may be duplicated instead of shared, taking up more disk space than necessary (unless the filesystem had built-in support for deduplication)
        -   e.g. packrat for R install a copy of the needed packages for each project
        -   in contrast, renv for R has a global shared cache of packages, so that identical packages can be shared for different projects, see <https://cloud.r-project.org/web/packages/renv/vignettes/renv.html>
-   upsides of Guix:
    -   Guix has system-level libraries, applications, language specific pakcages all at the same level, and can be managed in the same way.
        -   e.g. `r-tidyverse` is the Guix package for the R [tidyverse](https://www.tidyverse.org/) package, which depends on many other R packages, all of can also be managed by Guix
        -   e.g. `python-numpy` is the Guix package for the Python [numpy](https://numpy.org/) package, which depends on `gfortran@7.5.0` (version 7.5.0), `lapack@3.9.0`, `openblas@0.3.9`, `python-cython@0.29.21` and `python-pytest@5.3.5`
        -   e.g. `r-xml` is the Guix package for the R [xml](https://cran.microsoft.com/web/packages/XML/index.html) package, which (as of this writing, at version 3.99-0.5) depends on libxml2@2.9.10, pkg-config@0.29.2 and zlib@1.2.11, but these libraries are managed by Guix in the same way as any other dependencies
        -   the philosophy of Guix is really to manage as many dependencies as sensible, e.g. `emacs-projectile` is the Guix package for the [projectile](https://docs.projectile.mx/projectile/index.html) package of the [GNU Emacs](https://www.gnu.org/software/emacs/) text editor
        -   Guix can also mange R itself as a package, so the R version can also be managed just as any other packages in your project
        -   therefore, Guix can handle dependencies across multiple programming language, and mixing with system level dependencies
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
        -   a list of packages can be recorded in an _manifest_ file, which is a plain text file that can be easily version-controlled
        -   e.g. an manifest file for some R packages may look like this (this is in fact [Scheme](https://en.wikipedia.org/wiki/Scheme_(programming_language)) code, because Guix is implemented as a _domain specific language_ in [Guile](https://www.gnu.org/software/guile/) implementation of Scheme):

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
                -   Guix also provides _time-machine_ to conveniently use particular commit(s) of channel(s) (in the form as exported above) for any Guix actions, see [Invoking guix time-machine](https://guix.gnu.org/manual/en/guix.html#Invoking-guix-time_002dmachine)
                -   therefore, by keeping two easily version-controlled plain text files (the manifest file for packages and the exported channel description), a set of particular versions of packages can be recorded reproducibly
        -   there are a few ways to use the manifest file (and can be used together with channel description through time-machine for maximum benefits), in fact a manifest file is only a convenient way to specify a list of packages, so can be used for different commands of Guix:
            -   install the specified packages in one transaction (on the default profile of the user):
                -   this is inconvenient for managing different sets of packages for different projects, because the actions are applied to the default profile of the user
                -   there are better ways for managing per-project packages
            -   a better way is to install the specified packages in a separate profile for the project:
                -   each user can create multiple profiles in addition to the default one, and each profile can has a separate set of packages and its own sequence of generations
                -   each profile can be activated as needed, much like a virtual environment in Python
                -   the Guix install/upgrade/remove actions can be applied to specified profile, see the `-p` option of [Invoking-guix-package](https://guix.gnu.org/manual/en/guix.html#Invoking-guix-package) for details
                -   but the downside is that when the set of packages in the manifest file is changed, the user need to remember to reapply the manifest file to the chosen profile to update the set of packages
                -   if the packages for each project are rarely changed, using profiles can be a reasonable way of managing per-projet packages, but there is still a better way
            -   spawns a new shell with the specified packages accessible in the `PATH`, by using `guix environment`:
                -   `guix environment` is a powerful and flexible command in Guix
                    -   we can choose to have the packages themselves be accessible in the shell, by placing the list of packages or using the `-m` option for manifest file after the `--ad-hoc` option
                    -   or we can choose to have the direct dependencies of the packages be accessible, or both
                        -   having the dependencies be accessible is useful for developing a package, because we may need the dependencies for test building the package and tweak as neede
                    -   we can either have an interactive shell with the needed packages accessible,
                    -   or we can directly execute a command in the new shell by placing the command (and the arguments) after a `--` at the end of the list of packages
                    -   can choose to have only the specified packages be accessible in the new shell by using the `--pure` option, the default is to augment current `PATH`
                    -   can even choose to run the command inside an isolated container, which uses the isolation capability of the Linux kernel in a similar way to docker containers
                    -   of course this can be combined with guix time-machine
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
                -   e.g. another use is in running batch jobs, e.g. a [Jenkins](https://www.jenkins.io/) job:
                    -   suppose we have the same `pkgs.scm` and `channels.scm` in the project root
                    -   also suppose we want to batch run the R script `myscript.R`
                    -   assuming we have the same alias as above, we can run as (or config the Jenkins job to run this)

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


### Avoid dependency hell by including all dependencies {#avoid-dependency-hell-by-including-all-dependencies}

-   including all depedencies is also a valid way to avoid dependency hell
-   this has been common practice in Windows or MacOS for many years
-   in Linux, smilar strategy has become more common in recent years, e.g (among others).
    -   [flatpak](https://flatpak.org/) provides bundled dependencies and sandboxing. This is mainly for application distribution
    -   [AppImage](https://appimage.org/) aims at providing a universal format for application that can run in different Linux distributions, with dependencies included. This is mainly for application distribution
    -   [docker](https://www.docker.com/) provided bundled dependencies in an _docker image_, and can run programs in an isolated (optionally for network and filesystem access) _container_. This is commonly used for deployment and preparing a consistent development environment.
        -   docker images are often built using a _dockerfile_ which is a text file with imperative directives
        -   docker images are often built on top of another base image which already has included a lot of needed parts, so that by choosing (or first building) a suitable base image, images can often be built with a relatively simple dockerfile.
        -   once a docker image is built, the same running environment can be easily reproduced by starting another container using the same image
        -   docker images are organized in layers, where identical layers can be shared between images
            -   roughly, each directive in the dockerfile creates a layer
-   I will focus more on docker, as I do not have much experience with either flatpak or AppImage, but I think the comments apply similarly.
-   downsides:
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
    -   of course, sometimes the intention is simply to build the latest version of some packages, without reproducibility requirements, then dockerfiles is sufficient
-   upsides of Guix:
    -   strong reproducibility guarantee
        -   package definition contains the hash of source file for package building
        -   isolated environment is used for package building, to make the build process as deterministic as possible
        -   therefore need only use the same channel description file and Guix time machine to get the exact set of package definitions
        -   together with a manifest file of desired packages, the desired set of packages and their dependencies can be exactly reproduced easily
    -   Guix can be used to produce docker image
        -   Guix can pack a set of packages and their dependencies (and nothing more, e.g. no leftover files only for building some packages) into various different formats, e.g. docker image, tarball and squashfs image, see the `-f` option in [Invoking guix pack](https://guix.gnu.org/manual/en/guix.html#Invoking-guix-pack) for details
            -   Guix can also build a vm-image, disk-image or docker image through `guix system vm-image`, `guix system disk-image` or `guix system docker-image` respectively, see [Invoking guix system](https://guix.gnu.org/manual/en/html_node/Invoking-guix-system.html#Invoking-guix-system) for details
        -   of course, this can be combined with Guix time machine through a channel file, and to use manifest files for the set of packages wanted
        -   therefore, we can use Guix to produce docker image instead of using dockerfile to enjoy better reproduciblity, and continue to use the surrounding infrastructure built around docker images, e.g. [Kubernetes](https://kubernetes.io/)
        -   therefore, Guix can be used in a complementary way to docker, if you do not wish to completely replace docker with Guix
    -   Guix also allow running programs in container:
        -   by using the isolation capability of the Linux kernel, Guix environment allows running programs in container basically in the same way as docker
        -   the user can also control the sharing of current working directory and the network
        -   see the `--container` or `-C` option and other related options in [Invoking guix environment](https://guix.gnu.org/manual/en/guix.html#Invoking-guix-environment) for details, but note that it requires a Linux kernel at least as new as version 3.19
    -   more fine grained sharing
        -   in Guix, each exact version of each package is identified by a hash (together with package name and version number), and identical packages are cached in the store and easily shared, either directly needed, or indirectly needed as a dependency of another package
        -   this sharing is automatic, so does not need careful arrangement of the user, and much more fine-grained than simple layer sharing in docker image
        -   there has been discussion among Guix developers to extend the sharing to individual files, so that identical files in different packages can also be shared


### Use another functional package manager such as Nix {#use-another-functional-package-manager-such-as-nix}

-   [Nix](https://en.wikipedia.org/wiki/Nix_package_manager) origingated from Dolstra, E.'s PhD research [The Purely Functional Software Deployment Model](https://nixos.org/~eelco/pubs/phd-thesis.pdf), and refers to two related things:
    -   Nix is a dynamically typed, functional programming language with lazy evaluation, designed for implemented the Nix functional package manager
    -   Nix is a functional package manager implemented using the Nix programming language
        -   Nix can be used on Linux and MacOS
        -   Guix is inspired by Nix
        -   basically Guix is a reimplementation of the Nix package manager using Guile instead of the Nix programming lanuage, and in the process can correct some crufts and add some improvements
-   [NixOS](https://nixos.org/) is a Linux distribution based on the Nix package manager, in the same way that Guix system is a Linux distribution based on the Guix package manager
-   Nix is also a fine choice of functional package manager
-   upsides of Nix / downsides of Guix:
    -   while Guix is only available in Linux, Nix is available in both Linux and MacOS. However, the supporting and testing of packages in MacOS may not be as good as in Linux
    -   since Nix was developed (first around 2004) much earlier than Guix (first release in 2013, see [GNU Guix Releases](https://en.wikipedia.org/wiki/GNU_Guix#Releases)), Nix has more packages
-   downsides of Nix / upsides of Guix:
    -   the Nix package manager is implemented in the Nix programming language, which is a very specialized and niche programming language because it is not used in other contexts or purposes
        -   on the other hand, Guix is implemented in Guile, which is a dialect of Scheme. Scheme is a general purpose programming language useful in other contexts
        -   of course, for simple use, the user of Nix need not know much about the underlying implementation language, just as Guix user need not know Guile to simple uses
        -   but for more complicated uses, some knowledge of Nix (respectively Guile in the case of Guix) is needed, in which case learning a general purpose language may be considered more useful than learning a niche language, and the tools normally used for Scheme (e.g. editor support, REPL, debuggers) can be used for Guix hacking
        -   also, the data structures available in the Nix language is more limited, whereas Guix can use distinct data structures for different kinds of objects
            -   e.g. in Nix, each package is represented as a _function_ that takes inputs from a large mapping; but in Guix a package is represented as a first-class object in Scheme, forming an explicit dependency graphs, which easily allows different kinds of processing on the dependencies (e.g. substitute a particular package with a modified one; or plotting the dependency graph for visualization)
    -   Guix was developed much later than Nix, and therefore has a chance to improve on some early decisions of Nix after the years of real world usage experience
        -   e.g. the use of richer Scheme data structures in Guix for different kinds of objects
        -   e.g. the rough equivalent to `guix environment` in Nix is [nix-shell](https://nixos.wiki/wiki/Development_environment_with_nix-shell), which is originally intended to provide a shell where the build-time dependencies of a package is available for development and testing
            -   if you want to have some packages avaiable in the new shell, you can define a dummy package with the desired packages as build-time dependencies
            -   in contrast, `guix environment` allows specifying the list packages of which their dependencies are wanted, or the list of packages which themselves are wanted (the `--ad-hoc` option), or a mixture of the two
                -   apparently `nix-shell` now also allows this: <https://nixos.org/manual/nix/unstable/command-ref/nix-shell.html>
            -   also, to my knowledge `nix-shell` does not have the convenient time machien as `guix time-machine` to pinpoint the set of packages at particular commit of channels, but similar effect could be achieved by some Nix constructs in the `shell.nix` or `default.nix` file manually
                -   see [Pinning Nixpkgs](https://nixos.wiki/wiki/FAQ/Pinning_Nixpkgs) for examples of pinning versions of packages
    -   Nix relies more heavily on the shell for various build steps and occasionally causes trouble due to the escaping and substitutions in the shell, while Guix relies mostly on Scheme, so does not have issues with escaping
-   see the following for more comparison between Nix and Guix:
    -   <https://news.ycombinator.com/item?id=16490027>
    -   <https://sandervanderburg.blogspot.com/2012/11/on-nix-and-gnu-guix.html>
    -   <https://www.reddit.com/r/GUIX/comments/hxcq7d/guix_vs_nix/>
-   in short, both Nix and Guix are fine choices of functional package manager, and share a lot of similarities


## A closer look at Guix {#a-closer-look-at-guix}


### How you might have designed Guix {#how-you-might-have-designed-guix}

TODO


### Overview of the different parts of Guix {#overview-of-the-different-parts-of-guix}

TODO


## Try out Guix {#try-out-guix}


### Get Guix {#get-guix}

Refer to [System Installation](https://guix.gnu.org/manual/en/guix.html#System-Installation) for details on getting guix running
(either the guix system, or just the guix package manager), we brifely
summarize the main ways below. At the time of writing, the guix
version is 1.2.0.


#### Run guix system in qemu virtual machine with pre-made image {#run-guix-system-in-qemu-virtual-machine-with-pre-made-image}

Running guix in a virtual machine is probably the easiest way to
try it out. [Qemu](https://www.qemu.org/) is a generic and open source emulator and virtualizer
that can be used to run a pre-made image.

Refer to [Running Guix in a Virtual Machine](https://guix.gnu.org/manual/en/guix.html#Running-Guix-in-a-VM) for more details. At the
time of writing, the guix version of the image is 1.2.0 for x86\_64
architecture. We summarize the steps below:

-   install qemu following the instructions at [https://www.qemu.org/download/](https://www.qemu.org/download/)
-   download the [guix-system-vm-image-1.2.0.x86\_64-linux.xz](https://ftp.gnu.org/gnu/guix/guix-system-vm-image-1.2.0.x86_64-linux.xz) compressed image
-   decompress to get `guix-system-vm-image-1.2.0.x86_64-linux`

    ```shell
    xz -d guix-system-vm-image-1.2.0.x86_64-linux.xz
    ```
-   invoke qemu on the image with

    ```shell
    qemu-system-x86_64 \
       -nic user,model=virtio-net-pci \
       -enable-kvm -m 1024 \
       -device virtio-blk,drive=myhd \
       -drive if=none,file=./guix-system-vm-image-1.2.0.x86_64-linux,id=myhd
    ```
-   you can change arguments such as `-m` as appropriate to set the
    amount of memory desired. Note that you may need to tweak the
    arguments of invoking qemu a bit, depending on you host system.
    E.g. in macOS Mojave 10.14.6, I need to use the following:

    ```shell
    qemu-system-x86_64 \
       -nic user,model=virtio-net-pci \
       -m 1024 \
       -device virtio-blk,drive=myhd \
       -drive if=none,file=./guix-system-vm-image-1.2.0.x86_64-linux,id=myhd \
       -accel hvf
    ```
-   wait for a while, then you will be in the default xfce desktop of
    a guix system, i.e. a GNU/Linux distribution built around
    guix. But note that by default, not many packages are installed,
    e.g. even a web browser is not installed by default. You may
    install them as needed, e.g. you may install `icecat` (the GNU
    version of Firefox browser) by:

    ```shell
    guix package -i icecat
    ```
-   optional: since the command for invoking qemu can be very long,
    you may put it into a shell script to more conveniently invoke the
    vm
-   optional: you may use other front-ends (some are GUI) to more
    conveniently manage qemu vm, see
    <https://wiki.archlinux.org/index.php/Libvirt#Client>


#### Install the guix system, the GNU/Linux distribution built on the Guix package manager {#install-the-guix-system-the-gnu-linux-distribution-built-on-the-guix-package-manager}

You can also install a guix system in either a virtual or physical machine.

-   Get the guix system install ISO image

    Refer to [USB Stick and DVD Installation](https://guix.gnu.org/manual/en/guix.html#USB-Stick-and-DVD-Installation)

    -   download the compressed 64-bit ISO from <https://ftp.gnu.org/gnu/guix/guix-system-install-1.2.0.x86_64-linux.iso.xz>
        -   you may instead get the compressed 32-bit ISO from <https://ftp.gnu.org/gnu/guix/guix-system-install-1.2.0.i686-linux.iso.xz>
    -   optionally verify the authenticity of the image:
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
    -   decompress the image with `xz -d` command to get `guix-system-install-1.2.0.x86_64-linux.iso`

        ```shell
        xz -d guix-system-install-1.2.0.x86_64-linux.iso.xz
        ```

-   Install in a fresh qemu virtual machine

    Refer to [Installing Guix in a Virtual Machine](https://guix.gnu.org/manual/en/guix.html#Installing-Guix-in-a-VM)

    -   install qemu following the instructions at [https://www.qemu.org/download/](https://www.qemu.org/download/)
    -   create a fresh disk image in qcow2 format, where you can choose an
        appropriate size, e.g. 50G (the created image file will initially
        be much smaller, but will grow in size when it is filled up)

        ```shell
        qemu-img create -f qcow2 guix-system.img 50G
        ```
    -   boot the installation ISO image in the vm:

        ```shell
        qemu-system-x86_64 -m 1024 -smp 1 -enable-kvm \
          -nic user,model=virtio-net-pci -boot menu=on,order=d \
          -drive file=guix-system.img \
          -drive media=cdrom,file=guix-system-install-1.2.0.system.iso
        ```
    -   note that you may need to tweak the arguments to
        `qemu-system-x86_64` depending on your host system, e.g. you may
        need to replace `-enable-kvm` with `-accel hvf` in macOS. The
        `-enable-kvm` is optional anyway, but will significantly improve
        performance.
    -   note the few things mentioned in <https://guix.gnu.org/manual/en/guix.html#Preparing-for-Installation>
        -   The graphical installer is available on TTY1.
        -   You can obtain root shells on TTYs 3 to 6 by hitting ctrl-alt-f3, ctrl-alt-f4, etc.
        -   TTY2 shows this documentation and you can reach it with ctrl-alt-f2.
        -   Documentation is browsable using the Info reader commands (see [Stand-alone GNU Info](https://www.gnu.org/software/texinfo/manual/info-stnd/info-stnd.html#Top)).
        -   The installation system runs the GPM mouse daemon, which allows you to select text with the left mouse button and to paste it with the middle button.
    -   for simplicity, choose [Guided Graphical Installation](https://guix.gnu.org/manual/en/guix.html#Guided-Graphical-Installation) to finish the installation
    -   after the installation, you can get into guix with the default
        xfce desktop whenever you invoke the vm
    -   optional: since the command for invoking qemu can be very long,
        you may put it into a shell script to more conveniently invoke the
        vm
    -   optional: you may use other front-ends (some are GUI) to more
        conveniently manage qemu vm, see
        <https://wiki.archlinux.org/index.php/Libvirt#Client>

-   Install in other virtual machine

    You can also install guix system using the ISO in other types of
    virtual machines such as:

    -   [VirtualBox](https://www.virtualbox.org/) (free)
    -   [Parallels](https://www.parallels.com/) (proprietary)
    -   [VmWare](https://www.vmware.com/) (proprietary)
    -   and many others, see <https://en.wikipedia.org/wiki/Comparison_of_platform_virtualization_software>

    Similar to installing in qemu vm:

    -   note the few things mentioned in <https://guix.gnu.org/manual/en/guix.html#Preparing-for-Installation>
        -   The graphical installer is available on TTY1.
        -   You can obtain root shells on TTYs 3 to 6 by hitting ctrl-alt-f3, ctrl-alt-f4, etc.
        -   TTY2 shows this documentation and you can reach it with ctrl-alt-f2.
        -   Documentation is browsable using the Info reader commands (see [Stand-alone GNU Info](https://www.gnu.org/software/texinfo/manual/info-stnd/info-stnd.html#Top)).
        -   The installation system runs the GPM mouse daemon, which allows you to select text with the left mouse button and to paste it with the middle button.
    -   for simplicity, choose [Guided Graphical Installation](https://guix.gnu.org/manual/en/guix.html#Guided-Graphical-Installation) to finish the installation
    -   after the installation, you can get into guix with the default
        xfce desktop whenever you invoke the vm

-   Install on physical machine

    Installing guix system on physical machine is similar to installing in
    virtual machine, except that you need to transfer the ISO image to
    either a USB stick or a DVD. It is recommended that you try to install
    in a virtual machine first to get familiar with the process, see the
    preceding subsection.

    Refer to <https://guix.gnu.org/manual/en/guix.html#USB-Stick-and-DVD-Installation>

    -   Copying to a USB Stick

        -   Insert a USB stick of 1 GiB or more into your machine
        -   if on a Linux machine,
            -   determine its device name, e.g. `/dev/sdb` or `/dev/sdc`. Note
                that it is VERY important to determine the correct device name,
                otherwise you may mistakenly wipe out another device.
            -   assuming that the USB stick is known as `/dev/sdX`, copy the image
                with:

                ```shell
                # access to /dev/sdX usually requires root privilege
                # also, /dev/sdX should NOT be mount
                sudo dd if=guix-system-install-1.2.0.x86_64-linux.iso of=/dev/sdX status=progress
                sudo sync
                ```
        -   if on a different OS or you prefer to use a GUI, you can try these programs:
            -   <https://www.tipard.com/resource/burn-iso-to-usb.html>
            -   <https://techrrival.com/best-usb-bootable-softwares/>

    -   Burning on a DVD

        -   Insert a blank DVD into your machine
        -   if on a Linux machine,
            -   determine its device name. Note that it is VERY important to
                determine the correct device name, otherwise you may mistakenly
                wipe out another device.
            -   assuming that the DVD drive is known as `/dev/srX`, copy the image
                with:

                ```shell
                # access to /dev/srX usually requires root privilege
                growisofs -dvd-compat -Z /dev/srX=guix-system-install-1.2.0.x86_64-linux.iso
                ```
        -   if on a different OS or you prefer to use a GUI, you can try these programs:
            -   <https://en.wikipedia.org/wiki/List_of_ISO_image_software>
            -   <https://dvdcreator.wondershare.com/dvd-tips/best-free-iso-burner.html>

    -   Installtion

        Similar to installing in qemu vm:

        -   note the few things mentioned in <https://guix.gnu.org/manual/en/guix.html#Preparing-for-Installation>
            -   The graphical installer is available on TTY1.
            -   You can obtain root shells on TTYs 3 to 6 by hitting ctrl-alt-f3, ctrl-alt-f4, etc.
            -   TTY2 shows this documentation and you can reach it with ctrl-alt-f2.
            -   Documentation is browsable using the Info reader commands (see [Stand-alone GNU Info](https://www.gnu.org/software/texinfo/manual/info-stnd/info-stnd.html#Top)).
            -   The installation system runs the GPM mouse daemon, which allows you to select text with the left mouse button and to paste it with the middle button.
        -   for simplicity, choose [Guided Graphical Installation](https://guix.gnu.org/manual/en/guix.html#Guided-Graphical-Installation) to finish the installation
        -   after the installation, you can get into guix with the default
            xfce desktop


#### Install the guix package manager in existing GNU/Linux distribution (possibly in a VM) {#install-the-guix-package-manager-in-existing-gnu-linux-distribution--possibly-in-a-vm}

Besides installing guix system, you may also choose to only install
guix package manager on top of a GNU/Linux distribution (named
"foreign distro in guix documentation", which need not be a guix
system, e.g. Debian, Ubuntu, Fedora, ArchLinux, etc).

Refer to [Binary Installation](https://guix.gnu.org/manual/en/guix.html#Binary-Installation)

-   the easiest way is to use the shell installer script:
    -   download and execute the shell installer script as root

        ```shell
        cd /tmp
        wget https://git.savannah.gnu.org/cgit/guix.git/plain/etc/guix-install.sh
        chmod +x guix-install.sh
        ./guix-install.sh
        ```

        -   you may be asked to import gpg key, if so, just follow the instructions, then re-run the installer script
    -   after that, check [Application Setup](https://guix.gnu.org/manual/en/guix.html#Application-Setup) for extra configuration you might need
-   alternatively, you may choose to install the guix package from the
    package manager of your distribution, if there is one, e.g.:
    -   ArchLinux: <https://wiki.archlinux.org/index.php/Guix#AUR_Package_Installation>
    -   Slackware: <https://slackbuilds.org/repository/14.1/system/guix/>
    -   Fedora: <https://copr.fedorainfracloud.org/coprs/lantw44/guix/>


### Brief summary of commonly used commands in Guix {#brief-summary-of-commonly-used-commands-in-guix}

Here we list some common commands and usage of Guix, see [Getting Started](https://guix.gnu.org/manual/en/guix.html#Getting-Started) of the Guix manual for more details.


#### Basic usage of Guix {#basic-usage-of-guix}

-   Help on Guix command

    Help on the main usage, to see the available commands

    ```shell
    guix --help
    ```

    Help on each command, e.g. the `guix package` command

    ```shell
    guix package --help
    ```

-   Search package

    ```shell
    guix search keywords-here
    # guix package -s keywords-here
    ```

-   Show information of package(s)

    ```shell
    guix show package1 package2
    ```

-   Update channel(s)

    To update the channels so that all the package definitions are updated, but the packages themselves are not automatically updated.

    ```shell
    guix pull
    ```

-   Install package(s)

    You can install one or more packages in one transaction

    ```shell
    guix install package1 package2
    ```

    An alternative way

    ```shell
    guix package -i package1 package2
    # guix package --install package1 package2
    ```

-   Remove package(s)

    ```shell
    guix remove package1 package2
    ```

    An alternative way

    ```shell
    guix package -r package1 package2
    # guix package --remove package1 package2
    ```

-   Upgrade package(s)

    ```shell
    guix upgrade package1 package2
    ```

    An alternative way

    ```shell
    guix package -u package1 package2
    # guix package --upgrade package1 package2
    ```

-   Install/Remove/Upgrade package(s) in one transaction

    You can also install/remove/upgrade package(s) in a single transaction, simply list the package with suitable options of `guix package`, `-i` for installing, `-r` for removing, `-u` for upgrading:

    ```shell
    guix package -i package1 -r package2 -u package3
    ```

-   List installed packages

    ```shell
    guix package -I
    # guix package --list-installed
    ```

-   Rollback to the previous generation

    ```shell
    guix package --roll-back
    ```

-   List generations

    ```shell
    guix package -l
    # guix package --list-generations
    ```

-   Switch generations

    `PATTERN` can be a generation number or `+n` to go forward n generations; or `-n` to go backward n generations

    ```shell
    guix package -S PATTERN
    # guix package --switch-generation=PATTERN
    ```

    E.g. to go forward one generation

    ```shell
    guix package -S +1
    # guix package --switch-generation=+1
    ```

    E.g. to go backward one generation, similar to `--roll-back`, but if the specified generation does not exist, the current generation will not change

    ```shell
    guix package -S -1
    # guix package --switch-generation=-1
    ```


#### Using manifest files to specify a list of packages {#using-manifest-files-to-specify-a-list-of-packages}

A manifest file is a Scheme file that evaluates to a list of guix
_packages_ (as Scheme object). A convenient way is to use the
`specifications->manifest` function which takes a list of package
names as strings and returns a list of the corresponding package
objects. It can be used in a few contexts such as `guix package`
to install packages, or `guix environment` to spawn a shell with
specified packages (see below), through the `-m` or `--manifest`
option. The advantage of a manifest file is that the list of
packages can be easily version controlled (e.g. with `git`).

An example of manifest file (let's say `pkgs.scm`) can be:

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

-   Install the list of packages in a manifest file

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


#### Using Guix profiles {#using-guix-profiles}

Default guix actions (install/upgrade/remove) operates on the
default _profile_ of each user (it is a symbolic link, default
location is at `$HOME/.guix-profile`), but each user and create
multiple profiles, activate the profiles as needed in similar ways
to Python _virtual environments_, and specify the profile to
operate on with each action through the `-p` (or `--profile`)
option (see [Invoking guix package](https://guix.gnu.org/manual/en/guix.html#Invoking-guix-package)):

-   Create profile

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

-   Install or upgrade or remove packages in a profile

    Any usual `guix package` actions can be performed on a chosen
    profile by using the `-p` (or `--profile`) option, e.g. to
    install another package `r-yaml` to the profile `~/my-ds-profile`
    just created:

    ```shell
    guix package -i r-yaml -p ~/my-ds-profile
    ```

    E.g. install packages using a manifest file in a profile:

    ```shell
    guix package -m the_manifest_file -p path_to_profile
    ```

-   List profiles

    ```shell
    guix package --list-profiles
    ```

-   Activate a profile

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

-   Delete a profile

    Somehow guix does not provide convenient command to delete a
    profile. Deleting a profile involves removing the profile
    symbolic link and its siblings that point to specific
    generations. E.g. to delete the `~/my-ds-profile` profile created
    above:

    ```shell
    rm ~/my-profile ~/my-profile-*-link
    ```

-   Export current or chosen profile as a manifest file

    You can export the list of packages of a profile (if not specified, will use the default profile) to a manifest file. E.g. to export the default profile:

    ```shell
    guix package --export-manifest > pkgs1.scm
    ```

    E.g. to export the above `~/my-ds-profile` profile:

    ```shell
    guix package --export-manifest -p ~/my-ds-profile > pkgs2.scm
    ```


#### Using Guix environment {#using-guix-environment}

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
dependencies, as we will see in the demo section.

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

-   Spawn a shell with dependencies of some packages

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

    Also check out `--load` option for loading a file for package.

-   Spawn a shell with some packages themselves

    E.g. to get game `kmahjongg` itself:

    ```shell
    # guix environment --ad-hoc pkg1 pkg2 ...
    guix environment --ad-hoc kmahjongg
    ```

    Can also use `-m` (possibly multiple times) to specify manifest file(s):

    ```shell
    # guix environment --ad-hoc pkg1 pkg2 ... -m pkgs1.scm -m pkgs2.scm ...
    guix environment --ad-hoc kmahjongg -m pkgs.scm
    ```

    Also check out `--load` option for loading a file for package.

-   Spawn a shell and directly execute some commands in it

    e.g. to get `R` and start it

    ```shell
    # guix environment --ad-hoc pkg1 ... -- pkg1
    guix environment --ad-hoc r -- R
    ```

-   Spawn a shell where original environment variables are unset

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

-   Spawn a shell to run things in an isolated environment

     For further isolation, can use the `--container` option to run in
    a container where only the gnu store and (by default) the current
    directory are mounted. You may check `guix environment --help` for
    other relevant options, e.g. `--expose`, `--network`, `--share`,
    `--no-cwd`, `--user`.

    E.g. to run R in a container:

    ```shell
    # guix environment --ad-hoc pkg1 ... -- pkg1
    guix environment --container --ad-hoc r -- R
    ```

    But seems this feature is not very mature yet. When I try the
    above command (on guix (GNU Guix)
    510e24f973a918391d8122fd6ad515c0567bf23e).  it gives this error:

    ```text
    guix environment: error: mount: mount "/home/peter" on "/tmp/guix-directory.QDXQmx//home/peter": Invalid argument
    ```


#### Using Guix time-machine {#using-guix-time-machine}

The `guix time-machine` can be used to execute other guix commands (e.g. `guix package`, or `guix environment`) at specified guix revisions (exact commits of the channels, which can be produced by `guix describe`)

-   Install package in a particular revision

    E.g. to install `emacs` at the revision in the file `channels.scm`

    ```shell
    guix time-machine -C channels.scm package -i emacs
    ```

    The `channels.scm` could be obtained with `guix describe`, e.g.

    ```shell
    guix describe -f channels > channels.scm
    ```

    The `channels.scm` will look something like (exact content of course depends on your channels)

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

-   Use time-machine with guix environment

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

    ```shell
    guix time-machine -C channels.scm -- environment --ad-hoc -m pkgs.scm -- R
    ```


#### Use Guix to create docker image {#use-guix-to-create-docker-image}

TODO


## A little demo of per-project dependency management {#a-little-demo-of-per-project-dependency-management}


### Demo settings {#demo-settings}

Suppose we have two projects `~/guix_demo/proj1` and
`~/guix_demo/proj2`, each with different channel file `channels.scm`
and manifest file `pkgs.scm`, so the two projects uses different
versions of R (proj1 at R 3.6.3, proj2 at R 4.0.3) and different R
packages (at different versions).

Below we spawn two shells that allows us to work on the two projects
_at the same time_, and we show most of the output. If you try the
commands, you should get the same outputs, except possibly the locale
output from R which depends on your local setting.


#### Sample files {#sample-files}

You may either manually create the following files, or clone the git
repository [https://github.com/peterloleungyau/guix\_demo](https://github.com/peterloleungyau/guix_demo) created for
convenience (you should check that the files are the same as below).

If you are in guix but somehow do not have `git` installed yet, you
may either install it and then clone the repository by:

```shell
# install git, and also need ssh
guix package -i git openssh
# clone the repo
cd ~
git clone https://github.com/peterloleungyau/guix_demo.git
```

-   First project

    `~/guix_demo/proj1/channels.scm`:

    ```scheme
    (list (channel
           (name 'guix)
           (url "https://git.savannah.gnu.org/git/guix.git")
           (commit "89909327d017198969436237acc7c93823ff8147")))
    ```

    `~/guix_demo/proj1/pkgs.scm`:

    ```scheme
    (specifications->manifest
     '(
       ;; R
       "r"
       "r-yaml"
       "r-xgboost"
       "r-jsonlite"
       ))
    ```

    `~/guix_demo/proj1/fit1.R`

    ```R
    library(xgboost)

    data(iris)

    is_setosa <- as.numeric(iris$Species == "setosa")
    iris_x <- as.matrix(iris[c("Sepal.Length", "Sepal.Width",
                               "Petal.Length", "Petal.Width")])
    iris_data <- xgb.DMatrix(data = iris_x, label = is_setosa)

    param <- list(max_depth = 3, eta = 1, nthread = 1,
                  objective = "binary:logistic", eval_metric = "auc")

    fit <- xgboost(params = param, data = iris_data, nrounds = 10)

    print(fit)

    ```

-   Second project

    `~/guix_demo/proj2/channels.scm`

    ```scheme
    (list (channel
           (name 'guix)
           (url "https://git.savannah.gnu.org/git/guix.git")
           (commit "9904a15a4c838362673c1affdbaf1e83d92fe8ff")))
    ```

    `~/guix_demo/proj2/pkgs.scm`:

    ```scheme
    (specifications->manifest
     '(
       ;; R
       "r"
       "r-glmnet"
       "r-jsonlite"
       ))
    ```

    `~/guix_demo/proj2/fit2.R`

    ```R
    library(glmnet)

    data(iris)

    is_setosa <- as.numeric(iris$Species == "setosa")
    iris_x <- as.matrix(iris[c("Sepal.Length", "Sepal.Width",
                               "Petal.Length", "Petal.Width")])

    fit <- glmnet(x = iris_x, y = is_setosa, family = "binomial")

    summary(fit)
    print(fit)

    ```


### Open a shell for the first project {#open-a-shell-for-the-first-project}

First open a new shell with guix (could be the guix package manager on
a Linux distribution, or the guix system) to create an environment for
proj1:

```shell
cd ~/guix_demo/proj1
guix time-machine -C channels.scm -- environment --ad-hoc -m pkgs.scm -- R
```

Then wait a while for the R prompt to appear. The first time executing
the above may take quite some time to download and/or build the
packages. Subsequent run should be much faster after the needed
packages are cached.

In the R REPL, we load the few packages and show their versions:

```text
R version 3.6.3 (2020-02-29) -- "Holding the Windsock"
Copyright (C) 2020 The R Foundation for Statistical Computing
Platform: x86_64-unknown-linux-gnu (64-bit)

R is free software and comes with ABSOLUTELY NO WARRANTY.
You are welcome to redistribute it under certain conditions.
Type 'license()' or 'licence()' for distribution details.

R is a collaborative project with many contributors.
Type 'contributors()' for more information and
'citation()' on how to cite R or R packages in publications.

Type 'demo()' for some demos, 'help()' for on-line help, or
'help.start()' for an HTML browser interface to help.
Type 'q()' to quit R.

> library(xgboost)
> library(yaml)
> library(jsonlite)
> sessionInfo()
R version 3.6.3 (2020-02-29)
Platform: x86_64-unknown-linux-gnu (64-bit)

Matrix products: default
BLAS/LAPACK: /gnu/store/vax1vsg3ivf0r7j7n2xkbi1z3r0504l9-openblas-0.3.7/lib/libopenblasp-r0.3.7.so

locale:
[1] C

attached base packages:
[1] stats     graphics  grDevices utils     datasets  methods   base

other attached packages:
[1] jsonlite_1.6.1  yaml_2.2.1      xgboost_1.0.0.2

loaded via a namespace (and not attached):
[1] compiler_3.6.3    magrittr_1.5      Matrix_1.2-18     tools_3.6.3
[5] stringi_1.4.6     grid_3.6.3        data.table_1.12.8 lattice_0.20-41
>
```


### Open another shell for the second project {#open-another-shell-for-the-second-project}

Then _while keeping the first shell open_ with the R prompt, we open a
new shell, then do the same for proj2:

```shell
cd ~/guix_demo/proj2
guix time-machine -C channels.scm -- environment --ad-hoc -m pkgs.scm -- R
```

In the R REPL, we also load the packages and show the versions:

```text
R version 4.0.3 (2020-10-10) -- "Bunny-Wunnies Freak Out"
Copyright (C) 2020 The R Foundation for Statistical Computing
Platform: x86_64-unknown-linux-gnu (64-bit)

R is free software and comes with ABSOLUTELY NO WARRANTY.
You are welcome to redistribute it under certain conditions.
Type 'license()' or 'licence()' for distribution details.

R is a collaborative project with many contributors.
Type 'contributors()' for more information and
'citation()' on how to cite R or R packages in publications.

Type 'demo()' for some demos, 'help()' for on-line help, or
'help.start()' for an HTML browser interface to help.
Type 'q()' to quit R.

> library(glmnet)
Loading required package: Matrix
Loaded glmnet 4.1
> library(jsonlite)
> sessionInfo()
R version 4.0.3 (2020-10-10)
Platform: x86_64-unknown-linux-gnu (64-bit)

Matrix products: default
BLAS/LAPACK: /gnu/store/bs9pl1f805ins80xaf4s3n35a0x2lyq3-openblas-0.3.9/lib/libopenblasp-r0.3.9.so

locale:
[1] C

attached base packages:
[1] stats     graphics  grDevices utils     datasets  methods   base

other attached packages:
[1] jsonlite_1.7.2 glmnet_4.1     Matrix_1.3-2

loaded via a namespace (and not attached):
[1] compiler_4.0.3   survival_3.2-7   splines_4.0.3    codetools_0.2-18
[5] grid_4.0.3       iterators_1.0.13 foreach_1.5.1    shape_1.4.5
[9] lattice_0.20-41
>
```


### Close both shells {#close-both-shells}

Since we directly run `R` when creating the guix environment, so
now quitting R with `q()` will also exit the spawn shell. So now we
can exit both shells spawn above.

If you now re-run the above commands, it should be much faster,
because the needed packages are cached, and guix need only check
that the packages needed are there.


### Run R script for the first project {#run-r-script-for-the-first-project}

We also run an R script:

```shell
cd ~/guix_demo/proj1
guix time-machine -C channels.scm -- environment --ad-hoc -m pkgs.scm -- Rscript fit1.R
```

The output:

```text
[1]     train-auc:1.000000
[2]     train-auc:1.000000
[3]     train-auc:1.000000
[4]     train-auc:1.000000
[5]     train-auc:1.000000
[6]     train-auc:1.000000
[7]     train-auc:1.000000
[8]     train-auc:1.000000
[9]     train-auc:1.000000
[10]    train-auc:1.000000
##### xgb.Booster
raw: 2.6 Kb
call:
  xgb.train(params = params, data = dtrain, nrounds = nrounds,
    watchlist = watchlist, verbose = verbose, print_every_n = print_every_n,
    early_stopping_rounds = early_stopping_rounds, maximize = maximize,
    save_period = save_period, save_name = save_name, xgb_model = xgb_model,
    callbacks = callbacks)
params (as set within xgb.train):
  max_depth = "3", eta = "1", nthread = "1", objective = "binary:logistic", eval_metric = "auc", silent = "1"
xgb.attributes:
  niter
callbacks:
  cb.print.evaluation(period = print_every_n)
  cb.evaluation.log()
# of features: 4
niter: 10
nfeatures : 4
evaluation_log:
    iter train_auc
       1         1
       2         1
---
       9         1
      10         1
```


### Run R script for the second project {#run-r-script-for-the-second-project}

We also run an R script:

```shell
cd ~/guix_demo/proj2
guix time-machine -C channels.scm -- environment --ad-hoc -m pkgs.scm -- Rscript fit2.R
```

The output:

```text
Loading required package: Matrix
Loaded glmnet 4.1
           Length Class     Mode
a0          78    -none-    numeric
beta       312    dgCMatrix S4
df          78    -none-    numeric
dim          2    -none-    numeric
lambda      78    -none-    numeric
dev.ratio   78    -none-    numeric
nulldev      1    -none-    numeric
npasses      1    -none-    numeric
jerr         1    -none-    numeric
offset       1    -none-    logical
classnames   2    -none-    character
call         4    -none-    call
nobs         1    -none-    numeric

Call:  glmnet(x = iris_x, y = is_setosa, family = "binomial")

   Df  %Dev  Lambda
1   0  0.00 0.43500
2   1 11.32 0.39640
3   1 20.79 0.36110
4   1 28.87 0.32910
5   1 35.87 0.29980
6   1 41.99 0.27320
7   1 47.38 0.24890
8   1 52.17 0.22680
9   1 56.44 0.20670
10  1 60.27 0.18830
11  1 63.72 0.17160
12  1 66.83 0.15630
13  1 69.64 0.14240
14  1 72.19 0.12980
15  1 74.51 0.11830
16  2 76.69 0.10780
17  2 78.74 0.09818
18  2 80.61 0.08946
19  2 82.30 0.08151
20  2 83.83 0.07427
21  2 85.23 0.06767
22  2 86.49 0.06166
23  2 87.65 0.05618
24  2 88.70 0.05119
25  2 89.67 0.04664
26  2 90.54 0.04250
27  2 91.34 0.03872
28  2 92.08 0.03528
29  2 92.74 0.03215
30  2 93.35 0.02929
31  2 93.91 0.02669
32  2 94.42 0.02432
33  2 94.89 0.02216
34  2 95.32 0.02019
35  2 95.71 0.01840
36  2 96.07 0.01676
37  2 96.40 0.01527
38  2 96.70 0.01392
39  2 96.97 0.01268
40  2 97.22 0.01155
41  2 97.45 0.01053
42  2 97.67 0.00959
43  2 97.86 0.00874
44  2 98.04 0.00796
45  2 98.20 0.00726
46  3 98.35 0.00661
47  3 98.49 0.00602
48  3 98.62 0.00549
49  3 98.73 0.00500
50  3 98.84 0.00456
51  3 98.94 0.00415
52  3 99.03 0.00378
53  3 99.11 0.00345
54  3 99.19 0.00314
55  3 99.25 0.00286
56  3 99.32 0.00261
57  3 99.37 0.00238
58  3 99.43 0.00216
59  3 99.48 0.00197
60  3 99.52 0.00180
61  3 99.56 0.00164
62  3 99.60 0.00149
63  3 99.63 0.00136
64  3 99.66 0.00124
65  3 99.69 0.00113
66  3 99.72 0.00103
67  3 99.74 0.00094
68  3 99.76 0.00085
69  3 99.78 0.00078
70  3 99.80 0.00071
71  3 99.82 0.00065
72  3 99.83 0.00059
73  3 99.85 0.00054
74  3 99.86 0.00049
75  3 99.87 0.00045
76  3 99.88 0.00041
77  3 99.89 0.00037
78  3 99.90 0.00034
```

TODO


## Other related topics {#other-related-topics}

TODO

-   use Guix in other OS than GNU/Linux


### Guix system {#guix-system}


### Custom guix channels {#custom-guix-channels}


### Guix deploy {#guix-deploy}
