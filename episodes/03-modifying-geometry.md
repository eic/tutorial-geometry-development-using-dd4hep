---
title: "Modifying geometry"
teaching: 20
exercises: 40
---

::::::::::::::::::::::::::::::::::::::::::::: questions

- How do we modify or add geometry defined in DD4hep?

:::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::: objectives

- Understand the structure of a geometry plugin source file.

:::::::::::::::::::::::::::::::::::::::::::::

Until now we have interacted with the read-only geometry that is stored inside the container. We will now move on to modifying this geometry. That requires that we work in a local copy of the geometry.

## Checking out your local copy of the geometry

We will start with a local copy of the `epic` repository:

```bash
$ cd ~/eic/
$ git clone https://github.com/eic/epic
$ cd epic
$ ls
bin    calibrations    compact         macro      reports           scripts  templates
build  CMakeLists.txt  configurations  README.md  requirements.txt  src      views
```

As you can tell, the content of the repository itself is quite different from the installed version (as is often the case for other software as well). You will recognize, however, the `compact` directory with the subsystem xml files.

### Important: to follow this tutorial we will need to checkout an older version of the geometry

```bash
$ git checkout 25.08.0
```

In order to compile and install the local geometry repository into a local directory, we can use the following commands:

```bash
$ cd ~/eic/epic
$ cmake -B build -S . -DCMAKE_INSTALL_PREFIX=install
$ cmake --build build -- install
```

::::::::::::::::::::::::::::::::::::::::::::: callout

Note: To speed up compilation, you may add the option `-j4` to the last command, where `4` corresponds to the number of cores you can use.

:::::::::::::::::::::::::::::::::::::::::::::

This will install the geometry into the directory `~/eic/epic/install/` (and subdirectories). You will notice that `~/eic/epic/install/share/epic` contains the same files that we explored earlier inside the `/opt/detector` directory.

As before, we now need to load the environment for this geometry. We can again use the `bin/thisepic.sh` script for this, though now we must use the one installed in our local installation directory:

```bash
$ source install/bin/thisepic.sh
$ env | grep DETECTOR
```

You should now see that the `$DETECTOR_PATH` variable points to your local install path.

When we run `dd_web_display --export $DETECTOR_PATH/$DETECTOR_CONFIG.xml` now, we will use the local geometry parametrization and the local geometry plugins. (Note: As before, downloads of fieldmaps and calibration files will be necessary.)

::::::::::::::::::::::::::::::::::::::::::::: challenge

## Quick Exercise: build a local geometry

- Ensure that you have a local copy of the geometry repository which you can compile and install in a local directory.
- Verify that, after sourcing the `bin/thisepic.sh` script, the `DETECTOR_PATH` points to the correct local install directory.
- Verify that `dd_web_display` can indeed export the geometry for the detector subsystem configuration you used before.

::::::::::::::: solution

After `cmake -B build -S . -DCMAKE_INSTALL_PREFIX=install` and `cmake --build build -- install`,
sourcing `install/bin/thisepic.sh` sets `DETECTOR_PATH` to `~/eic/epic/install/share/epic`.
Running `dd_web_display --export $DETECTOR_PATH/epic_vertex_only.xml` then exports the geometry from
your local build.

:::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::

## Anatomy of a detector plugin

### Introduction

It may be clear at this point how to make modifications to the parametrization, commit them to a branch in the local repository, and submit a pull request on GitHub to include them in the main branch.

For the remainder of this lesson we will focus on the vertex barrel detector using the much reduced geometry configuration `epic_vertex_only.xml` so that any change made are more evident.

What we have not covered yet is the discussion of what goes into a detector plugin. Let's look at the `epic_VertexBarrel` plugin we encountered earlier. The names of the plugins may not agree with the source files in the `src/` directory. This allows us to support multiple detector types with the same source files. In this case, `epic_VertexBarrel` is defined in the file `src/BarrelTrackerWithFrame_geo.cpp`.

If you are not sure what file in the `src` directory builds the plugin you are looking for, find the `type` in the xml detector definition and use the `grep` shell command.

