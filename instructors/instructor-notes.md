---
title: "Instructor Notes"
---

This tutorial builds on [Setting Up Your Environment](https://eic.github.io/tutorial-setting-up-environment/):
learners are expected to already have a working `eic-shell` before the session.

## Before the session

- Ask learners to complete the [Setup](../learners/setup.md) page in advance so that `eic-shell` is
  downloaded and working. Downloading the container can take a long time on systems without
  `/cvmfs`.
- The modifying-geometry episode compiles a local clone of the [`epic`](https://github.com/eic/epic)
  repository. The first `cmake --build` can take a while; encourage learners to use `-j` and to
  start the clone/build early.

## Timing

The lesson is roughly 45 minutes of teaching plus an hour of exercises. The compile step in the
final episode and the optional `npsim` runs dominate the wall-clock time, so keep an eye on the
container build finishing before moving on.
