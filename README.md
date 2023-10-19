Slurm
-----

Goals: 

* Familiarize yourself with batch scripts
* Understand how to modify and submit an existing batch script
* Understand how to run a job with slurm
* Explore job layout on the node.
* Understand how to query the queue for your jobs status and read the results


Frontier uses SchedMD's Slurm Workload Manager for scheduling and managing jobs. Slurm maintains similar functionality to other schedulers such as IBM's LSF, but provides unique control of Frontier's resources through custom commands and options specific to Slurm. A few important commands can be found in the conversion table below, but please visit SchedMD's `Rosetta Stone of Workload Managers <https://slurm.schedmd.com/rosetta.pdf>`__ for a more complete conversion reference. 

Slurm documentation for each command is available via the ``man`` utility, and on the web at `<https://slurm.schedmd.com/man_index.html>`__. Additional documentation is available at `<https://slurm.schedmd.com/documentation.html>`__.

Some common Slurm commands are summarized in the table below. More complete examples are given in the Monitoring and Modifying Batch Jobs section of this guide.


| Command      | Action/Task                                    | LSF Equivalent                     |
|:-------:     | :----------                                    | :-------------                     |
| ``squeue``   | Show the current queue                         | ``bjobs``                          |
| ``sbatch``   | Submit a batch script                          | ``bsub``                           |
| ``salloc``   | Submit an interactive job                      | ``bsub -Is $SHELL``                |
| ``srun``     | Launch a parallel job                          | ``jsrun``                          |
| ``sacct``    | View accounting information for jobs/job steps | ``bacct``                          |
| ``scancel``  | Cancel a job or job step                       | ``bkill``                          |
| ``scontrol`` | View or modify job configuration.              | ``bstop``, ``bresume``, ``bmod``   |

This challegne will guide your through using ``sbatch``,  ``srun`` and ``squeue``. We will be submitting the jobs via batch scripts that allow us to take advantage of the scheduer to manage the workload since we all need to share a limit nubmer of nodes for the crash course.  Let's start by first setting up our test code and then learning how to run it with a batch script.  

Compiling the test code
-----------------------
We will use a code called `hello_mpi_omp` writen by Tom Papatheodore as our test example. This code's output will display where each process runs on the compute node in its output. To start we will just see how to run it with a batch script. 

To begin, make sure you are in the directory for the challenge by doing 

```
cd ~/hands-on-with-Frontier-/challenges/Srun

```

Do ``ls`` to verify that you see `hello_mpi_omp.c`, `makefile` and `submit.sl’ listed. 

If you like, you may look at the code by doing 
```
vi hello_mpi-omp.c

```
But you may not understand what it is doing until you have done the MPI and OpenMP challenges. 

To close the file from vi do "esc". ":q". 


We will use a makefile, which is a file that specifies how to compile the program. If you are curious you may view the makefile by doing `` vi Makefile``, but you do not need to understand that file to achieve the goals of this exercise. 

We will use the default programming environment on Frontier to compile this code, which means using the Cray programming environment and Cray-mpich for MPI. On Frontier these are set up when you login. If running this tutorial on other machines, you would need to use their documentation to learn how to setup a programming environment to support MPI and OpenMP.

To use the makefile to compile the code on Frontier, do:
```
make
```
If all goes right, you will see that this produces an executable file called `hello_mpi_omp`. You may use `ls` to verify that the file is present. If you don't see it retrace your steps to find what you missed. You may also ask an instructor. 

Now that we have an executable to run, lets use a batch script to run it! 

Batch Scripts
-------------

The most common way to interact with the batch system is via batch scripts. A batch script is simply a shell script with added directives to request various resoruces from or provide certain information to the scheduling system.  Aside from these directives, the batch script is simply the series of commands needed to set up and run your job.

To submit a batch script, use the command ``sbatch submit.sl``

Consider the following batch script:

```
#!/bin/bash
#SBATCH -A TRN###
#SBATCH -J srun_<enter your name here>
#SBATCH -o %x-%j.out
#SBATCH -t 10:00
#SBATCH -p batch
#SBATCH -N 1

# number of OpenMP threads
export OMP_NUM_THREADS=1

# jsrun command to modify 
srun -N 1 -n 1 -c 1 ./hello_mpi_omp

```

In the script, Slurm directives are preceded by ``#SBATCH``, making them appear as comments to the shell. Slurm looks for these directives through the first non-comment, non-whitespace line. Options after that will be ignored by Slurm (and the shell).


| Line | Description                                                                                     |
| :--: | :----------                                                                                     |
|    1 | Shell interpreter line                                                                          |
|    2 | OLCF project to charge                                                                          |
|    3 | Job name                                                                                        |
|    4 | Job standard output file (``%x`` will be replaced with the job name and ``%j`` with the Job ID) |
|    5 | Walltime requested (in ``MM:SS`` format). See the table below for other formats.                |
|    6 | Partition (queue) to use                                                                        |
|    7 | Number of compute nodes requested                                                               |
|    8 | Blank line                                                                                      |
|    9 | Comment                                                                                         |
|   10 | Sets the number of OpenMP threads                                                                |
|   11 | Blank line                                                                                      |
|   12 | Comment                                                                                         |
|   13 | Run the job ( add layout details )                                                              |


We will modify a simple batch script now to give you practice. 
Open the batch script with vi (or your favored text editor). To use Vi do the following:

```
vi submit.sl

```
Use "esc", ":i" to put vi in insert mode, use the explainations in the example above to make the following modififcations to the batch script. 
1. Change the project to the project ID for this tutorial.
2. Customize your job's name
3. Change the time from 10 minutes to 8 minutes.

Submit the batch script to run by doing 

```
sbatch submit.sl

