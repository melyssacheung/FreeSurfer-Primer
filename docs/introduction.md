---
title: "FreeSurfer-Primer|Introduction"
layout: default
---
## Background
### Cortical Surface Reconstruction 
FS initially emerged from efforts to constrain the [inverse problem for EEG/MEG source localization](http://www.scholarpedia.org/article/Source_localization) using anatomically accurate cortical surface models (\cite{dale_improved_1993}).  Dale & Sereno's major innovation was to use the gray/white boundary instead of the pial surface to reconstruct the cortex -- effectively curtailing the challenge of conforming a continuous spherical surface to a highly irregular and non-continuous cortical gyri.  Unfortunately, these early attempts did not produce accurate surface models and required extensive user intervention to correct topological defects. The accuracy of surface deformation techniques is constrained by three considerations:

**Iso-Intensity Surfaces** -- the assumption that the MRI intensity of gray/white and pial surfaces are constant. However, this assumption is not accurate. Cortical tissue varies in microanatomy and thus the signal intensity is quite variable across cortex.  For example, myelin-rich regions will appear much brighter than myelin-poor regions.

**Topological Constraints** -- surfaces must be constrained to not self intersect.  This problem is made exceptionally difficult since the cortex is tightly packed making the demarcation between adjacent sulcal banks a hard computational problem for planar Eulerian methods.

**Curvature minimization** -- smooth surfaces are generated through curvature minimization, however cortex contains many regions with high curvature.

Fischl & Dale addressed nonuniform iso-intensity by adaptively determining MR intensity thresholds for a boundary at each point in an image. Topological constraints can be ignored by avoiding curvature minimization and using a second-order polynomial patch instead of a plane surface. Limitations of curvature minimization algorithms have been avoided via application of fast triangle-triangle intersection borrowed from the computer graphics literature (\cite{fischl_measuring_2000}).  

These workarounds have been proven to produce accurate and sensitive (up to 0.25 mm) measurements of cortical thickness that withstood considerable variation in sequence parameters, acquisition artifacts and tissue variability \cite{fischl_automated_2001}.  Many groups have now validated Fischl & Dale's reconstruction algorithm with histological and manual measures across many scanner platforms, in the presence of subject motion, different acquisition parameters, correlations with behavioral measures and analysis types \cite{dickerson_detection_2008,  han_reliability_2006, rosas_regional_2002, ReutEtal14, vonEconomo}.  

### Whole Brain Segmentation 
Once the cortical surface reconstruction approach had been tackled, Fischl et al. turned their attention to whole brain segmentation. The major difficulty in performing algorithmic volumetric segmentation of the gray matter is the high variability of image intensity across brain structures. A Bayesian approach was adopted to solve this problem using a different model to compute the image likelihood at each position in space. A non-stationary, anistrophic Markov Random Field model was used to define the prior on structure identity given relative spatial location to other macro-structures. The technique was validated using a training set of manually labeled images, achieving comparable performance to extensively trained personnel and exhibited excellent performance with pathological samples \cite{fischl_whole_2002, fischl_sequence-independent_2004}.

### Other Contributions
Many other groups have contributed improvements and additional modules to the FS software suite: 
 
* Tractography \cite{yendiki_automated_2011}

* Alignment of Cortical Folding Patterns and Subcortical/Ventricular Structures
\cite{postelnicu_combined_2009}

* Analysis of Cortical Curvature Development \cite{pienaar_methodology_2008}

* Skull Stripping \cite{segonne_hybrid_2004}

* Statistical Models for Longitudinal Analyses \cite{reuter_within-subject_2012}

* Gyral Based Cortical Parcellation \cite{desikan_automated_2006}

* Insensitivity to Pulse Sequences \cite{han_reliability_2006}

* High Resolution Segmentation of Hippocampal Subfields \cite{van_leemput_automated_2009}

* Probabilistic Label Segmentation \cite{sabuncu_generative_2010}


Thanks to the efforts of those that validated and built-upon the software, cognitive neuroscientists and research clinicians can accurately measure a variety of morphometrics in relation to brain function. Now that you are stoked after perusing this exciting scientific historical backdrop, continue on to learn how to install FS and run your first cortical reconstruction.