```bash
$ grep -r epic_VertexBarrel src/
src/BarrelTrackerWithFrame_geo.cpp:DECLARE_DETELEMENT(epic_VertexBarrel,    create_BarrelTrackerWithFrame)
```

The DECLARE_DETELEMENT line, at the bottom of the `src/BarrelTrackerWithFrame_geo.cpp` file provides dd4hep with the link which tells it to call the `create_BarrelTrackerWithFrame` function when an xml detector definition is given the `epic_VertexBarrel` type.

We now know that changing the content of the `create_BarrelTrackerWithFrame` function should be called when the `epic_vertex_only.xml` is used when dd4hep loads a geometry.

### Passing parameters from xml

Next we will take a deeper dive into the `create_BarrelTrackerWithFrame` function to pick out the key components and how it is configured by the xml file `compact/tracking/vertex_barrel.xml`

```c++
static Ref_t create_BarrelTrackerWithFrame(Detector& description, xml_h e, SensitiveDetector sens)
```

here `description` contains access to the full tree defined from the main detector xml file. `e` contains the specific information contained within the xml tree within the `<detector>` blocks.

There are a few ways to access these xml configuration parameters. Some xml elements have methods provided by dd4hep which allow direct access to the values, such as:

```c++
Material                     air      = description.air();
int                          det_id   = x_det.id();
string                       det_name = x_det.nameStr();
```

