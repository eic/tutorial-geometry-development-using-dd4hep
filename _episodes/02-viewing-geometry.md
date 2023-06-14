---
title: "Viewing the geometry"
teaching: 5
exercises: 5
questions:
- "How can we view the geometry?"
objectives:
- "Understand how to export the geometry with `dd_web_display`."
- "Know of multiple way in which to open ROOT TGeo geometries."
keypoints:
- "The geometry, exported to the ROOT TGeo file, can be viewed with ROOT."
---
Before we move to discussion of the detector plugins in the next part, let's discuss visualization on your local system.

The geometry included in `eic-shell` can be viewed with the ROOT geometry browser. However, we first need to export it from the built-in DD4hep format to the ROOT TGeo format, with a small utility program `dd_web_display`. To do this, we will need to ensure we are in a directory where we have write access, such as the directory `~/eic/`.
```console
$ cd ~/eic/
$ dd_web_display --export $DETECTOR_PATH/$DETECTOR_CONFIG.xml
```

> Note: If you are not inside `eic-shell`, you may need to be connected to the internet as your run this command since a magnetic fieldmap will need to be downloaded.
{: .callout}

The `dd_web_display` utility will create a ROOT file in the current directory that can be opened with the geometry viewer online at [https://eic.phy.anl.gov/geoviewer/](https://eic.phy.anl.gov/geoviewer/) or [https://root.cern/js/latest](https://root.cern/js/latest), or a local ROOT installation.

For local ROOT installations, the following commands may be helpful:
```
TGeoManager::Import("detector_geometry.root");
gGeoManager->GetTopVolume()->Draw("ogl") 
```

The geometry viewer has to make decisions on what to draw in order to keep the number of facets small enough. This means that detectors with a large number of repeated components may not be drawn, or other detectors may not be drawn when those detectors are drawn. For this reason, we also have the subsystem-specific entry points xml files. For visualization of specific subsystems, these files are recommended.

> Exercise:
> - Export a different detector configuration from the default `epic.xml`, and export this to ROOT TGeo format.
> - Open the exported ROOT file in the geometry viewer at [https://eic.phy.anl.gov/geoviewer/](https://eic.phy.anl.gov/geoviewer/) or [https://root.cern/js/latest](https://root.cern/js/latest).
{: .challenge}
