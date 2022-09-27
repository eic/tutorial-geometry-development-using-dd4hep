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
athena-nightly
calibrations
ecce-nightly
epic-nightly
fieldmaps
lib
setup.sh
share
```

> Note: `ls -1` lists the files with 1 file per line, i.e. in 1 column.
{: .callout}

The `epic-nightly` directory contains the current 'nightly build' of the EPIC geometry.
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
Similarly, this will have loaded several beamline environment variables, with similar names and meaning:
- `BEAMLINE_CONFIG` (`ip6`),
- `BEAMLINE_CONFIG_VERSION` (`master`),
- `BEAMLINE_PATH` (`/opt/detector/epic-nightly/share/epic/`).

> Exercise:
> - Load the standard EPIC geometry and verify (with e.g. `echo $DETECTOR`) that the environment variables are set.
> - Load another geometry and verify that the environment variables are indeed different.
{: .challenge}

## What is stored at those locations?

We will now take a look in the directory pointed to with the environment variable `$DETECTOR_PATH`, the location of the geometry resources:
```console
$ ls $DETECTOR_PATH
calibrations           ecce_inner_detector.xml  epic_18x275.xml          epic_pfrich_only.xml
compact                ecce_ip6.xml             epic_calorimeters.xml    epic_pid_only.xml
ecce_18x275.xml        ecce_pfrich_only.xml     epic_central.xml         epic_sciglass.xml
ecce_calorimeters.xml  ecce_pid_only.xml        epic_dirc_only.xml       epic_tof_only.xml
ecce_central.xml       ecce_sciglass.xml        epic_drich_only.xml      epic_tracking_only.xml
ecce_dirc_only.xml     ecce_tof_only.xml        epic_full.xml            epic_vertex_only.xml
ecce_drich_only.xml    ecce_tracking_only.xml   epic_imaging.xml         epic.xml
ecce_full.xml          ecce_vertex_only.xml     epic_inner_detector.xml  fieldmaps
ecce_imaging.xml       ecce.xml                 epic_ip6.xml             ip6
```
You will see many xml files, all of which are entry points to the geometry in certain configurations. For example, `epic_drich_only.xml` includes the geometry that has only the DRICH (and, possibly, the central beamline). The default configuration, `epic.xml`, is typically the configuration you will want to use. That is the value that `DETECTOR_CONFIG` will be set to by default.

> Note: In the EPIC geometry you may also see some files that still refer to ECCE; these are kept for backward compatibility and will disappear in a future geometry release.
{: .callout}

Let's take a look in *the default entry point file*, pointed at by the `DETECTOR_CONFIG` environment variable. This is the file `epic.xml`:
```console
$ less $DETECTOR_PATH/$DETECTOR_CONFIG.xml
```
(Note: Use `q` to exit `less`, or use any editor you prefer.)

The xml file includes several blocks, but look in particular for the following lines:
- `<include ref="${DETECTOR_PATH}/compact/definitions.xml"/>`: This line includes the overall detector parametrization file (think of this as a detector parameter table similar to what the EIC Menagerie provides).
- `<include ref="${DETECTOR_PATH}/compact/tracker.xml"/>`: This line includes the tracker subsystems; there are other include lines that load other subsystems.
- `<include ref="${BEAMLINE_PATH}/ip6/far_forward.xml"/>`: This line includes the far forward subsystems; note the different path prefix and directory `ip6`.

Several of these included files (e.g. `tracker.xml`) include even more files (e.g. `tracker/vertex_barrel.xml`).

Let's now take a look at *a particular detector subsystem end point file* (which does not include any more files), namely `tracker/vertex_barrel.xml`.
```console
$ less $DETECTOR_PATH/compact/tracker/vertex_barrel.xml
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

The parameters are then used in the `detector` block to define the detector itself (abridged):
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
