---
title: "Modifying geometry"
teaching: 20
exercises: 20
questions:
- "How do we modify or add geometry defined in DD4hep?"
objectives:
- "Understand the structure of a geometry plugin source file."
keypoints:
- "To add or modify geometry, we add geometry plugins written in C++."
---
Until now we have interacted with the read-only geometry that is stored inside the container. We will now move on to modifying this geometry. That requires that we work in a local copy of the geometry.

## Checking out your local copy of the geometry

We will start with a local copy of the `epic` repository:
```console
$ cd ~/eic/
$ git clone https://github.com/eic/epic
$ cd epic
$ ls
bin    calibrations    compact         macro      reports           scripts  templates
build  CMakeLists.txt  configurations  README.md  requirements.txt  src      views
```

As you can tell, the content of the repository itself is quite different from the installed version (as is often the case for other software as well). You will recognize, however, the `compact` directory with the subsystem xml files.

In order to compile and install the local geometry repository into a local directory, we can use the following commands:
```console
$ cd ~/eic/epic
$ cmake -B build -S . -DCMAKE_INSTALL_PREFIX=install
$ cmake --build build -- install
```

> Note: To speed up compilation, you may add the options `-j4` to the last command, where `4` corresponds to the number of cores you can use.
{: .callout}

This will install the geometry into the directory `~/eic/epic/install/` (and subdirectories). You will notice that `~/eic/epic/install/share/epic` contains the same files that we explored earlier inside the `/opt/detector` directory.

As before, we now need to load the environment for this geometry. We can again use the `setup.sh` script for this, though now we must use the one installed in our local installation directory:
```console
$ source install/setup.sh
```

When we run `dd_web_display --export $DETECTOR_PATH/$DETECTOR_CONFIG.xml` now, we will use the local geometry parametrization and the local geometry plugins. (Note: As before, downloads of fieldmaps and calibration files will be necessary.)

> Exercise:
> - Ensure that you have a local copy of the geometry repository which you can compile and install in a local directory.
> - Verify that, after sourcing the `setup.sh` script, the `DETECTOR_PATH` points to the correct local install directory.
> - Verify that `dd_web_display` can indeed export the geometry for the detector subsystem configuration you used before.
{: .challenge}

## Anatomy of a detector plugin

It may be clear at this point how to make modifications to the parametrization, commit them to a branch in the local repository, and submit a pull request on GitHub to include them in the main branch.

What we have not covered yet is the discussion of what goes into a detector plugin. Let's look at the `epic_VertexBarrel` plugin we encountered earlier. The names of the plugins may not agree with the source files in the `src/` directory. This allows us to support multiple detector types with the same source files. In this case, `epic_VertexBarrel` is defined in the file `src/BarrelTrackerWithFrame_geo.cpp`.

> Exercise:
> - Add a printout to the detector geometry plugin that corresponds to the type you picked earlier.
> - Recompile and rerun the `dd_web_display` step to verify that the printout statement has been added.
{: .challenge}
