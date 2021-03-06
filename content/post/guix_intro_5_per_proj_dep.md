+++
title = "Guix Introduction Part 5: Per-Project Depedency Management"
date = 2021-05-13
tags = ["Guix", "Functional Package Manager", "Reproducibility"]
categories = ["Guix"]
draft = false
author = "Peter Lo"
+++

This is the fifth part of a brief introduction to the Guix functional
package manager, and how it could be used to manage dependencies of
projects, much like virtual environments for Python, but with much
larger scope.

This time we show a little demo of managing per-project dependencies
with Guix, through the `guix time-machine` and `guix environment`
commands.


## Demo settings {#demo-settings}

Suppose we have two projects `~/guix_demo/proj1` and
`~/guix_demo/proj2`, each with different channel file `channels.scm`
and manifest file `pkgs.scm`, so the two projects use different
versions of R (proj1 at R 3.6.3, proj2 at R 4.0.3) and different R
packages (at different versions).

Below we spawn two shells that allow us to work on the two projects
_at the same time_, and we show most of the output. If you try the
commands, you should get the same outputs, except possibly the locale
output from R which depends on your local setting.


### Sample files {#sample-files}

You may either manually create the following files, or clone the git
repository [https://github.com/peterloleungyau/guix\_demo](https://github.com/peterloleungyau/guix%5Fdemo) created for
convenience (you should check that the files are the same as below).

If you are in Guix but somehow do not have `git` installed yet, you
may install it and then clone the repository by:

```shell
# install git, and also need ssh
guix package -i git openssh
# clone the repo
cd ~
git clone https://github.com/peterloleungyau/guix_demo.git
```

-   First project
    -   `~/guix_demo/proj1/channels.scm`:

        ```scheme
        (list (channel
               (name 'guix)
               (url "https://git.savannah.gnu.org/git/guix.git")
               (commit "89909327d017198969436237acc7c93823ff8147")))
        ```

    -   `~/guix_demo/proj1/pkgs.scm`:

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

    -   `~/guix_demo/proj1/fit1.R`

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
    -   `~/guix_demo/proj2/channels.scm`

        ```scheme
        (list (channel
               (name 'guix)
               (url "https://git.savannah.gnu.org/git/guix.git")
               (commit "9904a15a4c838362673c1affdbaf1e83d92fe8ff")))
        ```

    -   `~/guix_demo/proj2/pkgs.scm`:

        ```scheme
        (specifications->manifest
         '(
           ;; R
           "r"
           "r-glmnet"
           "r-jsonlite"
           ))
        ```

    -   `~/guix_demo/proj2/fit2.R`

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


## Open a shell for the first project {#open-a-shell-for-the-first-project}

First open a new shell with Guix (could be the Guix package manager on
a Linux distribution, or the Guix system) to create an environment for
proj1:

```shell
cd ~/guix_demo/proj1
guix time-machine -C channels.scm -- environment --ad-hoc -m pkgs.scm -- R
```

Then wait a while for the R prompt to appear. The first time executing
the above may take quite some time to download and/or build the
packages. Subsequent runs should be much faster after the needed
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


## Open another shell for the second project {#open-another-shell-for-the-second-project}

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

You may try running some R commands in both shells to see that they
are running at the same time.


## Close both shells {#close-both-shells}

Since we directly run `R` when creating the `guix environment`, so
now quitting R with `q()` will also exit the spawn shell. So now we
can exit both shells spawn above.

If you now re-run the above commands, it should be much faster,
because the needed packages are cached, and Guix needs only to check
that the packages needed are there.


## Run R script for the first project {#run-r-script-for-the-first-project}

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


## Run R script for the second project {#run-r-script-for-the-second-project}

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


## A possible way to manage dependency using Guix on Jenkins {#a-possible-way-to-manage-dependency-using-guix-on-jenkins}

We briefly discuss using Guix to help manage dependencies for jobs
running on Jenkins.

{{< figure src="/ox-hugo/guix_intro_jenkins.png" caption="Figure 1: Illustration of using Guix for dependency management with Jenkins" >}}

-   **Basic ideas:**
    -   the Guix package manager can be installed along with Jenkins on
        a Linux distribution
    -   the Jenkins can be a single master node, or one master node with
        one or more worker nodes
    -   each master or worker node that may need to run jobs needs to
        have the Guix package manager installed
        -   if private channels are used, they need to be configured in
            each master or worker node
    -   optionally, one or more nodes can also be a substitution server by running `guix publish`
        -   the Guix in each master or worker node would need to be
            configured to have this extra substitution server
        -   if there is one node in the same network used for development
            (e.g. an VM in AWS Virtual Private Cloud), it is an ideal
            candidate to serve as the substitution server because during
            development, the needed packages for a project need to be
            downloaded or built anyway, so can be shared to other worker
            nodes when the script is later run in batch mode, to save
            re-building the packages in the worker nodes.
    -   the dependencies of each project (or job) is recorded in the
        associated git repository, e.g. by having one manifest file
        `pkgs.scm`, and one channels files `channels.scm`.
    -   the Jenkins job can run `guix time-machine` and `guix
              environment` as discussed above:

        ```shell
        guix time-machine -C channels.scm -- environment -m pkgs.scm -- THE_COMMAND_HERE
        ```
-   **Benefits:**
    -   due to the reproducibility of Guix, we can be confident that the
        packages used in development are the exact same versions as in
        the batch jobs
    -   the dependencies of each projects are also version controlled in
        the project git repository.
    -   this method is hassle-free:
        -   no need to manually install needed dependencies in needed worker nodes
        -   will not "forget" to update the dependencies in some worker nodes
        -   there will not conflicts of dependencies for different
            projects, because each job is run in a `guix environment`


## What's next? {#what-s-next}

In this part we showed a little demo of using Guix to manage
per-project dependencies using the `guix time-machine` and `guix
  environment` commands, mainly for batch script execution. Next time we
attempt to do the same per-project dependency management when you are
developing locally (not necessarily in Linux) and connecting to a remote
server (or a local VM) with Guix installed.
