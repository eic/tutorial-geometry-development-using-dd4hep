---
title: "Geometry Definition"
teaching: 20
exercises: 20
questions:
- "How do we define geometry using DD4hep?"
objectives:
- "Know where standard geometries as stored in `eic-shell`."
- "Understand the structure of a geometry description file."
keypoints:
- "Compact XML files are used to store parameters which are used by compiled plugins."
---
We start the discussion of the geometry definition with an overview of the locations of geometry files, and what is included in these files.

## Location of standard geometries in `eic-shell`

Several standard geometry versions are included in `eic-shell` under the `/opt/detector/` location. This includes (currently) at least the following:
```console
$ ls -1 /opt/detector/
calibrations
epic-22.12.0
epic-23.01.0
epic-23.03.0
epic-23.03.0-alt-img
epic-23.05.0
epic-23.05.1
epic-23.05.2
epic-23.06.0
epic-alternative-geometries-img-ecal
epic-main
epic-nightly
fieldmaps
lib
setup.sh
share
```

> Note: `ls -1` lists the files with 1 file per line, i.e. in 1 column.
{: .callout}

The `epic-nightly` directory contains the current 'nightly build' of the ePIC geometry.
```console
$ ls -1 /opt/detector/epic-nightly/
lib
setup.sh
share
```

You can load a geometry by 'sourcing' the `setup.sh` file. Sourcing the top-level `setup.sh` file will load the default geometry (which is currently `epic-nightly`):
```console
$ source /opt/detector/setup.sh
```
or
```console
$ source /opt/detector/epic-nightly/setup.sh
```
Both commands should have the same effect:
- your prompt should have changed to `nightly> ` to indicate the geometry that is loaded,
- your shell environment will have the necessary variables loaded to work with the `epic-nightly` geometry.

You can verify the latter by investigating the values of several environment variables:
- `DETECTOR` is the name of the detector geometry that is loaded (`epic`),
- `DETECTOR_VERSION` is the version (i.e. GitHub branch or tag) that is loaded (`main`),
- `DETECTOR_CONFIG` is the detector configuration to use (i.e. whether to include MRICH or PFRICH, SciGlass or imaging ECAL),
- `DETECTOR_PATH` is the location that points to the geometry resources (`/opt/detector/epic-nightly/share/epic`).

> Note: For older geometries, this will have loaded several beamline environment variables, with similar names and meaning:
> - `BEAMLINE_CONFIG` (`ip6`),
> - `BEAMLINE_CONFIG_VERSION` (`master`),
> - `BEAMLINE_PATH` (`/opt/detector/epic-nightly/share/epic/`).
{: .callout}

> Exercise:
> - Load the standard ePIC geometry and verify (with e.g. `echo $DETECTOR`) that the environment variables are set.
> - Load another geometry and verify that the environment variables are indeed different.
{: .challenge}

## What is stored at those locations?

We will now take a look in the directory pointed to with the environment variable `$DETECTOR_PATH`, the location of the geometry resources:
```console
$ ls $DETECTOR_PATH
calibrations     epic_5x100.xml              epic_brycecanyon_no_pfrich.xml    epic_dirc_only.xml                       epic_hcal_gdml.xml        epic_ip6_extended.xml        epic_pfrich_gdml_only.xml  epic_tof_only.xml
compact          epic_5x41.xml               epic_brycecanyon_pfrich_gdml.xml  epic_drich_only.xml                      epic_imaging.xml          epic_ip6.xml                 epic_pfrich_only.xml       epic_tracking_only.xml
epic_10x100.xml  epic_arches_mrich_gdml.xml  epic_brycecanyon.xml              epic_forward_detectors_with_inserts.xml  epic_inner_detector.xml   epic_lfhcal_only.xml         epic_pid_only.xml          epic_vertex_only.xml
epic_10x275.xml  epic_arches.xml             epic_calorimeters.xml             epic_forward_detectors.xml               epic_ip6_arches.xml       epic_lfhcal_with_insert.xml  epic_sciglass_only.xml     epic.xml
epic_18x275.xml  epic_bhcal.xml              epic_central.xml                  epic_full.xml                            epic_ip6_brycecanyon.xml  epic_mrich_only.xml          epic_tof_endcap_only.xml   fieldmaps
```
You will see many xml files, all of which are entry points to the geometry in certain configurations. For example, `epic_drich_only.xml` includes the geometry that has only the dual RICH or dRICH (and, possibly, the central beamline). The default configuration, `epic.xml`, is typically the configuration you will want to use. That is the value that `DETECTOR_CONFIG` will be set to by default.

> Note: In older ePIC geometries you may also see some files that still refer to ECCE; these are kept for backward compatibility and were removed in more recent geometry release.
{: .callout}

Let's take a look in *the default entry point file*, pointed at by the `DETECTOR_CONFIG` environment variable. This is the file `epic.xml`:
```console
$ less $DETECTOR_PATH/$DETECTOR_CONFIG.xml
```
(Note: Use `q` to exit `less`, or use any editor you prefer.)

The xml file includes several blocks, but look in particular for the following lines:
- `<include ref="${DETECTOR_PATH}/compact/definitions.xml"/>`: This line includes the overall detector parametrization file (think of this as a detector parameter table similar to what the EIC Menagerie provides).
- `<include ref="${DETECTOR_PATH}/compact/tracking/vertex_barrel.xml"/>`: This line includes one of the tracker subsystems; there are other include lines that load other tracking subsystems, or even other types of subsystems.
- `<include ref="${BEAMLINE_PATH}/compact/far_forward.xml"/>`: This line includes the far forward subsystems.

Several of these included files (e.g. `far_forward.xml`) include even more files (e.g. `far_forward/ZDC.xml`).

Let's now take a look at *a particular detector subsystem end point file* (which does not include any more files), namely `tracking/vertex_barrel.xml`.
```console
$ less $DETECTOR_PATH/compact/tracking/vertex_barrel.xml
```
You will notice that the detector is described by parameters in a `define` block, such as (abridged):
```xml
  <define>
    <constant name="VertexBarrelMod_length"             value="VertexBarrel_length"/>
    <constant name="VertexBarrelMod_rmin"               value="VertexBarrel_rmin"/>

    <constant name="SiVertexSensor_thickness"           value="40*um"/>
  </define>
```
which can either use another parameter defined previously in the included files, or which can be defined in this file itself. A best practices is to define detailed parameters of each subsystem in the end point file, but to defer to the central definitions in the `definitions.xml` for quantities such as the overal size and location of the subsystem, or interfaces with other subsystems.

The parameters are then used in the `detector` block to define the detector itself (much abridged):
```xml
  <detectors>
    <detector
      id="VertexBarrel_0_ID"
      name="VertexBarrel"
      type="epic_VertexBarrel"
      readout="VertexBarrelHits"
      insideTrackingVolume="true">
      <dimensions
        rmin="VertexBarrelLayer1_rmin"
        rmax="VertexBarrelLayer3_rmax"
        length="VertexBarrelEnvelope_length" />
    </detector>
  </detectors>
```

The parametrization of the entire detector, down to the subsystems, is defined in these xml files. But where are the volumes created? The key here is the `type` field, which points to the detector type *plugin* that interprets the parametrization (here the type is `epic_VertexBarrel`). A well-written detector plugin can support many different detector configurations and parametrizations without the need to ever touch a line of code.

> Exercise:
> - Identify which subsystem or detector you are interested in.
> - Take a look in the `epic.xml` file and locate where this detector is included.
> - Locate the end point file that defines the parameters that describe this file.
> - Identify the detector plugin that is used for this detector.
{: .challenge}
