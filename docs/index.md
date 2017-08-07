---
title: "FreeSurfer-Primer|Home"
layout: default
---
## 3D Reconstruction of MR Brain Images
[Freesurfer](https://surfer.nmr.mgh.harvard.edu/) (FS) is a powerful (and freely available!) software package developed at the Martinos Center for Biomedical Imaging for automatized analyses of brain anatomy.  A non-inclusive list of its capabilities includes:

* Performing morphometric analysis of the brain _(i.e. cortical thickness, curvature, surface area, geodesic folding index, volume)_

* Creating study-specific anatomical templates

* Optimal alignment of cortical folding patterns across individuals

* Processing of functional imaging data using the FSFAST software suite

* Segmenting and aligning individual brain volumes to an anatomical atlas

* Rendering 3D models of the pial surface, white-matter and user-defined masks

Although FS possesses remarkable capabilities that greatly reduce the effort involved in analyzing structural MRI data, there are crucial caveats the user must be aware of. This document is the first in a series of training guides intended to bring a novice user up to speed on using and trouble-shooting FS.  

In this guide, we will describe how to install and configure your system for FS and then walk you through the steps of reconstructing a brain scan from raw data and viewing the final results.  Part II in this series, "Trouble Shooting & Manual Editing" will outline steps for artifact correction and manual intervention at various steps along the FS anatomical analysis workflow. Part III will cover statistical methods for morphometric analyses.

The materials presented here are collected from miscellaneous notes taken from the [FreeSurfer Wiki](https://surfer.nmr.mgh.harvard.edu/fswiki), [list-serv user postings](http://www.mail-archive.com/freesurfer@nmr.mgh.harvard.edu/), [published papers](https://www.zotero.org/freesurfer) and independent experimentation.  If a question comes up that is not addressed in this guide be sure to consult these resources first! If all else fails, ask questions! 