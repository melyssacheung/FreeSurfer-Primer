
## Recon-all: Under the Hood

While you are running your first subject you may notice that the terminal will quickly begin filling up with text describing the processing status, any errors that may have been encountered and details regarding individuals steps taken.  You can get more information about each of these steps and the usage of recon-all by opening a new terminal and typing:
```bash
recon-all -help
```
The help command also provides you with information regarding optional arguments you can flag each ```recon-all``` call with.  Of particular importance are the autorecon flags:
```bash
-autorecon1  :  process stages 1-5
-autorecon2  :  process stages 6-23
-autorecon3  :  process stages 24-34
```

Each of these flags can take additional options for algorithm parameters or to only run a subset of the processing stages. The individual stages and some optional parameters of the FS pre-processing pipeline are summarized below.

### Recon-all consists of 34 processing stages
#### Autorecon1

**Motion Correction and Conform** -- Multiple scans from the same subject are averaged together to minimize motion artifacts.  The input are the volumes found in file(s) ```mri/orig/XXX.mgz```. The output will be the volume ```mri/orig.mgz```. If no runs are found, then it looks for a volume in ```mri/orig```. If that volume is there, then it is used in subsequent processes as if it was the motion corrected volume. If no volume is found, then the process exits with errors.
    
**NU (Non-Uniform intensity normalization)** -- Non-parametric Non-uniform intensity Normalization (N3), corrects for intensity non-uniformity in MR data, making relatively few assumptions about the data. This runs the MINC tool ```nu_correct```. By default, four iterations of ```nu_correct``` are run. The flag ```-nuiterations``` specification of some other number of iterations.
    
**Talairach transform computation** -- The affine matrix required to register the scan to the MNI305 space is computed at this step.  You can/should check the talairach registration using ```tkregister2 --s subjid --fstal```. ```tkregister2``` allows you to compare the original volume against the talairach volume resampled into the original space. Run ```tkregister2 --help``` for more information. This step creates the files ```mri/transform/talairach.auto.xfm``` and ```talairach.xfm```.
    
**Intensity Normalization**  -- Renormalization of intensity. Performs intensity normalization of the ```orig``` volume and places the result in ```mri/T1.mgz```. Attempts to correct for fluctuations in intensity that would otherwise make intensity-based segmentation much more difficult. _Intensities for all voxels are scaled so that the mean intensity of the white matter is 110._ If there are problems with the normalization, users can add control points.
    
**Skull Strip** -- The skull and brainstem are removed from ```mri/T1.mgz``` and stores the result in ```mri/brainmask.auto.mgz``` and ```mri/brainmask.mgz```. Runs the ```mri_watershed``` program. _If the strip fails, users can specify seed points_ (```-wsseed```) _or change the threshold_(```-wsthresh```, ```-wsmore```, ```-wsless```). The ```-autorecon1``` stage ends here.
    
#### Autorecon2
**EM Register (linear volumetric registration)** -- Computes transform to align the ```mri/nu.mgz``` volume to the default GCA atlas found in ```$FREESURFER_HOME/average``` (see ```-gca``` flag for more info). Creates the file ```mri/transforms/talairach.lta```. The ```-autorecon2``` stage starts here.
    
**CA Intensity Normalization** -- Further normalization, based on GCA model. Creates ```mri/norm.mgz```.

**CA Non-linear Volumetric Registration** -- Computes a nonlinear transform to align with GCA atlas. Creates the file ```mri/transform/talairach.m3z```.

**Remove Neck** -- The neck region is removed from the NU-corrected volume ```mri/nu.mgz```. Makes use of transform computed from prior CA Register stage. Creates the file ```mri/nu_noneck.mgz```.

**EM Register, with skull** -- Computes transform to align volume ```mri/nu_noneck.mgz``` with GCA volume possessing the skull. Creates ```mri/transforms/talairach_with_skull.lta```.

**CA Label (Aseg: Volumetric Labeling) and Statistics** -- Labels subcortical structures, based in GCA model. Creates the files ```mri/aseg.auto.mgz``` and ```mri/aseg.mgz```.

**Intensity Normalization 2** (start here for control points) -- Performs a second (major) intensity correction using only the brain volume as the input (so that it has to be done after the skull strip). Intensity normalization works better when the skull has been removed. Creates a new ```brain.mgz volume```. The ```-autorecon2-cp``` stage begins here. If ```-noaseg``` flag is used, then ```aseg.mgz``` is not used by ```mri_normalize```.

**White matter segmentation** -- Attempts to separate white matter from everything else. The input is ```mri/brain.mgz```, and the output is ```mri/wm.mgz```. Uses intensity, neighborhood, and smoothness constraints. _This is the volume that is edited when manually fixing defects_. Calls ```mri_segment, mri_edit_wm_with_aseg, and mri_pretess```. To keep previous edits, run with ```-keepwmedits```. If ```-noaseg``` is used, them ```mri_edit_wm_aseg``` is skipped.

**Edit White Matter with Aseg** -- Any user edits to the segmentation is processed here.

