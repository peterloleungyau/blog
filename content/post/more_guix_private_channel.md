+++
title = "More Guix: Private Channel and Internal Package"
date = 2021-07-15
tags = ["Guix", "Functional Package Manager", "Reproducibility"]
categories = ["Guix"]
draft = false
author = "Peter Lo"
+++

Having introduced the basic ideas of Guix in previous posts, this time
we explore setting up a private Guix channel. Guix package definitions
are managed with git repositories called _channels_. There is an
official channel at <https://git.sjtu.edu.cn/sjtug/guix.git>, there are
also third party channels such as <https://gitlab.com/nonguix/nonguix>
(for some software which cannot be included in the official
distribution for ethical or policy-related reasons) and
<https://github.com/guix-science/guix-science.git> (for scientific
software, which cannot be included upstream). Unsurprisingly, you can
setup your own Guix channels for your internal packages. This time we
explore setting up private channels for your custom packages. With the
private channel and custom packages, we could enjoy the benefits of
Guix in per-project dependency management (and other benefits) by
maintaining a channels file and a manifest file for each project.


## Motivation for private Guix channel {#motivation-for-private-guix-channel}

Although it is recommended that you contribute package definitions to
Guix proper (see [Creating a Channel](https://guix.gnu.org/manual/en/html%5Fnode/Creating-a-Channel.html) and
see [contributing](https://guix.gnu.org/manual/en/html%5Fnode/Contributing.html) for details on how to contribute package
definitions), there are still reasons that you may want to maintain
private channels:

-   a needed package is not in the official channel (yet):

    If a needed package is not currently available in the official
    channel (or other third party channels), then you can either try to
    import it with `guix import` (see [Invoking guix import](https://guix.gnu.org/manual/en/html%5Fnode/Invoking-guix-import.html#Invoking-guix-import)), or write
    the package definition yourself. After that, you are encouraged to
    submit a patch to add the package to the official Guix channel (see
    [contributing](https://guix.gnu.org/manual/en/html%5Fnode/Contributing.html) for details). But it takes time for the maintainers to
    review and merge the patch. Meanwhile, you may choose to add the new
    package definition to your private channel, so that you can continue
    with your work.

-   a package is internal and should not be made public:

    If you do not wish to make a needed package public, e.g. internal
    package in your company, it is natural to put the package definition
    in a private channel, so that you can still enjoy the benefits of
    Guix for dependency management.


## Private Guix channel as git repository {#private-guix-channel-as-git-repository}

For convenience, we setup a git repository on [Github](https://github.com/), but other git
repository providers such as [GitLab](https://about.gitlab.com/), [Bitbucket](https://bitbucket.org/) should also work just
fine.


### Create a simple private Guix channel {#create-a-simple-private-guix-channel}

Reference: [Creating a Channel](https://guix.gnu.org/manual/en/html%5Fnode/Creating-a-Channel.html)

1.  Create a Github account at [Sign up](https://github.com/signup?ref%5Fcta=Sign+up&ref%5Floc=header+logged+out&ref%5Fpage=%2F&source=header-home) if you do not already have one
2.  Setup SSH key for Github for convenience
    -   Refer to [Generating a new SSH key and adding it to the ssh-agent](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)
        and [Adding a new SSH key to your GitHub account](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account) to conveniently
        access your Github account and private repositories without
        entering password.
    -   NOTE: I previously generated SSH key following the above
        guidelines from Github, and can push to and pull from my private
        repository just fine. But at the later testing step, Guix is
        unable to access the private channel with this error:

        ```shell
        $ guix time-machine -C channels.scm --disable-authentication --
        environment --ad-hoc python-radian
        Updating channel 'guix' from Git repository at
        'https://git.savannah.gnu.org/git/guix.git'...
        guix time-machine: warning: channel authentication disabled
        Updating channel 'my-guix-pkgs' from Git repository at
        'git@github.com:peterloleungyau/my-guix-pkgs.git'...
        guix time-machine: error: Git error: Failed to retrieve list of SSH
        authentication methods: Failed getting response
        ```

        After some googling, it seems to be a bug in `libssh2` which Guix
        uses for retrieving private channels:

        -   <https://github.com/libgit2/pygit2/issues/1013>
        -   <https://github.com/saltstack/salt/issues/57121>

        The suggested workaround is to generate the key in PEM format by
        using the option `-m PEM` in `ssh-keygen`, so I have tried this
        command to generate the SSH key and followed the rest of the
        instructions to add the public key to Github (and moved my old SSH
        keys from my `~/.ssh/` to a temporary folder) and then Guix can
        access the private repository (from both Guix in foreign
        distribution and Guix system):

        ```shell
        ssh-keygen -m PEM -t rsa -b 4096 -C "peterloleungyau@gmail.com"
        ```

        So if at the time that you are reading this and get the same
        error as above (which means the bug in `libssh2` is probably
        still unfixed), then you may try the above workaround.

    -   Make sure your ssh-agent is setup properly so that the key(s)
        added persist across reboots. E.g. refer to
        <https://stackoverflow.com/questions/3466626/how-to-permanently-add-a-private-key-with-ssh-add-on-ubuntu>
3.  Create a (private) git repository
    1.  Signin your GitHub account
    2.  Click "New repository" in the "+" drop down menu at top right corner
        -   Fill in the "Repository name", here we use `my-guix-pkgs`
        -   Optionally fill in description, e.g. "my private channel of Guix packages"
        -   choose whether this repository is "Public" or "Private". For illustration of internal packages, we choose "Private". You may also choose to make the channel public, i.e. as a third party channel.
        -   Optionally choose whether to initialize with a `README.md` file, a `.gitignore`, and a license.
        -   Click "Create repository" to finish

            \#+CAPTION Create new repository on Github
            ![](/ox-hugo/more_guix_private_channel_create_repo.png)

    3.  Follow the instructions to clone it to your local machine, repeated here for convenience:
        -   In your terminal, assuming you already have [git](https://git-scm.com/downloads) installed, clone
            with the `git clone` command, note that the exact URL will depend
            on your Github user name and your chosen repository name:

            ```shell
            # note that your url may be different, depending on your username and chosen repo name
            # the general url will be git@github.com:<user-name>/<repository-name>.git
            # also, we choose "SSH" because we already have setup the SSH key
            git clone git@github.com:peterloleungyau/my-guix-pkgs.git
            ```
        -   Note that if you have not added anything (e.g. README.md,
            `.gitignore`, or license) in the previous step, your
            repository will now be empty, but we will add content to it
            soon.
4.  Add personal package definitions

    The repository can contain package definitions organized as [Guile
    modules](https://www.gnu.org/software/guile/manual/guile.html#Modules), as different sub-directories. For example, if you have a
    file `my-packages/ds-tools.scm`, it corresponds to a Guile module
    `(my-packages ds-tools)`. You may organize the packages in a
    sensible way you like.

    For this illustration, we first create one file in the
    repository. At the time of writing, [radian](https://pypi.org/project/radian/), which is "A 21 century R
    console", is still not in the official Guix repository. And in a
    previous post [Guix Introduction Part 6: R Development with Guix]({{< relref "guix_intro_6_dev" >}}) we
    used `guix import` to import the relevant package and dependencies
    for `radian` (I know, I should have submitted this to the Guix
    channel as patch, but I am kind of lazy, and life gets in the
    way). So for illustration, we will use those package definitions as
    example.

    1.  Under your git repository cloned above, put the following file
        as `my-packages/ds-tools.scm` (note that we remove the last line
        `python-radian` which is only needed when the file is used with
        the `-l` option, but not needed in a channel):

        ```scheme
        (define-module (my-packages ds-tools)
          #:use-module (guix)
          #:use-module (guix licenses)
          #:use-module (guix download)
          #:use-module (guix git-download)
          #:use-module (gnu packages statistics)
          #:use-module (gnu packages python)
          #:use-module (gnu packages python-science)
          #:use-module (gnu packages python-xyz)
          #:use-module (gnu packages libffi)
          #:use-module (gnu packages check)
          #:use-module (gnu packages terminals)
          #:use-module (guix build-system python))

        (define-public python-lineedit
          (package
            (name "python-lineedit")
            (version "0.1.6")
            (source
              (origin
                (method url-fetch)
                (uri (pypi-uri "lineedit" version))
                (sha256
                  (base32
                    "0gvggy22s3qlz3r5lrwr5f4hzwbq7anyd2vfrzchldaf2mwm8ygl"))))
            (build-system python-build-system)
            (arguments `(#:tests? #f))
            (propagated-inputs
              `(("python-pygments" ,python-pygments)
                ("python-six" ,python-six)
                ("python-wcwidth" ,python-wcwidth)))
            (native-inputs
              `(("python-pexpect" ,python-pexpect)
                ("python-ptyprocess" ,python-ptyprocess)
                ("python-pyte" ,python-pyte)
                ("python-pytest" ,python-pytest)
                ("python-pytest-cov" ,python-pytest-cov)))
            (home-page "https://github.com/randy3k/lineedit")
            (synopsis
              "An readline library based on prompt_toolkit which supports multiple modes")
            (description
              "An readline library based on prompt_toolkit which supports multiple modes")
            (license #f)))

        (define-public python-rchitect
          (package
            (name "python-rchitect")
            (version "0.3.30")
            (source
              (origin
                (method url-fetch)
                (uri (pypi-uri "rchitect" version))
                (sha256
                  (base32
                    "1bg5vrgp447czgmjjky84yqqk2mfzwwgnf0m99lqzs7jq15q8ziv"))))
            (build-system python-build-system)
            (arguments `(#:tests? #f))
            (propagated-inputs
              `(("python-cffi" ,python-cffi)
                ("python-six" ,python-six)))
            (native-inputs
              `(("python-pytest" ,python-pytest)
                ("python-pytest-runner" ,python-pytest-runner)
                ("python-pytest-cov" ,python-pytest-cov)
                ("python-pytest-mock" ,python-pytest-mock)))
            (home-page "https://github.com/randy3k/rchitect")
            (synopsis "Mapping R API to Python")
            (description "Mapping R API to Python")
            (license #f)))

        (define-public python-pyte
          (package
            (name "python-pyte")
            (version "0.8.0")
            (source
             (origin
               (method url-fetch)
               (uri (pypi-uri "pyte" version))
               (sha256
                (base32
                 "1ic8b9xrg76z55qrvbgpwrgg0mcq0dqgy147pqn2cvrdjwzd0wby"))))
            (build-system python-build-system)
            (arguments
             '(#:phases
               (modify-phases %standard-phases
                 (add-after 'unpack 'remove-failing-test
                   ;; TODO: Reenable when the `captured` files required by this test
                   ;; are included in the archive.
                   (lambda _
                     (delete-file "tests/test_input_output.py")
                     #t)))))
            (propagated-inputs
             `(("python-wcwidth" ,python-wcwidth)))
            (native-inputs
             `(("python-pytest-runner" ,python-pytest-runner)
               ("python-pytest" ,python-pytest)))
            (home-page "https://pyte.readthedocs.io/")
            (synopsis "Simple VTXXX-compatible terminal emulator")
            (description "@code{pyte} is an in-memory VTxxx-compatible terminal
        emulator.  @var{VTxxx} stands for a series of video terminals, developed by
        DEC between 1970 and 1995.  The first and probably most famous one was the
        VT100 terminal, which is now a de-facto standard for all virtual terminal
        emulators.

        pyte is a fork of vt102, which was an incomplete pure Python implementation
        of VT100 terminal.")
            (license lgpl3+)))

        (define-public python-radian
          (package
            (name "python-radian")
            (version "0.5.10")
            (source
              (origin
                (method url-fetch)
                (uri (pypi-uri "radian" version))
                (sha256
                  (base32
                    "0plkv3qdgfxyrmg2k6c866q5p7iirm46ivhq2ixs63zc05xdbg8s"))))
            (build-system python-build-system)
            (arguments `(#:tests? #f))
            (propagated-inputs
              `(("python-lineedit" ,python-lineedit)
                ("python-pygments" ,python-pygments)
                ("python-rchitect" ,python-rchitect)
                ("python-six" ,python-six)))
            (native-inputs
              `(("python-coverage" ,python-coverage)
                ("python-pexpect" ,python-pexpect)
                ("python-ptyprocess" ,python-ptyprocess)
                ("python-pytest-runner" ,python-pytest-runner)
                ("python-pyte" ,python-pyte)
                ("python-pytest" ,python-pytest)))
            (home-page "https://github.com/randy3k/radian")
            (synopsis "A 21 century R console")
            (description "A 21 century R console")
            (license #f)))

        ```

    2.  Commit and push the file: in the terminal, in the directory of
        your cloned repository, type:

        ```shell
        # at the repository directory
        # stage the file
        git add my-packages/ds-tools.scm

        # check that the file is properly added
        git status

        # commit, with a commit message
        git commit -m "Added ds-tools.scm"

        # push to GitHub
        git push
        ```

        Now if you go to your GitHub repository, you should also see the
        committed file.

5.  Test the private channel
    1.  Create a channels file `channels.scm` somewhere, e.g. at `~/`:

        ```scheme
        (list (channel
               (name 'guix)
               (url "https://git.savannah.gnu.org/git/guix.git")
               (commit "9904a15a4c838362673c1affdbaf1e83d92fe8ff"))
              (channel
               (name 'my-guix-pkgs)
               (url "git@github.com:peterloleungyau/my-guix-pkgs.git")
               (commit "8cacb5380cb0339bd36238173d80354539ca4a59")
               (branch "master")))

        ```

        Note that it is recommended that you explicitly specify the
        branch of the private channel, and you should check whether the
        default branch is `master` or `main`, e.g. by checking from your
        Github.

        Also, you should replace the commit of `my-guix-pkgs`
        (`8cacb5380cb0339bd36238173d80354539ca4a59`) above with the
        commit of your private channel repository, which you can check
        with `git log` in that repository. Yours may not be the same as
        mine here, because I made more than one commit in creating the
        repository above.

        ```shell
        $ git log
        commit 8cacb5380cb0339bd36238173d80354539ca4a59 (HEAD -> master, origin/master, origin/HEAD)
        Author: Peter Lo <peterloleungyau@gmail.com>
        Date:   Thu Jul 8 00:28:35 2021 +0800

            Define module for ds-tools.

        commit b51d236ebbbdd134bafb64e5092342a2d058ec2a
        Author: Peter Lo <peterloleungyau@gmail.com>
        Date:   Wed Jul 7 00:12:31 2021 +0800

            Added ds-tools.scm
        (END)

        ```

    2.  Try to create a Guix environment by:

        ```shell
        # replace ~/channels.scm with the proper path to your created channels.scm
        guix time-machine -C ~/channels.scm -- environment --ad-hoc python-radian r-minimal -- radian
        ```

        Then wait for a while, if all goes well, then you should be in a
        `radian` REPL.


### Demo: add a sample R package built from Github {#demo-add-a-sample-r-package-built-from-github}

We also try to add a custom R package to our private channel, and also
put the R package in a _private_ repository, to illustrate creating
internal package.

1.  Repository for R package
    -   Reference for creating R package: the book [R Packages](https://r-pkgs.org/index.html) by Hadley
        Wickham and Jenny Bryan. [Chapter 2](https://r-pkgs.org/whole-game.html) of the book gives an example
        of creating a toy package.
    -   [Notes on Hadley's "R Packages"](https://gist.github.com/peterhurford/f71bf00d8866094eac6c) gives a quick summary of the book.
    -   The book has an example R package at
        <https://github.com/jennybc/foofactors> which is a good example
        because it depends on the [forcats](https://rdrr.io/cran/forcats/man/forcats-package.html) package, which is available as
        `r-forcats`, which is in the official Guix channel.
    -   We could have used this repository directly if we want to test an
        R package at public repository.
    -   Since we want to test an R at private repository, we will clone
        the repository and create a private one. I tried forking the
        repository, but after that Github does not allow changing the
        forked repository from public to private "for security reasons".
        1.  First clone <https://github.com/jennybc/foofactors> to your
            local machine, say at the home directory:

            ```shell
            # e.g. we clone to the home directory
            cd ~
            git clone https://github.com/jennybc/foofactors.git
            ```

        2.  Create a **private** empty repository at Github using the same
            steps as above in creating the private channel repository. I
            will keep the same name for this repository,
            i.e. `foofactors`. So the URL of my repository is
            `git@github.com:peterloleungyau/foofactors.git`
        3.  Push the local cloned repository to the empty repository at
            Github:

            ```shell
            cd ~/foofactors
            # change the remote origin to the new URL
            git remote set-url origin git@github.com:peterloleungyau/foofactors.git
            # now can push
            git push
            ```
        4.  Use `git log` to check the latest commit of the repository, to
            be used below. At the time of writing, the latest commit is
            `ef71e8d2e82fa80e0cfc249fd42f50c01924326d`
        5.  (Optional) You may check at Github that the repository is
            no longer empty.
2.  Package definition for the R package
    -   Besides using `guix import` to import existing package (e.g. from
        CRAN), the easiest way to write a package definition is to modify
        from a similar package.
    -   In our case, we want to write a package definition for an R
        package at Github.
    -   It would be convenient to clone the official Guix repository
        (<https://git.savannah.gnu.org/git/guix.git>) to your local
        machine:

        ```shell
        git clone https://git.savannah.gnu.org/git/guix.git
        # or you can do a shallow clone to get only the latest commit, to save time
        # git clone --depth 1 https://git.savannah.gnu.org/git/guix.git
        ```

        Alternatively you may try to find cloned Guix repository at
        Github, e.g. <https://github.com/zimoun/guix>
    -   Most R CRAN packages are in `gnu/packages/cran.scm`
    -   So we may search `git` in `cran.scm` to see if we can find some
        useful package definitions as reference:

        ```shell
        # find anything related to git
        grep -w -C 5 git ~/guix/gnu/packages/cran.scm
        # or you can open cran.scm with your favorite text editor and search
        ```
    -   At the time of writing, the Guix repo is at commit
        `7760d28920a920791645c4485f1345af45ee7787`, and from the above
        search, it seems `r-sankeyd3` is useful as a reference, because
        it uses an explicit commit from a git repository as the package
        source. Its package definition is reproduced here for easy
        reference:

        ```scheme
        (define-public r-sankeyd3
          (let ((commit "fd50a74e29056e0d67d75b4d04de47afb2f932bc")
                (revision "1"))
            (package
             (name "r-sankeyd3")
             (version (git-version "0.3.2" revision commit))
             (source
              (origin
               (method git-fetch)
               (uri (git-reference
                     (url "https://github.com/fbreitwieser/sankeyD3")
                     (commit commit)))
               (file-name (git-file-name name version))
               (sha256
                (base32
                 "0jrcnfax321pszbpjdifnkbrgbjr43bjzvlzv1p5a8wskksqwiyx"))))
             (build-system r-build-system)
             (propagated-inputs
              `(("r-d3r" ,r-d3r)
                ("r-htmlwidgets" ,r-htmlwidgets)
                ("r-shiny" ,r-shiny)
                ("r-magrittr" ,r-magrittr)))
             (home-page "https://github.com/fbreitwieser/sankeyD3")
             (synopsis "Sankey network graphs from R")
             (description
              "This package provides an R library to generate Sankey network graphs
        in R and Shiny via the D3 visualization library.")
             ;; The R code is licensed under GPLv3+.  It includes the non-minified
             ;; JavaScript source code of d3-sankey, which is released under the
             ;; 3-clause BSD license.
             (license (list license:gpl3+ license:bsd-3)))))
        ```
    -   We note a few things of the package definition:
        -   name: `r-sankeyd3`, the Guix convention for R package is to
            have the `r-` prefix, and prefer lower case.
        -   commit: `fd50a74e29056e0d67d75b4d04de47afb2f932bc` is the git commit
        -   version: 0.3.2
        -   revision: "1", seems here just for the version name
        -   url: the URL of the git repository
        -   sha256: seems some kind of hash in base32 format
        -   build-system: `r-build-system` as this is an R package
        -   propagated-inputs: the dependencies, which will be installed
            _visibly_ (i.e. as if the user also manually installed the
            propagated input) together with this package
        -   home-page: the homepage of the package
        -   synopsis: a short description of the package
        -   description: a longer description of the package
        -   license: seems can specify one or more licenses as a list
    -   We therefore need to figure out these information for `foofactors`
    -   The `foofactors/DESCRIPTION` file provides a lot of useful
        information, reproduced below for convenience:

        ```text
        Package: foofactors
        Title: Make Factors Less Aggravating
        Version: 0.0.0.9000
        Authors@R:
            person("Jane", "Doe", email = "jane@example.com", role = c("aut", "cre"))
        Description: Factors have driven people to extreme measures, like ordering
            custom conference ribbons and laptop stickers to express how HELLNO we
            feel about stringsAsFactors. And yet, sometimes you need them. Can they
            be made less maddening? Let's find out.
        License: MIT + file LICENSE
        Encoding: UTF-8
        LazyData: true
        RoxygenNote: 7.1.1
        Suggests:
            testthat
        Imports:
            forcats

        ```

        -   name: following the convention we will use `r-foofactors`
        -   version: 0.0.0.9000
        -   revision: can just use 1
        -   commit: determined above using `git log` to be `ef71e8d2e82fa80e0cfc249fd42f50c01924326d`
        -   url: the URL of the private repository
            `git@github.com:peterloleungyau/foofactors.git`
        -   sha256: this is a hash of the source content
            -   if the source is fetched with `url-fetch`, then we can use
                `guix download` command at the terminal to download and
                calculate the hash
            -   but since now the source is fetched with `git-fetch`,
                and currently `guix download` does not support this
            -   instead, according to the [Extended example](https://guix.gnu.org/cookbook/en/html%5Fnode/Extended-example.html) of Guix packing,
                we can use `guix hash` on the **freshly** cloned repository as
                follows:

                ```shell
                # clone the repository, make sure the content is not changed
                git clone git@github.com:peterloleungyau/foofactors.git
                # go to the repo
                cd foofactors
                # checkout the latest commit
                # alternatively, you can checkout a specific commit
                git checkout HEAD
                # finally calculate hash
                guix hash -rx .
                ```
            -   in our case, the calculated hash is `1hmfwac2zdl8x6r21yy5b257c4891106ana4j81hfn6rd0rl9f72`
        -   build-system: `r-build-system` as this is an R package
        -   propagated-inputs: this package imports `forcats`, so include
            `r-forcats` which is in Guix's official repository already
        -   home-page: can simply use the original Github repository
            <https://github.com/jennybc/foofactors>
        -   synopsis: a short description, maybe modify from the title "A R package to make factors less aggravating."
        -   description: just copy the description above.
        -   license: MIT license, which is called [expat](https://www.gnu.org/licenses/license-list.html#Expat) in Guix, which is
            `license:expat` in `guix/licenses.scm`.
    -   With the above determined information, and referring to
        `gnu/packages/cran.scm` of the Guix repository to add some needed
        modules, we add the following file to our local channel
        repository at `~/my-guix-pkgs/my-packages/r-pkgs.scm`:

        ```scheme
        (define-module (my-packages r-pkgs)
          #:use-module ((guix licenses) #:prefix license:)
          #:use-module (guix packages)
          #:use-module (guix download)
          #:use-module (guix git-download)
          #:use-module (guix utils)
          #:use-module (guix build-system r)
          #:use-module (gnu packages)
          #:use-module (gnu packages statistics))

        (define-public r-foofactors
          (let ((commit "ef71e8d2e82fa80e0cfc249fd42f50c01924326d")
                (revision "1"))
            (package
              (name "r-foofactors")
              (version (git-version "0.0.0.9000" revision commit))
              (source
               (origin
                 (method git-fetch)
                 (uri (git-reference
                       (url "git@github.com:peterloleungyau/foofactors.git")
                       (commit commit)))
                 (file-name (git-file-name name version))
                 (sha256
                  (base32
                   "1hmfwac2zdl8x6r21yy5b257c4891106ana4j81hfn6rd0rl9f72"))))
              (build-system r-build-system)
              (propagated-inputs
               `(("r-forcats" ,r-forcats)))
              (home-page "https://github.com/jennybc/foofactors")
              (synopsis "A R package to make factors less aggravating.")
              (description
               "Factors have driven people to extreme measures, like ordering
        custom conference ribbons and laptop stickers to express how HELLNO we
        feel about stringsAsFactors. And yet, sometimes you need them. Can they
        be made less maddening? Let's find out.")
              (license license:expat))))
        ```

        Note that we define a module corresponding to the path of the file,
        and we use some modules that seem to be needed, and also the
        `(gnu packages statistics)` module because the package depends on
        `r-forcats` which resides in `(gnu packages statistics)` by
        checking with `guix search r-forcats` which shows the location of
        the file to be `gnu/packages/statistics.scm:5588:2`.

        But the above package definition has not been tested yet, and as
        we will see, it has problem that needs a workaround.
    -   Local testing before committing and pushing
        -   Before committing and pushing the package definition, it is good
            to first test building and using it locally, so that we can fix
            problem in the package definition.
        -   We may try to build it using `guix build`. With the `-L`
            option, we can specify extra path containing modules to be
            pre-pended to existing channels, so that we can test
            packages not yet committed to the channel.

            ```shell
            # use the previous channels file, so that its dependencies will be consistent.
            guix time-machine -C ~/channels.scm -- build -L ~/my-guix-pkgs/ r-foofactors

            ```

            But then this gave error that "ssh is not found", so
            `git-fetch` is unable to fetch from the private repository. Asking
            in the [guix-devel@gnu.org](https://lists.gnu.org/mailman/listinfo/guix-devel) mailing list confirms that currently
            `git-fetch` is indeed unable to fetch private repository over
            ssh, and Luis Felipe suggested a workaround of using
            `git-checkout` directly instead of using `origin` and
            `git-fetch`:

            -   <https://lists.gnu.org/archive/html/guix-devel/2021-07/msg00090.html>
            -   <https://lists.gnu.org/archive/html/guix-devel/2021-07/msg00091.html>
        -   We therefore modify the package definition above to use
            `git-checkout` and confirm that it can be built successfully
            (assuming your SSH key and ssh-agent has been setup correctly):

            ```scheme
            (define-module (my-packages r-pkgs)
              #:use-module ((guix licenses) #:prefix license:)
              #:use-module (guix packages)
              #:use-module (guix download)
              #:use-module (guix git)
              #:use-module (guix git-download)
              #:use-module (guix utils)
              #:use-module (guix build-system r)
              #:use-module (gnu packages)
              #:use-module (gnu packages statistics))

            (define-public r-foofactors
              (let ((commit "ef71e8d2e82fa80e0cfc249fd42f50c01924326d")
                    (revision "1"))
                (package
                  (name "r-foofactors")
                  (version (git-version "0.0.0.9000" revision commit))
                  (source
                   (git-checkout
                    (url "git@github.com:peterloleungyau/foofactors.git")
                    (commit commit)))
                  (build-system r-build-system)
                  (propagated-inputs
                   `(("r-forcats" ,r-forcats)))
                  (home-page "https://github.com/jennybc/foofactors")
                  (synopsis "A R package to make factors less aggravating.")
                  (description
                   "Factors have driven people to extreme measures, like ordering
            custom conference ribbons and laptop stickers to express how HELLNO we
            feel about stringsAsFactors. And yet, sometimes you need them. Can they
            be made less maddening? Let's find out.")
                  (license license:expat))))

            ```

            Note that we have added the `(guix git)` module for `git-checkout`. We
            have also changed `origin` to `git-checkout`, and the source hash is
            no longer needed. Since a commit of a git repository is already
            some kind of hash, we can still be confident that the source has not been
            tampered with.
        -   Now that the package can be built, we also try it in a `guix
                   environment` which can also accept the `-L` option for extra
            module path, together with R:

            ```shell
            guix time-machine -C ~/channels.scm -- environment -L ~/my-guix-pkgs/ --ad-hoc r-foofactors r-minimal -- R
            ```

            This should give an R REPL, then we may try the quick demo
            shown in the README of <https://github.com/jennybc/foofactors>
    -   After the local testing, we may update the private channel,
        commit and push:

        ```shell
        # get to the private repository directory
        cd ~/my-guix-pkgs/
        # add, commit, push
        git add my-packages/r-pkgs.scm
        git commit -m "Added r-foofactors."
        git push
        # also note the latest commit
        git log
        ```
3.  Test the package from Guix through the private channel
    1.  Update the channels file for updated commit

        With the updated private channel, we also update the channels
        file `~/channels.scm` for updated commit, which is
        `13255b9d3550bc8b2ff8b987d3d5621447783a3f` from `git log`:

        ```scheme
        (list (channel
               (name 'guix)
               (url "https://git.savannah.gnu.org/git/guix.git")
               (commit "9904a15a4c838362673c1affdbaf1e83d92fe8ff"))
              (channel
               (name 'my-guix-pkgs)
               (url "git@github.com:peterloleungyau/my-guix-pkgs.git")
               (commit "13255b9d3550bc8b2ff8b987d3d5621447783a3f")
               (branch "master")))
        ```

        In general, when a channel is updated, it is just a simple
        matter of updating the commit in channels file as above, _when_
        a project needs to use the updated package(s),
    2.  Try to create a Guix environment with updated channels file

        ```shell
        guix time-machine -C ~/channels.scm -- environment --ad-hoc r-foofactors r-minimal -- R
        ```

        And we can try the same quick demo as above, and verify that we
        get the expected results.


## (Optional) Channel Authentication {#optional--channel-authentication}

The above setup of private channel and custom packages is sufficient
for needs of teams for internal packages. You may choose to add
channel authentication.
References:

-   [Creating a Channel](https://guix.gnu.org/manual/en/html%5Fnode/Creating-a-Channel.html#Creating-a-Channel)
-   [Channel Authentication](https://guix.gnu.org/manual/en/html%5Fnode/Channel-Authentication.html)
-   [Specifying Channel Authorizations](https://guix.gnu.org/manual/en/html%5Fnode/Specifying-Channel-Authorizations.html)

Here we just briefly point to some relevant material.  There are a few
things needed:

1.  Setup OpenPGP key for each committer
    -   e.g. refer to <https://askubuntu.com/questions/100281/how-do-i-make-a-pgp-key>
    -   e.g. refer to [Installing and Using PGP](https://nsrc.org/workshops/2014/btnog/raw-attachment/wiki/Track3Agenda/2-1-1.pgp-lab.html)
2.  Export the OpenPGP key of all committers, e.g. with `gpg --export`
    -   refer to [Exporting your public key with GPG](https://nsrc.org/workshops/2014/btnog/raw-attachment/wiki/Track3Agenda/2-1-1.pgp-lab.html#exporting-your-public-key-with-gpg)
3.  Introduce an initial `.guix-authorizations` which lists the keys
    of each authorized developer. And the commit should be signed.
    -   refer to [Commit Access](https://guix.gnu.org/manual/en/html%5Fnode/Commit-Access.html) for signing git commits
    -   an example `.guix-authorizations` is:

        ```scheme
        ;; Example '.guix-authorizations' file.

        (authorizations
         (version 0)               ;current file format version

         (("AD17 A21E F8AE D8F1 CC02  DBD9 F8AE D8F1 765C 61E3"
           (name "alice"))
          ("2A39 3FFF 68F4 EF7A 3D29  12AF 68F4 EF7A 22FB B2D5"
           (name "bob"))
          ("CABB A931 C0FF EEC6 900D  0CFB 090B 1199 3D9A EBB5"
           (name "charlie"))))
        ```
4.  Put all the OpenPGP keys that were ever mentioned in
    `.guix-authorizations`, stored as (either binary or ASCII-armored)
    `.key` files, put in the branch named `keyring`.
5.  Advertise the channel introduction, for instance in the README of
    the channel. The channel introduction looks like this and contains
    the key for the first commit of the channel:

    ```scheme
    (channel
      (name 'some-channel)
      (url "https://example.org/some-channel.git")
      (introduction
       (make-channel-introduction
        "6f0d8cc0d88abb59c324b2990bfee2876016bb86"
        (openpgp-fingerprint
         "CABB A931 C0FF EEC6 900D  0CFB 090B 1199 3D9A EBB5"))))
    ```

You may look at these examples of third party channels to see the
parts mentioned above:

-   <https://gitlab.com/nonguix/nonguix>
-   <https://github.com/guix-science/guix-science.git>


## Summary {#summary}

We have demonstrated setting up a private channel as a private GitHub
repository, for internal R packages, where the package source can be
in private GitHub repository. This setup is sufficient for using Guix
for per-project dependency management in the presence of internal
packages, by maintaining a channels file and a manifest file for each
project. We also briefly point out the steps needed to authenticate
the private channel, though this is optional.