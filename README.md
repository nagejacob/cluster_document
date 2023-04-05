# Slurm User Guide for IIL@HIT
## Overview
Currently, our cluster has 1 master node, 5 GTX1080Ti nodes and 2 RTX2080Ti nodes, and may merge 5 A6000 nodes in the next few days. The GPU limit is set to 2 or 4 GPUs for graduate students and PhD candidates, respectively. It contains 8x16T HDDs which are mounted on every node with NFS, so the home directory of each user is implicitly synchronized between the nodes. The disk limit is set to 1TB for each user. As we do not use fault-tolerant file system, there is no guarantee for data persistence.

## Super Quick Start

#### 1. SSH Connection
Please contact 朱航延(QQ 906213291) to create your user account, as well as config SSH keys. For safety reason, we use randomly generate the password when creating user account and discard the password immediately. Authenticating with SSH keys is the only way to connect to the cluster.

At this time, the default SSH port of the cluster is banned by Network Information Center (NIC), so we sort to use port 1022. In this tutorial, we take user name `lijunyi` as example, you could connect to the cluster through SSH as follows:
```
ssh -p1022 lijunyi@10.160.22.41
```

#### 2. Environment
We pre-install multi-user anaconda, and enable it for each user by default. However, as there are dozens of users, directly creating conda environment at `$PREFIX/envs` may mess up. We recommend users to create and activate conda environment in there home directories:
```
mdkir ~/anaconda
conda create -p ~/anaconda/env_name python=3.10
conda activate -p ~/anaconda/env_name
```
One may install his own anaconda and config the conda and pip mirrors to mirrors.hit.edu.cn as well.

#### 3. Interactive Job
One can alloc an interactive job and start a console for create environments, debug, etc. We restrict the corresponding CPU and memory resources for each GPU, e.g., 4 CPUs and 30G memory for each GTX1080Ti GPU.
To prevent occupy GPUs with interactive job for a long time, we limit the duration time to 4 hours, and one can re-alloc new interactive job when time exceeds.
Examples are as follows:
```
salloc --gres=gpu:1080:1
srun --pty /bin/bash
```
where '1080' is the GPU type in ['1080', '2080'], '1' is the GPU count. As `srun` only starts one terminal, you may use `tmux` in it to create multiple terminals for run codes and show statistics, etc.

#### 4. Batch Job
Batch job has no time limit, and may be used for training models for several days. One may first create a shell script named `job.slurm` that specifies job details:
```
#!/bin/bash
#SBATCH --job-name=default
#SBATCH --gres=gpu:1080:1
#SBATCH --output=%j.out
#SBATCH --error=%j.err

conda activate -p ~/anaconda/env_name
cd path_to_code
python train.py
```
Then submit the job:
```
sbatch job.slurm
```

#### 5. Other Commands
show running jobs,
```
squeue -u `whoami`
```
cancel running jobs by JOBID,
```
scancel JOBID
```
show user resource limit,
```
sacctmgr show assoc format=user,grptres%80 | grep `whoami`
```
show cluster GPU usage,
```
slurm_gpustat --verbose
```
