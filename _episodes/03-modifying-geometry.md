---
title: "Modifying geometry"
teaching: 30
exercises: 30
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

> Quick Exercise:
> - Ensure that you have a local copy of the geometry repository which you can compile and install in a local directory.
> - Verify that, after sourcing the `setup.sh` script, the `DETECTOR_PATH` points to the correct local install directory.
> - Verify that `dd_web_display` can indeed export the geometry for the detector subsystem configuration you used before.
{: .challenge}

## Anatomy of a detector plugin

# Introduction

It may be clear at this point how to make modifications to the parametrization, commit them to a branch in the local repository, and submit a pull request on GitHub to include them in the main branch.

For the remainder of this lesson we will focus on the vertex barrel detector using the much reduced geometry configuration `epic_vertex_only.xml` so that any change made are more evident.

What we have not covered yet is the discussion of what goes into a detector plugin. Let's look at the `epic_VertexBarrel` plugin we encountered earlier. The names of the plugins may not agree with the source files in the `src/` directory. This allows us to support multiple detector types with the same source files. In this case, `epic_VertexBarrel` is defined in the file `src/BarrelTrackerWithFrame_geo.cpp`.

If you are not sure what fine in the `src` directory builds the plugin you are looking for, find the `type` in the xml detector definition and use the `grep` shell command.

```console
$ grep -r epic_VertexBarrel src/
src/BarrelTrackerWithFrame_geo.cpp:DECLARE_DETELEMENT(epic_VertexBarrel,    create_BarrelTrackerWithFrame)
```

The DECLARE_DETELEMENT line, at the bottom of the `src/BarrelTrackerWithFrame_geo.cpp` file provides dd4hep with the link which tells it to call the `create_BarrelTrackerWithFrame` function when an xml detector definition is given the `epic_VertexBarrel` type.

We now know that changing the content of the `create_BarrelTrackerWithFrame` function should be called when the `epic_vertex_only.xml` is used when dd4hep loads a geometry.

# Passing parameters from xml

Next we will take a deeper dive into the `create_BarrelTrackerWithFrame` function to pick out the key components and how it is configured by the xml file `compact/tracking/vertex_barrel.xml`

```code
static Ref_t create_BarrelTrackerWithFrame(Detector& description, xml_h e, SensitiveDetector sens)
```

here `description` contains access to the full tree defined from the main detector xml file. `e` contains the specific information contained within the xml tree within the `<detector>` blocks.

There are a few ways to access these xml configuration parameters. Some xml elements have methods provided by dd4hep which allow direct access to the values, such as:

```code
51  Material                     air      = description.air();
52  int                          det_id   = x_det.id();
53  string                       det_name = x_det.nameStr();
```

