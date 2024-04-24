---
title: Setup
---
In advance of the training session, please ensure that
1. You have a GitHub account ([sign up here](https://github.com/signup))
1. Your GitHub account is a member of the [EIC organization](https://github.com/eic) on GitHub
- Email [the EICUG SWG conveners](mailto:eic-software-l-owner@lists.bnl.gov) with your GitHub account to be added
1. You have singularity/apptainer or docker (on Mac) installed and working
- `module load singularity` (on most JLab or BNL systems) (preferred option)
- Local system options:
  - systems that use cvmfs or download images can install **singularity or apptainer** (see [here](https://apptainer.org/docs/user/main/quick_start.html))
  - systems that download images can install **docker** (see [here](https://www.docker.com/))
- you should at a minimum be able to run either of the following commands and open a shell:
  - `singularity run docker://alpine`
  - `docker run --rm -it alpine`
1. Download `eic-shell`:
- Due to time and bandwidth limitations, this should be downloaded to your desired system before the live tutorial
- From the system you use with singularity/apptainer or docker, make a folder called `eic`
- From inside this directory:
  - Install by running the command:
```
curl --location https://get.epic-eic.org | bash
```
  - **or** you can save the file at https://get.epic-eic.org as `install.sh` and run this script by hand:
```
wget --output-document install.sh https://get.epic-eic.org
bash install.sh
```

If you have completed the numbered steps above, then you are ready for the tutorial. Additional information and training links are below:
- Basic familiarity with Unix shell and Git
  - Software Carpentry [Unix Shell](https://swcarpentry.github.io/shell-novice/) training (recommended)
  - Software Carpentry [Basic Git](https://swcarpentry.github.io/git-novice/) training (recommended)
- Basic familiarity with singularity and docker:
  - Software Carpentry [Incubator Singularity](https://carpentries-incubator.github.io/singularity-introduction/) training (optional)
  - HEP Software Foundation [Docker](https://hsf-training.github.io/hsf-training-docker/index.html) training (optional)

{% include links.md %}
---
title: Setup
---
In advance of the training session, please ensure that
- You have a GitHub account ([sign up here](https://github.com/signup))
- Your GitHub account is a member of the [EIC organization](https://github.com/eic) on GitHub
  - Email [the EICUG SWG conveners](mailto:eicug-software-conveners@eicug.org) with your GitHub account to be added
- You have some basic familiarity with Unix shell and Git
  - Software Carpentry [Unix Shell](https://swcarpentry.github.io/shell-novice/) training (required)
  - Software Carpentry [Basic Git](https://swcarpentry.github.io/git-novice/) training (recommended)
- You have singularity, apptainer, or docker (on Mac) installed and working
  - `module load singularity` (on most JLab or BNL systems)
  - you should at a minimum be able to run either of the following commands and open a shell:
    - `singularity run docker://alpine`
    - `docker run --rm -it alpine`
  - Software Carpentry [Incubator Singularity](https://carpentries-incubator.github.io/singularity-introduction/) training (optional)
  - HEP Software Foundation [Docker](https://hsf-training.github.io/hsf-training-docker/index.html) training (optional)

{% include links.md %}
