# blender-hpc-rendering
Automated rendering of Blender using SLURM on HPC
This project contains scripts and instructions for rendering Blender animations using a high-performance computing (HPC) cluster with SLURM.

## Features
- SLURM job script to parallelize frame rendering
- Output and log organization using subfolders
- Final video concatenation with `ffmpeg`

## Requirements
- Blender (loaded via `module load blender`)
- Access to an HPC cluster with SLURM
- `ffmpeg` (for combining frames into a video)

### 1. Set Output Directory in Blender
Set the output path in Blender to the directory you will use on the HPC system

### 2. Create a Bash Script for SLURM
Create a file, e.g., 'render.sh' with the following contents:

```
$!/bin/bash
$SBATCH --cpus-per-task=10
$SBATCH --partition=icomputeq
$SBATCH --array=1-<insert number of frames>:10

module load blender

FRAMES_PER_JOB=10
START_FRAME=$(( (SLURM_ARRAY_TASK_ID - 1) * FRAMES_PER_JOB + 1 ))
END_FRAME=$(( START_FRAME + FRAMES_PER_JOB - 1 ))
blender -b <insert blender filename> -noaudio -E CYCLES -s $START_FRAME  -e $END_FRAME -a 
```

### 3. Submit the Job
Use command *sbatch <insert (bash script.sh)>* to submit the file for rendering
```
sbatch render.sh
```

### 4. Monitor Job Status
To check for status use *sacct*
```
sacct
```
### 5. Concatenate Results with ffmpeg
After rendering is done, combine .mp4 segments:
```
ls *.mp4 | awk '{printf "file '\''%s'\''\n", $1}' > mylist.txt

module load spack

module load ffmpeg

ffmpeg -f concat -safe 0 -i <insert output name>.txt -c copy final_output.mp4
```

*`mp4` changes depending on the type of output file you need. MKV can also work, for this case our outputs are -a for animation so it is a video format we need.*

# HPC Rendering Using Sub Folders

Using separate output folders helps keep your project organized with the output files in one folder, and the logs of the project in another. Without doing so, there is a cluttered folder full of outputs and logs. This makes it harder to find any specific file if its a large animation. 

```
#!/bin/bash
#SBATCH --cpus-per-task=10
#SBATCH --partition=icomputeq
#SBATCH --array=1-<insert number of frames>:10
#SBATCH -o ./<insert folder name>/output.%a.out # STDOUT

module load blender

START_FRAME=$(($SLURM_ARRAY_TASK_ID-1))
END_FRAME=$(($SLURM_ARRAY_TASK_ID-1+10))
blender -b <insert blender filename> -noaudio -E CYCLES -s $START_FRAME -e $END_FRAME -o //<insert rendered frames folder name>/ -a 
```

See Step 5 in the previous section