**Fill/Cut Into Brainstem, and Hemispheres** --  This creates the subcortical mass from which the ```orig``` surface is created. The mid brain is cut from the cerebrum, and the hemispheres are cut from each other. The left hemisphere is binarized to 255. The right hemisphere is binarized to 127. The input is ```mri/wm.mgz``` and the output is ```mri/filled.mgz```. Calls ```mri_fill```. If the cut fails, then seed points can be supplied (see ```-cc-crs```, ```-pons-crs```, ```-lh-crs```, ```-rh-crs```). The actual points used for the cutting planes in the corpus callosum and pons can be found in ```scripts/ponscc.cut.log```. The stage ```-autorecon2-wm``` begins here. This is the last stage of volumetric processing. If ```-noaseg``` is used, then ```aseg.mgz``` is not used by ```mri_fill```.

**Tessellation** (begins per-hemisphere operations) -- This is the step where the ```orig``` surface (ie, ```surf/?h.orig.nofix```) is created. The surface is created by covering the filled hemisphere with triangles. Runs ```mri_tessellate```. The places where the points of the triangles meet are called vertices. Creates the file ```surf/?h.orig.nofix```. _Note: the topology fixer will create the surface ?h.orig._

**Smooth 1** -- After tesselation, the ```orig``` surface is very jagged because each triangle is on the edge of a voxel face and so are at right angles to each other. The vertex positions are adjusted slightly here to reduce the angle. This is only necessary for the inflation processes. Creates ```surf/?h.smoothwm(.nofix)```. Calls ```mris_smooth```. Smooth1 is the step just after tessellation, and smooth 2 is the step just after topology fixing.

