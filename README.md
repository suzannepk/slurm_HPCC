Slurm
-----

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



Batch Scripts
-------------

The most common way to interact with the batch system is via batch scripts. A batch script is simply a shell script with added directives to request various resoruces from or provide certain information to the scheduling system.  Aside from these directives, the batch script is simply the series of commands needed to set up and run your job.

To submit a batch script, use the command ``sbatch myjob.sl``

Consider the following batch script:

```

    #!/bin/bash
    #SBATCH -A ABC123
    #SBATCH -J RunSim123
    #SBATCH -o %x-%j.out
    #SBATCH -t 1:00:00
    #SBATCH -p batch
    #SBATCH -N 1024

    cd $MEMBERWORK/abc123/Run.456
    cp $PROJWORK/abc123/RunData/Input.456 ./Input.456
    srun ...
    cp my_output_file $PROJWORK/abc123/RunData/Output.456

```

In the script, Slurm directives are preceded by ``#SBATCH``, making them appear as comments to the shell. Slurm looks for these directives through the first non-comment, non-whitespace line. Options after that will be ignored by Slurm (and the shell).


| Line | Description                                                                                     |
| :--: | :----------                                                                                  |
|    1 | Shell interpreter line                                                                          |
|    2 | OLCF project to charge                                                                          |
|    3 | Job name                                                                                        |
|    4 | Job standard output file (``%x`` will be replaced with the job name and ``%j`` with the Job ID) |
|    5 | Walltime requested (in ``HH:MM:SS`` format). See the table below for other formats.             |
|    6 | Partition (queue) to use                                                                        |
|    7 | Number of compute nodes requested                                                               |
|    8 | Blank line                                                                                      |
|    9 | Change into the run directory                                                                   |
|   10 | Copy the input file into place                                                                  |
|   11 | Run the job ( add layout details )                                                              |
|   12 | Copy the output file to an appropriate location.                                                |


.. _frontier-interactive:

Interactive Jobs
----------------

Most users will find batch jobs an easy way to use the system, as they allow you to "hand off" a job to the scheduler, allowing them to focus on other tasks while their job waits in the queue and eventually runs. Occasionally, it is necessary to run interactively, especially when developing, testing, modifying or debugging a code.

Since all compute resources are managed and scheduled by Slurm, it is not possible to simply log into the system and immediately begin running parallel codes interactively. Rather, you must request the appropriate resources from Slurm and, if necessary, wait for them to become available. This is done through an "interactive batch" job. Interactive batch jobs are submitted with the ``salloc`` command. Resources are requested via the same options that are passed via ``#SBATCH`` in a regular batch script (but without the ``#SBATCH`` prefix). For example, to request an interactive batch job with the same resources that the batch script above requests, you would use ``salloc -A ABC123 -J RunSim123 -t 1:00:00 -p batch -N 1024``. Note there is no option for an output file...you are running interactively, so standard output and standard error will be displayed to the terminal.

.. _common-slurm-options:

Common Slurm Options
--------------------

The table below summarizes options for submitted jobs. Unless otherwise noted, they can be used for either batch scripts or interactive batch jobs. For scripts, they can be added on the ``sbatch`` command line or as a ``#BSUB`` directive in the batch script. (If they're specified in both places, the command line takes precedence.) This is only a subset of all available options. Check the `Slurm Man Pages <https://slurm.schedmd.com/man_index.html>`__ for a more complete list.

