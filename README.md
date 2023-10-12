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


We will use a makefile, which is a file that specifies how to compile the program. If you are curious you may view the makefile by doing `` vi makefile``, but you do not need to understand that file to achieve the goals of this exercise. 

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
    #SBATCH -A ABC123
    #SBATCH -J RunSim123
    #SBATCH -o %x-%j.out
    #SBATCH -t 10:00
    #SBATCH -p batch
    #SBATCH -N 1

    cd $MEMBERWORK/abc123/Run.456
    cp $PROJWORK/abc123/RunData/Input.456 ./Input.456
    srun ...
    cp my_output_file $PROJWORK/abc123/RunData/Output.456

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
|    9 | Change into the run directory                                                                   |
|   10 | Copy the input file into place                                                                  |
|   11 | Run the job ( add layout details )                                                              |
|   12 | Copy the output file to an appropriate location.                                                |

We will modify a simple batch script now to give you practice. Note in the simple script we will be running from, and writing to, the same directory that we submit the batch script from, so we will not need to have lines 9, 10 and 12, but those will be useful for you if you ever need to target your codes' reads and writes to the parallel filesystem’s directories in the future.

Open the batch script with Vi (or your favored text editor). To use Vi do the following:

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



Job Reason Codes
----------------

In addition to state codes, jobs that are pending will have a "reason code" to explain why the job is pending. Completed jobs will have a reason describing how the job ended. Some codes you might see include:


| Reason            | Meaning                                                                                                       |
|:------            | :------                                                                                                       |
| Dependency        | Job has dependencies that have not been met                                                                   |
| JobHeldUser       | Job is held at user's request                                                                                 |
| JobHeldAdmin      | Job is held at system administrator's request                                                                 |
| Priority          | Other jobs with higher priority exist for the partition/reservation                                           |
| Reservation       | The job is waiting for its reservation to become available                                                    |
| AssocMaxJobsLimit | The job is being held because the user/project has hit the limit on running jobs                              |
| ReqNodeNotAvail   | The requested a particular node, but it's currently unavailable (it's in use, reserved, down, draining, etc.) |
| JobLaunchFailure  | Job failed to launch (could due to system problems, invalid program name, etc.)                               |
| NonZeroExitCode   | The job exited with some code other than 0                                                                    |


Did you see that your job was queued and why it was not running yet? Its possible, that your job would have run before you could see your query 

When your job is done `ls` will show an output file in your working directory. 











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

