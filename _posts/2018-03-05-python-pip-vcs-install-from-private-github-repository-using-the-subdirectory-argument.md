---
title: Python Pip VCS Install from Private github Repository using the Subdirectory Argument
layout: post
---

Woah ... that title is a mouthful. Lemme break it down: 

  1. My python package(s) are in a private github repository (e.g. `www.github.com/safijari/my_repo`)
  2. The correct `setup.py` file is not in the root folder of the repo. (e.g. it's actually located at `my_package/setup.py`)
  3. I want to install it directly from github using pip.
  4. I use an ssh link to clone the repo. (e.g. `git@github.com:safijari/my_repo.git`).
Apparently the command to install `my_package` directly from the repo is: [code lang=text] pip install -e "git+git@github.com:safijari/my_repo.git#egg=my_package&subdirectory=my_package" [/code] That ... is ... long...! Let's break it down: 

  * `pip install -e ...` installs the package in `editable` mode, which is the only option for ssh based github repo access. **Note**: The **quotes** are super important in this specific use-case, DO NOT FORGET THEM.
  * `git+git@github.com:safijari/my_repo.git` is the actual url but with `git+` prepended. This signals to pip that the package is being installed from a VCS.
  * `#egg=my_package` tells pip that you want to install a package with the name `my_package`.
  * &`subdirectory=my_package` is the piece that allows the setup.py file to be located somewhere other than the repo root. This is also the piece that breaks if you do not use the aforementioned quotes!!!
And there you have it. An editable source install for the package will be done, same as any other source install. The ability to do this promises to simplify a big part of my workflow, hope it helps you too :).