A list of tags for which dd4hep provides a convenient conversion method can be found in the [dd4hep UnicodeValues header](https://dd4hep.web.cern.ch/dd4hep/reference/UnicodeValues_8h_source.html).

More often you may be wanting to define a parameter by a tag of your choice or if you're wanting to be certain how it's being handled. The following (abridged) code is an example of how to access parameters of any name.

```c++
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

::::::::::::::::::::::::::::::::::::::::::::: callout

Note:

- No support blocks actually currently appear in the `compact/tracking/vertex_barrel.xml` file so this block of code won't be run.
- In the xml file, there must be no white space between the parameter, = sign and the value.

:::::::::::::::::::::::::::::::::::::::::::::

A fuller description of how to access and use the xml parameters is given in [section 2.4 of the dd4hep manual](https://dd4hep.web.cern.ch/dd4hep/usermanuals/DD4hepManual/DD4hepManualch2.html#x3-210002.4).

::::::::::::::::::::::::::::::::::::::::::::: challenge

## Exercise: add a configuration parameter

- Create and checkout a new branch forked from the 25.08.0 branch.
- Add a new configuration parameter into `compact/tracking/vertex_barrel.xml`.
- Add code to `src/BarrelTrackerWithFrame_geo.cpp` which will read the new parameter and a print statement to display its value to the terminal.
- Recompile and rerun the `dd_web_display` step using `epic_vertex_only.xml` to verify that the printout statement has been added.
- Change the values in the xml and rerun to verify the value is being read properly.

::::::::::::::: solution

Add a `<constant>` (or an attribute on the `<detector>` element) in `vertex_barrel.xml`, read it in
`create_BarrelTrackerWithFrame` with `getAttrOrDefault(x_det, _Unicode(yourparam), default)`, and
print it with e.g. `std::cout`. After `cmake --build build -- install` and re-running
`dd_web_display`, your printout appears in the terminal and changes when you edit the xml value.

:::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::: callout

Note: Changes to the xml files in `install/share/epic/`... can be made without recompiling the code, however they will be overwritten when the code is recompiled. In order to test temporary changes a top level configuration file can be copied to a path outside of `install`. This then needs to be edited to internally point to the compact file you are editing rather than the path given by the install, `${DETECTOR_PATH}`.

:::::::::::::::::::::::::::::::::::::::::::::

### Building new components

DD4hep geometries are built in a similar hierarchical way to Geant4 geometries.

- Shape - 3D shape of the component. Can be a simple shape, made from boolean combinations of simple shapes or imported from CAD.
- Volume - Adds physical properties to the shape such as its material and if its sensitive.
- Placement - Position(s) that the volume is located within your detector geometry, this can be a position nested within another volume, containing unique identifier.

We will start by looking at the shapes and volumes. The most common shapes you are likely to find yourself using are `Box` and `Tube`. In `src/BarrelTrackerWithFrame_geo.cpp` both are used, `module_component` is defined as `Box`, the volume of which takes its material from the xml description:

```c++
      Box          c_box(x_comp.width() / 2, x_comp.length() / 2, x_comp.thickness() / 2);
      Volume       c_vol(c_nam, c_box, description.material(x_comp.materialStr()));
```

`layer` volumes into which `module_component`s are later placed are described as a `Tube` but this time the volume is directly given the air material:

```c++
    Tube       lay_tub(x_barrel.inner_r(), x_barrel.outer_r(), x_barrel.z_length() / 2.0);
    Volume     lay_vol(lay_nam, lay_tub, air); // Create the layer envelope volume.
```

In DD4hep there is a type of volume called an Assembly which contains volumes placed within itself but doesn't have a shape of its own. This is very useful for arranging volumes without needing a container volume defined which might have overlaps of their own where none are really present.

::::::::::::::::::::::::::::::::::::::::::::: callout

Notes:

- The [DD4hep shapes](https://dd4hep.web.cern.ch/dd4hep/usermanuals/DD4hepManual/DD4hepManualch2.html#x3-290002.9) are based directly off the [ROOT geometry shapes](https://root.cern.ch/root/htmldoc/guides/users-guide/Geometry.html#shapes). A useful, probably out of date table comparing the ROOT, DD4hep and Geant4 shape names can be found in [DD4hep issue #588](https://github.com/AIDASoft/DD4hep/issues/588).
- DD4hep provides some shape plugins directly, so very basic geometries can be described directly in the xml with no additional code.

:::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::: challenge

## Exercise: build a new volume

- Create a new simple volume within the hierachy in `src/BarrelTrackerWithFrame_geo.cpp`.
- Recompile and rerun the `dd_web_display` step using `epic_vertex_only.xml` locating the new shape(s) you have added in the ROOTJS viewer.
- Build a new tube volume which contains tracking layers.

::::::::::::::: solution

Construct a shape (e.g. `Tube` or `Box`), wrap it in a `Volume` with a material, and place it with
`placeVolume` inside an existing volume in `create_BarrelTrackerWithFrame`. After
`cmake --build build -- install` and re-exporting with `dd_web_display`, the new shape appears in the
JSROOT viewer.

:::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::

### Testing overlaps

It is important for running the Geant4 simulation that geometries do not overlap. When stepping through the geometry a particle cannot know which volume it is in. An overlap check is run by GitHub when you request that your changes are merged into the main branch of the epic code.

```bash
python scripts/checkOverlaps.py -c ${DETECTOR_PATH}/epic_vertex_only.xml
```

::::::::::::::::::::::::::::::::::::::::::::: challenge

## Exercise: run the overlap check

- Run the overlap check on your geometry with the added component.
- Change some parameters to add/remove the overlap and compare the output.

::::::::::::::: solution

`python scripts/checkOverlaps.py -c ${DETECTOR_PATH}/epic_vertex_only.xml` reports any overlapping
volumes with their names and overlap distance. After moving your new volume so it intersects a
neighbour, the check lists the new overlap; moving it clear again removes it from the report.

:::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::

### Readout

Placed volumes can be made sensitive by setting e.g.

```c++
  c_vol.setSensitiveDetector(sens);
```

The type of information that will be saved to the output is defined usually as either:

```c++
  sens.setType("tracker");
  sens.setType("calorimeter");
```

In the xml file the readout for the detector is passed in the `readout` field of the `detector` definition

```xml
    <detector
      id="VertexBarrel_0_ID"
      name="VertexBarrel"
      type="epic_VertexBarrel"
      readout="VertexBarrelHits"
      insideTrackingVolume="true">
```

Where the readout name references a `readout` block also defined in the xml description

```xml
  <readouts>
    <readout name="VertexBarrelHits">
      <segmentation type="CartesianGridXY" grid_size_x="0.020*mm" grid_size_y="0.020*mm" />
      <id>system:8,layer:4,module:12,sensor:2,x:32:-16,y:-16</id>
    </readout>
  </readouts>
```

Here the name given will appear as the name of the branch containing the hits in the output edm4hep file. dd4hep provides very convenient segmentation to the readout which allows hits in a readout volume to be divided up to locations beyond its natural boundaries, this is configured by the `x` and `y` parameters as well as the `grid_size`.

The readout branch contains the information on the hit energy deposited, time of arival etc. which is usually found in a simulation output but in addition it contains `CellID` which is a 64 bit field which uniquely identifies the detector segmentation.

In the case of `VertexBarrelHits`, 8 bits always required by the system, 4 bits locate a specific layer, 12 a module, 2 a sensor and 32 the remaining x-y segmentation. In the code, dd4hep requires a separate hierachy of the geometry detector elements which are given tags and numbers so they can be uniquely identified. This hieracy doesn't have to strictly follow the way the volumes are themselves constructed.

```c++
  DetElement mod_elt(lay_elt, module_name, module);
  pv = lay_vol.placeVolume(module_env, tr);
  pv.addPhysVolID("module", module);
  mod_elt.setPlacement(pv);
```

Here `mod_elt` is given the parent element `layer_elt`, the name and module number. Then the element is attached to a placed volume which has been given the physical volume id `module`.

To run the simulation and produce an output file containing the detector hits you can use `npsim`. I would suggest only using a small sample of events given by the `--numberOfEvents` flag.

```bash
$ npsim --runType run --compactFile $DETECTOR_PATH/epic_vertex_only.xml --inputFiles root://dtn-eic.jlab.org//volatile/eic/EPIC/EVGEN/SIDIS/pythia6-eic/1.0.0/18x275/q2_0to1/pythia_ep_noradcor_18x275_q2_0.000000001_1.0_run9.ab.hepmc3.tree.root --numberOfEvents 100 --outputFile test.edm4hep.root
```

Inside the output file `test.edm4hep.root` there should be 4 trees:

```
events
runs
metadata
podio_metadata
```

The events tree contains the readout of your detectors, in this example it should contain only 7 branches,

```
MCHeader
MCParticles
_MCParticles_parents
_MCParticles_daughters
VertexBarrelHits
_VertexBarrelHits_MCParticle
```

The `MCParticles` branch contains information on all of the particles described by your generator and any secondaries produced in the simulation. `VertexBarrelHits` contains the hit information of the vertex barrel and has the association branch `_VertexBarrelHits_MCParticle` which references the particle in the `MCParticles` branch which caused the hit.

::::::::::::::::::::::::::::::::::::::::::::: challenge

## Exercise: run a simulation and inspect the output

- Run the simulation with a small dataset.
- Look at the output datafiles using your favorite ROOT browser.
- Change the sensitive type of the `BarrelTrackerWithFrame_geo.cpp` and compare the output to what you first saw.
- Try to make your new tube volume sensitive by setting it as sensitive and adding a `DetElement` and giving it the necessary `addPhysVolID` values not currently used by the tracker.
- Open ended - Move your new Tube detector into a separate src file (or create a new simple detector) and include it separately in the xml definition and readout.

::::::::::::::: solution

Running `npsim ... --numberOfEvents 100 --outputFile test.edm4hep.root` produces a file whose
`events` tree holds the `VertexBarrelHits` collection. Switching `sens.setType("tracker")` to
`"calorimeter"` changes the stored hit type. Making the new tube sensitive requires
`setSensitiveDetector(sens)`, a matching `DetElement`, and an `addPhysVolID` value that does not
collide with the existing `layer`/`module`/`sensor` fields in the readout `<id>`.

:::::::::::::::

:::::::::::::::::::::::::::::::::::::::::::::

### ACTS?

ToDo

::::::::::::::::::::::::::::::::::::::::::::: keypoints

- To add or modify geometry, we add geometry plugins written in C++.

:::::::::::::::::::::::::::::::::::::::::::::
