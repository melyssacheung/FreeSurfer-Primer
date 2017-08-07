---
title: "FreeSurfer-Primer|Introduction"
layout: default
---
## Installation & System Configuration
The following installation instructions are tailored to OS X/Linux users with some familiarity of command line usage.  FS is not compatible with Windows. You will need to install a Linux Virtual Machine (VM) if you would like to run FS on a Windows machine.

1. [Download FreeSurfer](ftp://surfer.nmr.mgh.harvard.edu/pub/dist/freesurfer/6.0/freesurfer-Darwin-lion-stable-pub-v5.3.0.dmg}{Download FreeSurfer)

2. Add the FS start-up routine to a file that is read every time you log into a terminal session.  To do this open a terminal, move into your home directory and open ```.bash_profile``` in a text editing program such as sublime, emacs, vi or nano
```bash
cd ~
emacs .bash_profile
```
Edit your ```.bash_profile``` to include the commands below.  Make sure the [she-bang](https://en.wikipedia.org/wiki/Shebang_(Unix)) (```#!/bin/bash```) is included at the top if you are creating the file from scratch.  This tells your terminal what program is used to run these commands.  
```bash
#!/bin/bash
# FREESURFER SETUP
export FREESURFER_HOME="/Applications/freesurfer"
source $FREESURFER_HOME/SetUpFreeSurfer.sh
export SUBJECTS_DIR="/Applications/freesurfer/subjects"
```
After verifying everything looks ok, type the ```control``` key then ```x```, the ```control``` key again, then ```c``` to exit and save (for emacs). You will be prompted whether you want to save the file before exiting.  Type ```y``` then hit ```enter``` to do so. 
The commands you added to your ```.bash_profile``` will put the FS tools at your disposal each time you open a terminal window.  Note that you may eventually want to change your ```SUBJECTS_DIR``` to point to the path of your own data directory:
```bash
SUBJECTS_DIR="$HOME/FreeSurfer-Primer/mris"
# See what's inside SUBJECTS_DIR
```
The ```ls``` command prints out all of the subjects in your directory. Here we have set the FreeSurfer-Primer training data as the subjects directory and printed the contents below:
```bash
ls -l $SUBJECTS_DIR
total 0
drwxr-xr-x  6 cfmiuser  staff  204 sub-01
drwxr-xr-x  6 cfmiuser  staff  204 sub-02
drwxr-xr-x  6 cfmiuser  staff  204 sub-03
drwxr-xr-x  6 cfmiuser  staff  204 sub-04
drwxr-xr-x  5 cfmiuser  staff  170 sub-05
drwxr-xr-x  6 cfmiuser  staff  204 sub-06
drwxr-xr-x  6 cfmiuser  staff  204 sub-07
drwxr-xr-x  6 cfmiuser  staff  204 sub-08
drwxr-xr-x  6 cfmiuser  staff  204 sub-09
drwxr-xr-x  6 cfmiuser  staff  204 sub-10
```
Each subject has their own folder containing raw data that has been imported using ```recon-all -s $subject -i $rawmri``` (explained below).

3. Next open up the ```.dmg``` file you downloaded in step 1 and go through the installer. Then register on the FS website to get a [license](https://surfer.nmr.mgh.harvard.edu/registration.html).

4. When it is emailed to you, use ```emacs``` to create a license file then copy and paste in the key. You will need an administrator password for this since you must use super-user do:
```bash
sudo emacs /Applications/freesurfer/license.txt
Enter password:
```

5. IMPORTANT: You must read (i.e., source) your ```.bash_profile``` so you can use FS in your current terminal session
```bash
cd $HOME
source .bash_profile
```

6. Run your first test of the installation by typing the following into a terminal
```bash
freeview -v $FREESURFER_HOME/subjects/bert/mri/orig.mgz
```

## Reconstruct them all!

### Quick-Start Tutorial 
The most attractive feature of FS is that it makes cortical surface reconstruction and subcortical segmentation for a single subject as easy as typing in a single command and coming back the next day to check your results.  To get started, you must set up your environment and import your imaging data.

Set the environment by following step #2 above and sourcing your ```.bash_profile```. Once the environment for FS has been set, you are ready to reconstruct your first 3D surfaces from a subject's structural scan.  

### Importing Data
To begin, you must first import the raw image files to your ```SUBJECTS_DIR``` by typing the following into a terminal:
```bash
subject='sub-01'
mridata='~/sub-01.01.nii'
mridatatwo='~/sub-01.02.nii'
recon-all -s $subject -i $mridata -i $mridata2
```

The ```-i``` option converts structural scans in NIFTI format to ```.mgh``` and initializes the directory structure required for running the ```recon-all``` command.  Multiple input images can be specified if more than one run exists for a given subject.  These runs can be averaged together to generate cleaner images.

### What's In Each Subjects Directory?
After importing your data to the FreeSurfer directory structure, take a look in ```SUBJECTS_DIR``` and you should see a folder for the subject with the following directories:
```bash
ls $SUBJECTS_DIR/$subject
label 		# surface labels (i.e, brodmann areas)
mri      	# volumetric files (.mgh/.mgz)
scripts     # status logs, commands, errors kept here
stat     	# data tables
surf  		# surface files (edges & vertices)
tmp 		# keep control.dat here
touch 		# temporary files
trash 		# ovewritten files are moved here
```
Enter the following to view this subject's imported structural scan.
```bash
freeview -v  $SUBJECTS_DIR/$subject/mri/orig/001.mgz
```

Once it is confirmed that the subject has been successfully imported, we can begin the processing stream. Use the ```-all```, ```-3T``` and ```qcache``` options to peform all steps, perform registration and intensity normalization assuming input image was collected on a 3 Tesla scanner and generate smoothed surfaces registered to FreeSurfer's fsaverage space.

```bash
recon-all -s <subjectname> -all -3T -qcache
```

The pre-processing can take up to 24 or as little as 3 hours depending on your machine.  Optimizing this work flow is absolutely crucial when performing a large study with many subjects.

### Speeding up Run Times
#### Built-in Parallelization
As of FreeSurfer v6.0, parallelization has been built into various steps in the pipeline using the OpenMP toolkit. You can enable this by adding the ```-parallel``` flag to your recon-all command. If have newly installed FreeSurfer on macOS Sierra, you will need to make sure you have the most up-to-date version of the ```gcc``` compiler and associated FreeSurfer binaries.
```bash
brew install gcc # if not installed
brew upgrade gcc # if already installed
source $FREESURFER_HOME
sudo -E fs_update
Enter Password:

```
#### Multicore Processing
If you have access to a multicore machine you may be interested in running many invocations  ```recon-all``` at the same time.  A quick and dirty shortcut for parallelizing ```recon-all``` is to run the command in as many terminal prompts as there are processors.  Unfortunately, since the quick and dirty method parallelizes at the highest level, system resources are not efficiently utilized.  

The [GNU parallel tool](https://www.gnu.org/software/parallel/) is an excellent way for efficiently parallelizing any job on a Mac or Linux machine.  You can install this tool using the [homebrew](http://brew.sh) package installer for OS X,
```bash
brew install parallel
```

Once installed, a quick way to get started is to put all of your subjects' anatomical scans in a single folder and typing the following command to import and run in a single call,
```bash
# move to a data directory containing raw files for each subject
cd ~/data
# print list of files in data directory
ls
total 0
drwxr-xr-x  6 cfmiuser  staff  204 sub-01.nii.gz
drwxr-xr-x  6 cfmiuser  staff  204 sub-02.nii.gz
drwxr-xr-x  6 cfmiuser  staff  204 sub-03.nii.gz
drwxr-xr-x  6 cfmiuser  staff  204 sub-04.nii.gz
drwxr-xr-x  5 cfmiuser  staff  170 sub-05.nii.gz
drwxr-xr-x  6 cfmiuser  staff  204 sub-06.nii.gz
drwxr-xr-x  6 cfmiuser  staff  204 sub-07.nii.gz
drwxr-xr-x  6 cfmiuser  staff  204 sub-08.nii.gz
drwxr-xr-x  6 cfmiuser  staff  204 sub-09.nii.gz
drwxr-xr-x  6 cfmiuser  staff  204 sub-10.nii.gz
# now import and run in parallel
find -f *.nii.gz | parallel --bar -k --link \ #line break
recon-all -s {.} -i {} -all -3T -qcache -parallel
```
The ```--bar``` option includes a progress bar with estimated time to completion. The ```-k``` and ```--link``` options tell parallel to execute recon in the order files are piped (|) into the standard input for the program. We use a ```{}``` to refer to each element in the list piped from the ```find -f *.nii.gz``` command. The ```{.}``` removes the extension ```.nii.gz``` from each item in the list files generated with the find command.

#### High Performance Computing on a Grid Engine System
..coming soon..