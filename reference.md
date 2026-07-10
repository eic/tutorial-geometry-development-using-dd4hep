---
title: 'Reference'
---

## Glossary

EIC
:   Electron-Ion Collider

DD4hep
:   Detector Description for High Energy Physics, a toolkit that provides a single, central source
    of detector information for simulation and reconstruction.

eic-shell
:   The EIC standard software environment, a singularity/apptainer or docker container with a
    curated selection of software components used for EIC simulations and analysis.

compact file
:   An XML file holding the parameters of a detector (sub)system, interpreted at build time by a
    compiled detector plugin.

detector plugin
:   A C++ function, registered with `DECLARE_DETELEMENT`, that builds the volumes of a detector from
    the parameters supplied in the compact XML file.
