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