```

To see what state your job is in use 
```
squeue -u <your_user_id>

````
The sections below will help you understand how to read the output that is given. 

Job States
----------

A job will transition through several states during its lifetime. Common ones include:


| Code | State      | Description                                                                    |
| :--- |:-----      | :----------                                                                    | 
| CA    | Canceled   | The job was canceled (could've been by the user or an administrator)          |
| CD    | Completed  | The job completed successfully (exit code 0)                                  |
| CG    | Completing | The job is in the process of completing (some processes may still be running) |
| PD    | Pending    | The job is waiting for resources to be allocated                              |
| R     | Running    | The job is currently running                                                  |



Did you see that your job was queued and why it was not running yet? It’s possible that your job would have run before you could see your query, so there might not be an entry for it in the queue. 
The other way to check if it ran, is to see if there is an output file in you directory.


When your job is done `ls` will show the output file.  

Srun Results for Example 1
--------------------------
Let's look at the output of your job and use that example to begin to understand what the srun commands do to organize work on the compute node. To interpret the results you need to understand some basics of the node hardware, and the parallel programming models MPI and OpenMP.   

The compute nodes are composed of hardware cores (CPUs) that have several hardware threads each. Most modern HPC nodes also have GPUs, but we will not focus on those yet. 

To organize work in parallel, we use MPI tasks and OpenMP threads. These are specified by the program and each does a specific task as set by the programmer. In the case of our hello_mpi_omp program, each MPI task gets the name of the node running the code and organizes its associated OpenMP processes to store their process IDs and the ID of the hardware thread from the cpu core that each ran on in a varible and then write that information to the output file. In real HPC applications, the MPI tasks and OpenMP processes are used to organize the parallel solving of math, such as matrix algebra, or to distribute data. 

The output of hello_mpi_omp will look like this:

MPI taskID, OpenMP process ID, Hardware Thead ID, Node ID 
```
MPI 000 - OMP 000 - HWT 001 - Node crusher035
```
The example's srun was setup fpr  1 node (-N 1), 1 MPI task (-n 1), with 1 MPI task per core and 0 OpenMP processes. For Srun that looks like: 
```
srun -N 1 -n 1 -c 1 ./hello_mpi_omp
```
The output from hello_mpi_omp
``` 
MPI 000 - OMP 000 - HWT 001 - Node frontier035
```
This means MPI task 000 and OpenMP process 000 ran on hardware thread 001 on node 35. 

To see if you got the same result from your job, do:

```
ls
```

You will see something like this:

```
hello_mpi_omp    hello_mpi_omp.o    submit.sl
hello_mpi_omp.c  Makefile         *srun_YOUR-JOB-NAME-<your-job-ID-number>.out*
```
do 

```
vi   srun_YOUR-JOB-NAME-<your-job-ID-number>

```
where you replace `YOUR-JOB-NAME` and `your-job-ID-number` with the name and number from the file you listed.  

If you don't see output that looks like, 
```
MPI 000 - OMP 000 - HWT 001 - Node frontier035
```
Retrace your steps or ask for help from the TAs. 



OK now let’s add more OpenMP processes:









Use
Common Slurm Options
--------------------

The table below summarizes options for submitted jobs. Unless otherwise noted, they can be used for either batch scripts or interactive batch jobs. For scripts, they can be added on the ``sbatch`` command line or as a ``#BSUB`` directive in the batch script. (If they're specified in both places, the command line takes precedence.) This is only a subset of all available options. Check the `Slurm Man Pages <https://slurm.schedmd.com/man_index.html>`__ for a more complete list.


| Option                 | Example Usage                              | Description                                                                          |
|:------                 |:--------------                             |:-----------                                                                          |          
| ``-A``                 | ``#SBATCH -A ABC123``                      | Specifies the project to which the job should be charged                             |
| ``-N``                 | ``#SBATCH -N 1024``                        | Request 1024 nodes for the job                                                       |
| ``-t``                 | ``#SBATCH -t 4:00:00``                     | Request a walltime of 4 hours. <br> Walltime requests can be specified as minutes, hours:minutes, hours:minuts:seconds <br> days-hours, days-hours:minutes, or days-hours:minutes:seconds |                                  
| ``--threads-per-core`` | ``#SBATCH --threads-per-core=2``           | Number of active hardware threads per core. Can be 1 or 2 (1 is default)<br> **Must** be used if using ``--threads-per-core=2`` in your ``srun`` command.|
| ``-J``                 | ``#SBATCH -J MyJob123``                    | Specify the job name (this will show up in queue listings)                           |
| ``-o``                 | ``#SBATCH -o jobout.%j``                   | File where job STDOUT will be directed (%j will be replaced with the job ID).        |
|                        |                                            | If no `-e` option is specified, job STDERR will be placed in this file, too.         |
| ``-e``                 | ``#SBATCH -e joberr.%j``                   | File where job STDERR will be directed (%j will be replaced with the job ID).        |
|                        |                                            | If no `-o` option is specified, job STDOUT will be placed in this file, too.         |
| ``--mail-type``        | ``#SBATCH --mail-type=END``                | Send email for certain job actions. Can be a comma-separated list. Actions include <br>  BEGIN, END, FAIL, REQUEUE, INVALID_DEPEND, STAGE_OUT, ALL, and more.|
| ``--mail-user``        | ``#SBATCH --mail-user=user@somewhere.com`` | Email address to be used for notifications.                                          |
| ``--reservation``      | ``#SBATCH --reservation=MyReservation.1``  | Instructs Slurm to run a job on nodes that are part of the specified reservation.    |


Many other states and job reason codes exist. For a more complete description, see the ``squeue`` man page (either on the system or online).