+------------------------+--------------------------------------------+--------------------------------------------------------------------------------------+
| Option                 | Example Usage                              | Description                                                                          |
+========================+============================================+======================================================================================+
| ``-A``                 | ``#SBATCH -A ABC123``                      | Specifies the project to which the job should be charged                             |
+------------------------+--------------------------------------------+--------------------------------------------------------------------------------------+
| ``-N``                 | ``#SBATCH -N 1024``                        | Request 1024 nodes for the job                                                       |
+------------------------+--------------------------------------------+--------------------------------------------------------------------------------------+
| ``-t``                 | ``#SBATCH -t 4:00:00``                     | Request a walltime of 4 hours.                                                       |
|                        |                                            | Walltime requests can be specified as minutes, hours:minutes, hours:minuts:seconds   |
|                        |                                            | days-hours, days-hours:minutes, or days-hours:minutes:seconds                        |
+------------------------+--------------------------------------------+--------------------------------------------------------------------------------------+
| ``--threads-per-core`` | ``#SBATCH --threads-per-core=2``           | | Number of active hardware threads per core. Can be 1 or 2 (1 is default)           |
|                        |                                            | | **Must** be used if using ``--threads-per-core=2`` in your ``srun`` command.       |
+------------------------+--------------------------------------------+--------------------------------------------------------------------------------------+
| ``-d``                 | ``#SBATCH -d afterok:12345``               | Specify job dependency (in this example, this job cannot start until job 12345 exits |
|                        |                                            | with an exit code of 0. See the Job Dependency section for more information          |
+------------------------+--------------------------------------------+--------------------------------------------------------------------------------------+
| ``-C``                 | ``#SBATCH -C nvme``                        | Request the burst buffer/NVMe on each node be made available for your job. See       |
|                        |                                            | the Burst Buffers section for more information on using them.                        |
+------------------------+--------------------------------------------+--------------------------------------------------------------------------------------+
| ``-J``                 | ``#SBATCH -J MyJob123``                    | Specify the job name (this will show up in queue listings)                           |
+------------------------+--------------------------------------------+--------------------------------------------------------------------------------------+
| ``-o``                 | ``#SBATCH -o jobout.%j``                   | File where job STDOUT will be directed (%j will be replaced with the job ID).        |
|                        |                                            | If no `-e` option is specified, job STDERR will be placed in this file, too.         |
+------------------------+--------------------------------------------+--------------------------------------------------------------------------------------+
| ``-e``                 | ``#SBATCH -e joberr.%j``                   | File where job STDERR will be directed (%j will be replaced with the job ID).        |
|                        |                                            | If no `-o` option is specified, job STDOUT will be placed in this file, too.         |
+------------------------+--------------------------------------------+--------------------------------------------------------------------------------------+
| ``--mail-type``        | ``#SBATCH --mail-type=END``                | Send email for certain job actions. Can be a comma-separated list. Actions include   |
|                        |                                            | BEGIN, END, FAIL, REQUEUE, INVALID_DEPEND, STAGE_OUT, ALL, and more.                 |
+------------------------+--------------------------------------------+--------------------------------------------------------------------------------------+
| ``--mail-user``        | ``#SBATCH --mail-user=user@somewhere.com`` | Email address to be used for notifications.                                          |
+------------------------+--------------------------------------------+--------------------------------------------------------------------------------------+
| ``--reservation``      | ``#SBATCH --reservation=MyReservation.1``  | Instructs Slurm to run a job on nodes that are part of the specified reservation.    |
+------------------------+--------------------------------------------+--------------------------------------------------------------------------------------+
| ``-S``                 | ``#SBATCH -S 8``                           | Instructs Slurm to reserve a specific number of cores per node (default is 8).       |
|                        |                                            | Reserved cores cannot be used by the application.                                    |
+------------------------+--------------------------------------------+--------------------------------------------------------------------------------------+
| ``--signal``           | ``#SBATCH --signal=USR1@300``              || Send the given signal to a job the specified time (in seconds) seconds before the   |
|                        |                                            | job reaches its walltime. The signal can be by name or by number (i.e. both 10 and   |
|                        |                                            | USR1 would send SIGUSR1).                                                            |
|                        |                                            ||                                                                                     |
|                        |                                            || Signaling a job can be used, for example, to force a job to write a checkpoint just |
|                        |                                            | before Slurm kills the job (note that this option only sends the signal; the user    |
|                        |                                            | must still make sure their job script traps the signal and handles it in the desired |
|                        |                                            | manner).                                                                             |
|                        |                                            ||                                                                                     |
|                        |                                            || When used with ``sbatch``, the signal can be prefixed by "B:"                       |
|                        |                                            | (e.g. ``--signal=B:USR1@300``) to tell Slurm to signal only the batch shell;         |
|                        |                                            | otherwise all processes will be signaled.                                            |
+------------------------+--------------------------------------------+--------------------------------------------------------------------------------------+


Slurm Environment Variables
---------------------------