**Inflate 1** -- Inflation of the ```surf/?h.smoothwm(.nofix)``` surface to ```create surf/?h.inflated```. The inflation attempts to minimize metric distortion so that distances and areas are preserved (ie, the surface is not stretched). In this sense, it is like inflating a paper bag and not a balloon. Inflate 1 is the step just after tessellation, and inflate2 is the step just after topology fixing. Calls ```mris_inflate```. Creates ```?h.inflated, ```?h.sulc```, ```?h.curv```, and ```?h.area```.

**QSphere** -- This is the initial step of automatic topology fixing. It is a quasi-homeomorphic spherical transformation of the inflated surface designed to localize topological defects for the subsequent automatic topology fixer. Calls ```mris_sphere```. Creates ```surf/?h.qsphere.nofix```.

**Automatic Topology Fixer** -- Finds topological defects (ie, holes in a filled hemisphere) using ```surf/?h.qsphere.nofix```, and changes the original surface (```surf/?h.orig.nofix```) to remove the defects. Changes the number of vertices. All the defects will be removed, but the user should check the ```orig``` surface in the volume to make sure that it looks appropriate. Calls ```mris_fix_topology```. Creates ```surf/?h.orig``` (by iteratively fixing ```surf/?h.orig.nofix```).

**White Matter Surfaces** Creates the ```?h.white surface```. The white surface is created by "nudging" the ```orig``` surface so that it closely follows the white-gray intensity gradient as found in the T1 volume.

**Smooth 2** -- Smooth the surfaces again (repeat Smooth 1).

**Cortical Ribbon Mask** -- Creates binary volume masks of the cortical ribbon, ie, each voxel is either a 1 or 0 depending upon whether it falls in the ribbon or not. Saved as ```?h.ribbon.mgz```. Uses mgz regardless of whether the ```-mgz``` option is used. The ```-autorecon2``` stage ends here.

#### Autorecon3
**Inflate 2** -- Inflates the ```orig``` surface into a sphere while minimizing metric distortion. This step is necessary in order to register the surface to the spherical atlas. (also known as the spherical morph). Calls ```mris_sphere```. Creates ```surf/?h.sphere```. The -autorecon3 stage begins here.

**Spherical Registration** -- Registers the ```orig``` surface to the spherical atlas through ```surf/?h.sphere```. The surfaces are first coarsely registered by aligning the large scale folding patterns found in ```?h.sulc``` and then fine tuned using the small-scale patterns as in ```?h.curv```. Calls ```mris_register```. Creates ```surf/?h.sphere.reg```.

**Spherical Registration, Contralateral hemisphere** -- Same as ipsilateral but registers to the contralateral atlas. Creates lh.rh.sphere.reg and rh.lh.sphere.reg.

**Spherical Mapping** -- Maps sphere to atlas labels.

**Map Average Curvature to Subject** -- Resamples the average curvature from the atlas to that of the subject. Allows the user to display activity on the surface of an individual with the folding pattern (ie, anatomy) of a group. Calls ```mrisp_paint```. Creates ```surf/?h.avg_curv```.

**Cortical Parcellation** (Labeling) -- Assigns a neuroanatomical label to each location on the cortical surface. Incorporates both geometric information derived from the cortical model (sulcus and curvature), and neuroanatomical convention. Calls ```mris_ca_label . -cortparc``` creates ```label/?h.aparc.annot``` and ```-cortparc2``` creates ```/label/?h.aparc.a2005s.annot```.

**Cortical Parcellation Statistics** -- Runs ```mris_anatomical_stats``` to create a summary table of cortical parcellation statistics for each structure, including 1. structure name 2. number of vertices 3. total surface area (mm2) 4. total gray matter volume (mm3) 5. average cortical thickness (mm) 6. standard error of cortical thicknessr (mm) 7. integrated rectified mean curvature 8. integrated rectified Gaussian curvature 9. folding index 10. intrinsic curvature index. For ```-parcstats```, the file is saved in ```stats/?h.aparc.stats```. For ```-parcstats2```, the file is saved in ```stats/?h.aparc.a2005s.stats```.

**Pial Surfs** Creates the thickness file (```?h.thickness```) and curvature file (```?h.curv```). The pial surface is created by expanding the white surface so that it closely follows the gray-CSF intensity gradient as found in the T1 volume. Calls ```mris_make_surfaces```.    

**WM/GM Contrast** -- ? No information available on this step


**Cortical Parcellation mapped to ASeg** -- Maps the cortical parcellation to the subcortical segmentation.

**Brodmann and ex vivo EC labels** -- Generate labels to map Brodmann areas and Entorhinal cortex.


## Command Line Cheat Sheet

```cd``` -- change directory.  
```bash
cd ~ #move into home directory
cd . #stay in current directory
cd ../ #move up one directory in the tree
```

```ls``` -- List directory contents.  Useful flags :

- ```-a``` list all directory contents (including hidden files)

- ```-l``` long form, show extra information

- ```-d``` show only directories, useful for getting lists to iterate over

-  ```-G``` output with colors

-  ```-S``` sort files by size

-  ```-s``` display number of file system blocks used by each file in units of 512 bytes
```bash
ls -alGS $SUBJECTS_DIR
```  

```mv``` -- Move or rename a file/directory.  Useful flags :
- ```-f``` overwrite files without asking permission
- ```-i``` return an error if attempting to overwrite existing file
- ```-n``` do not overwrite an existing file 
- ```-v``` verbose output, shows files after they are moved
```bash
mv -nv ~/Projects/ADS/data/Subjects/149959 ~/Projects/ADS/data/BadSubjects/149959
```                    

```cp``` -- Copy contents of source file/directory to a target file/directory.  Useful flags :
- ```-i``` return an error if attempting to overwrite existing file
- ```-n``` do not overwrite an existing file 
- ```-v``` verbose output, shows files after they are moved
- ```-R``` copy a source directory and all of it's subdirectories.  If the source file ends in a /, the contents of the directory are copied rather than the directory itself.
```bash
cp -ivR ~/Projects/ADS/data/Subjects/149959 ~/Projects/ADS/data/Subjects.BackUp/149959
```                                
```rm``` -- Delete a file or directory.  Useful flags :
- ```-f``` overwrite files without asking permission
- ```-i``` return an error if attempting to overwrite existing file
- ```-d``` delete directories
- ```-R/-r``` delete directories and all of it's sub directories (use this instead of -d)
- ```-v``` verbose output, shows files after they are moved

```bash
rm -rv  ~/Projects/ADS/data/BadSubjects/149959
```                                
```sudo``` -- Use before a command to over come permissions.  USE WITH CAUTION.  THIS WILL BREAK YOUR SYSTEM IF YOU ARE NOT CAREFUL TO TRIPLE-CHECK WHAT  YOU ARE MOVING/DELETING.
```bash
sudo rm -rv  ~/Projects/ADS/data/BadSubjects/149959
```            
```cat``` -- concatenate and print files.
This is a very useful command for viewing the contents of any file.
```bash
cat ~/.bash_profile #print your .bash_profile to the screen
cat ~/file1 > ~/file2 #overwrite file 2 with file 1
cat ~/file1 >> ~/file2 #append file 1 to file 2
```                            
```echo``` -- print a variable or a string to the terminal. Useful flags :

- ```-e``` allows string formatting, i.e.) $\backslash$n gives a newline

Simple usage:
```bash
echo "Hello World"
```
Define a variable and print it.  Note that you have to use \$ to refer to the variable after you define it.     
```bash
subjid="149959"
echo $subjid
echo "The subject ID is " $149959
```                        

Advanced usage.  Get a list of subjects by using \texttt{ls} and string formatting to print each subject name on a new line.  You must use parentheses around the output of a command with a leading \$ to store it in a variable.  This is very useful if used properly.
```bash
subjects=$(ls -d ~/Projects/ADS/data/Subjects/*)
echo -e "These are all the subjects in our study with data\n" 
echo -e "\n\t\%s" $subjects    
```

```touch``` -- create a new file
```bash
touch .cool_script
newcommand="freeview -v $SUBJECTS_DIR/149959/mri/T1.mgz"
echo $newcommand >> .cool_scirpt
. .cool_script
```

```date``` -- get the date.
You can use this to measure elapsed time of a script.
```bash
begin=$(date -u "+%s")
sleep 10 #sleep 10s, or run script here
end=$(date -u "+%s")
elapsed=$((END-START)) #double parenthesis to do arithmetic
echo "$(($elapsed %60)) seconds elapsed"
```