A list of tags dd4hep provides a conventient conversion method for can be found [here](https://dd4hep.web.cern.ch/dd4hep/reference/UnicodeValues_8h_source.html), (**There is almost certainly a better link**).

More often you may be wanting to define a parameter by a tag of your choice or if you're wanting to be certain how it's being handled. The following (abridged) code is an example of how to access parameters of any name.

```
  for (xml_coll_t su(x_det, _U(support)); su; ++su) {
    xml_comp_t  x_support         = su;
    double      support_thickness = getAttrOrDefault(x_support, _U(thickness), 2.0 * mm);
    double      support_length    = getAttrOrDefault(x_support, _U(length), 2.0 * mm);
    double      support_rmin      = getAttrOrDefault(x_support, _U(rmin), 2.0 * mm);
    double      support_zstart    = getAttrOrDefault(x_support, _U(zstart), 2.0 * mm);
    std::string support_name      = getAttrOrDefault<std::string>(x_support, _Unicode(name), "support_tube");
    std::string support_vis       = getAttrOrDefault<std::string>(x_support, _Unicode(vis), "AnlRed");
  }
```

The code loops over all `<support>` elements inside the `<detector>` block, located using `_U(support)` which interprets content as unicode. Inside each support node the `getAttrOrDefault` method sets the variables to the values given by the unicode `thickness` etc, or if they are not present in the support node sets a default value. If you want to require the parameter be defined in the xml file you could simply use `x_support.child(thickness)`.

> Note:
> - No support blocks actually currently appear in the `compact/tracking/vertex_barrel.xml` file so this block of code won't be run.
> - In the xml file, there must be no white space between the parameter, = sign and the value.
{: .callout}

A fuller description of how to access and use the xml parameters is given in section [2.4 of the dd4hep manual](https://dd4hep.web.cern.ch/dd4hep/usermanuals/DD4hepManual/DD4hepManualch2.html#x3-210002.4)


> Exercise:
> - Create and chackout a new branch forked from the main branch.
> - Add a new configuration parameter into `compact/tracking/vertex_barrel.xml` 
> - Add code to `src/BarrelTrackerWithFrame_geo.cpp` which will read the new parameter and a print statement to display its value to the terminal.
> - Recompile and rerun the `dd_web_display` step using `epic_vertex_only.xml` to verify that the printout statement has been added.
> - Change the values in the xml and rerun to verify the value is being read properly.
{: .challenge}

> Note: Changes to the xml files in `install/share/epic/`... can made without recompiling the code, however they will be overwritten when the code is recompiled. In order to test temporary changes a top level configuration file can be copied to a path outside of `install`. This then needs to be edited to internally point to the compact file you are editing rather than the path given by the install, `${DETECTOR_PATH}`.
{: .callout}

# Building new components

DD4hep geometries are built in a similar hierarchical way to Geant4 geometries.

> - Shape - 3D shape of the component. Can be a simple shape, made from boolean combinations of simple shapes or imported from CAD.
> - Volume - Adds physical properties to the shape such as its material and if its sensitive.
> - Placement - Position(s) that the volume is located within your detector geometry, this can be a position nested within another volume, containing unique identifier.

We will start by looking at the shapes anv volumes. The most common shapes you are likely to find yourself using are `Box` and `Tube`. In `src/BarrelTrackerWithFrame_geo.cpp` both are used, `module_component` is defined as `Box`, the volume of which takes its material from the xml description:

```code
172      Box          c_box(x_comp.width() / 2, x_comp.length() / 2, x_comp.thickness() / 2);
173      Volume       c_vol(c_nam, c_box, description.material(x_comp.materialStr()));
```

`layer` volumes into which `module_component`s are later placed are described as a `Tube` but this time the volume is directly given the air material:

```code
239    Tube       lay_tub(x_barrel.inner_r(), x_barrel.outer_r(), x_barrel.z_length() / 2.0);
240    Volume     lay_vol(lay_nam, lay_tub, air); // Create the layer envelope volume.
```

In DD4hep there is a type of volume called an Assembly which contains volumes placed within itself but doesn't have a shape of its own. This is very useful for arranging volumes without needing a container volume defined with might have overlaps of their own where none are really present.

> Notes: 
> - The [DD4hep shapes](https://dd4hep.web.cern.ch/dd4hep/usermanuals/DD4hepManual/DD4hepManualch2.html#x3-290002.9) are based directly off the [ROOT geometry shapes](https://root.cern.ch/root/htmldoc/guides/users-guide/Geometry.html#shapes). An useful, probably out of date table comparing the ROOT, DD4hep and Geant4 shape names can be found here:  [https://github.com/AIDASoft/DD4hep/issues/588](https://github.com/AIDASoft/DD4hep/issues/588)
> - DD4hep provides some shape plugins directly, so very basic geometries can be described directly in the xml with no additional code.
{: .callout}

> Exercise:
> - Create a new simple volume within the hierachy in `src/BarrelTrackerWithFrame_geo.cpp`.
> - Recompile and rerun the `dd_web_display` step using `epic_vertex_only.xml` locating the new shape(s) you have added in the ROOTJS viewer.
> - Build a new tube volume which contains tracking layers.
{: .challenge}

# Testing overlaps

It is important for running the Geant4 simulation that geometries do not overlap. When stepping through the geometry a particle cannot know which volume it is in. An overlap check is run by GitHub when you request that your changes are merged into the main branch of the epic code.

```
python scripts/checkOverlaps.py -c ${DETECTOR_PATH}/epic_vertex_only.xml
```

> Exercise:
> - Run the overlap check on your geometry with the added component.
> - Change some parameters to add/remove the overlap and compare the output.
{: .challenge}

# Readout 

Placed volumes can be made sensitive by setting e.g.

```code
194  c_vol.setSensitiveDetector(sens);
```

The type of information that will be saved to the output is defined usually as either:

```code
  sens.setType("tracker");
  sens.setType("calorimeter");
```

In the xml file the readout for the detector is passed in the `readout` field of the `detector` definition

```code
    <detector
      id="VertexBarrel_0_ID"
      name="VertexBarrel"
      type="epic_VertexBarrel"
      readout="VertexBarrelHits"
      insideTrackingVolume="true">
```

Where readout references a `readout` also defined in the xml description

```code
  <readouts>
    <readout name="VertexBarrelHits">
      <segmentation type="CartesianGridXY" grid_size_x="0.020*mm" grid_size_y="0.020*mm" />
      <id>system:8,layer:4,module:12,sensor:2,x:32:-16,y:-16</id>
    </readout>
  </readouts>
```

Here the name given will appear as the name of the branch containing the hits in the output edm4hep file. dd4hep provides very convenient segmentation to the readout which allows hits in a readout volume to be divided up to locations beyond its natural boundaries, this is configures by the `x` and `y` parameters as well as the `grid_size`.

The readout branch contains the information on the hit energy deposited, time of arival etc. which is usually found in a simulation output but in addition it contains `CellID` which is a 64 bit field which uniquely identifies the detector segmentation. 

In the case of `VertexBarrelHits`, 8 bits always required by the system, 4 bits locate a specific layer, 12 a module, 2 a sensor and 32 the remaining x-y segmentation. In the code, dd4hep requires a separate hierachy of the geometry detector elements which are given tags and numbers so they can be uniquely identified. This hieracy doesn't have to strictly follow the way the volumes are themselves constructed.

```
  DetElement mod_elt(lay_elt, module_name, module);
  pv = lay_vol.placeVolume(module_env, tr);
  pv.addPhysVolID("module", module);
  mod_elt.setPlacement(pv);
```

Here `mod_elt` is give the parent element `layer_elt`, the name and module number. Then the element is attached to a placed volume which has been given the physical volume id `module`.

> Exercise:
> - Run the simulation...
> - Change the sensitive type of the `BarrelTrackerWithFrame_geo.cpp` and compare the output to what you first saw.
> - Try to make your new tube volume sensitive by setting as sensitive and adding a `DetElement` and giving it the necessary `addPhysVolID`.

# ACTS?

ToDo