Slurm reads a number of environment variables, many of which can provide the same information as the job options noted above. We recommend using the job options rather than environment variables to specify job options, as it allows you to have everything self-contained within the job submission script (rather than having to remember what options you set for a given job).

Slurm also provides a number of environment variables within your running job. The following table summarizes those that may be particularly useful within your job (e.g. for naming output log files):

+--------------------------+-----------------------------------------------------------------------------------------+
| Variable                 | Description                                                                             |
+==========================+=========================================================================================+
| ``$SLURM_SUBMIT_DIR``    | The directory from which the batch job was submitted. By default, a new job starts      |
|                          | in your home directory. You can get back to the directory of job submission with        |
|                          | ``cd $SLURM_SUBMIT_DIR``. Note that this is not necessarily the same directory in which |
|                          | the batch script resides.                                                               |
+--------------------------+-----------------------------------------------------------------------------------------+
| ``$SLURM_JOBID``         | The job’s full identifier. A common use for ``$SLURM_JOBID`` is to append the job’s ID  |
|                          | to the standard output and error files.                                                 |
+--------------------------+-----------------------------------------------------------------------------------------+
| ``$SLURM_JOB_NUM_NODES`` | The number of nodes requested.                                                          |
+--------------------------+-----------------------------------------------------------------------------------------+
| ``$SLURM_JOB_NAME``      | The job name supplied by the user.                                                      |
+--------------------------+-----------------------------------------------------------------------------------------+
| ``$SLURM_NODELIST``      | The list of nodes assigned to the job.                                                  |
+--------------------------+-----------------------------------------------------------------------------------------+


Job States
----------

A job will transition through several states during its lifetime. Common ones include:

+-------+------------+-------------------------------------------------------------------------------+
| State | State      | Description                                                                   |
| Code  |            |                                                                               | 
+=======+============+===============================================================================+
| CA    | Canceled   | The job was canceled (could've been by the user or an administrator)          |
+-------+------------+-------------------------------------------------------------------------------+
| CD    | Completed  | The job completed successfully (exit code 0)                                  |
+-------+------------+-------------------------------------------------------------------------------+
| CG    | Completing | The job is in the process of completing (some processes may still be running) |
+-------+------------+-------------------------------------------------------------------------------+
| PD    | Pending    | The job is waiting for resources to be allocated                              |
+-------+------------+-------------------------------------------------------------------------------+
| R     | Running    | The job is currently running                                                  |
+-------+------------+-------------------------------------------------------------------------------+


Job Reason Codes
----------------

In addition to state codes, jobs that are pending will have a "reason code" to explain why the job is pending. Completed jobs will have a reason describing how the job ended. Some codes you might see include:

+-------------------+---------------------------------------------------------------------------------------------------------------+
| Reason            | Meaning                                                                                                       |
+===================+===============================================================================================================+
| Dependency        | Job has dependencies that have not been met                                                                   |
+-------------------+---------------------------------------------------------------------------------------------------------------+
| JobHeldUser       | Job is held at user's request                                                                                 |
+-------------------+---------------------------------------------------------------------------------------------------------------+
| JobHeldAdmin      | Job is held at system administrator's request                                                                 |
+-------------------+---------------------------------------------------------------------------------------------------------------+
| Priority          | Other jobs with higher priority exist for the partition/reservation                                           |
+-------------------+---------------------------------------------------------------------------------------------------------------+
| Reservation       | The job is waiting for its reservation to become available                                                    |
+-------------------+---------------------------------------------------------------------------------------------------------------+
| AssocMaxJobsLimit | The job is being held because the user/project has hit the limit on running jobs                              |
+-------------------+---------------------------------------------------------------------------------------------------------------+
| ReqNodeNotAvail   | The requested a particular node, but it's currently unavailable (it's in use, reserved, down, draining, etc.) |
+-------------------+---------------------------------------------------------------------------------------------------------------+
| JobLaunchFailure  | Job failed to launch (could due to system problems, invalid program name, etc.)                               |
+-------------------+---------------------------------------------------------------------------------------------------------------+
| NonZeroExitCode   | The job exited with some code other than 0                                                                    |
+-------------------+---------------------------------------------------------------------------------------------------------------+

Many other states and job reason codes exist. For a more complete description, see the ``squeue`` man page (either on the system or online).

