---
title: "Using Octave interactively on a compute node via salloc"
teaching: 0
exercises: 0
questions:
- "How to work interactively with Octave on Graham compute node?"
- "What are the appropriate tasks for working on a compute node via salloc?"
objectives:
- "Establish interactive access to Graham compute nodes and run performance tests"
- "Understand the appropriate use of interactive acces sot compute nodes"
keypoints:
- "salloc interactive access to compute nodes provides an ideal environment for performance profiling"
---

Working from the gra-vdi node we can now see that the code we developed on our local computers is compatible with the Nix installation of Octave that we have on the Graham system. The issue now is that we want to start testing at larger scales and the gra-vdi node is not optimized for computation performance, but rather for visualization performance. It is time to test the computation on the compute nodes. Although we can only access the compute nodes via the Slurm scheduler we are still able to work interactively on these nodes by requesting an interactive instance from the scheduler via salloc.

![Resources and tools]({{ page.root }}/fig/resource_tools.png)

It is typical to interact with the Slurm scheduler on Graham from the login node. Let's start by SSH'ing to graham.computecanada.ca from our local terminal.

> ## Note: on Linux and Mac this can be done by opening a new terminal. On Windows computers we recommend using GitBash or another client like MobaXterm.
{: .callout}
~~~
ssh jdesjard@graham.computecanada.ca
~~~
{: .source}
~~~
Last login: Thu Jun  6 10:14:21 2019 from 139.57.28.171

Welcome to the ComputeCanada/SHARCNET cluster Graham.

 Documentation: https://docs.computecanada.ca/wiki/Graham
Current issues: https://status.computecanada.ca/
       Support: support@computecanada.ca

NEWS

May 14: Due to the large numbers of file changes per day, backups are not
        completing every day. We are working to resolve the issue.  

Jun 3: The per user /scratch space quota has been reduced to 20 TB and 1M files
        https://docs.computecanada.ca/wiki/FS_quotas

~~~
{: .output}

> ## guest account will be asked to provide a password.
> If you do not have sshkeys setup for your account (which you ar unlikely to have with a guest## account, you will be prompted for a password each time you execute a secure command (scp, ssh) towards the system. In the long term we recommend setting up sshkeys for your accounts as this makes logins more convenient and secure.
{: .callout}

Now that we are on the graham login node we can navigate to our ~/SNSS_2019_Octave directory that we scp'ed over the network earlier.

~~~
cd ~/SNSS_2019_Octave
~~~
{: .source}

Although we can start programs in the login node and run procedures it strongly discouraged as this is the hardware where all users are logging into the system to interact with the scheduler and perform small (typically administrative) tasks. That being said we can prepare the environment for here in the login node and those changes will be carried over to the compute node when the scheduler assigns those resources to our job request.

Lets load our modules now so that they will be ready for use once we are assigned to a compute node.

~~~
module load nix
~~~
{: .source}

Typical unsupervised batch compute jobs are submitted to the scheduler via "sbatch", in order to get and interactive job on a compute node however we make that request with a call to "salloc".

~~~
salloc --account=cossw19-wa --reservation=cossw19-wr_cpu --cpus-per-task=1 --mem 1000m -t 01:00:00
~~~
{: .source}
~~~
salloc: Pending job allocation 15713317
salloc: job 15713317 queued and waiting for resources
salloc: job 15713317 has been allocated resources
salloc: Granted job allocation 15713317
salloc: Waiting for resource configuration
salloc: Nodes gra1021 are ready for job
NOTE: A newer set of packages have been released

newly released: 18.03.100.133402
currently used: 18.03.88.133351

You can make these newer packages available by running
(this can sometimes take a minute or so)

$ nix-channel --update

This is safe as it can always be undone by running

$ nix-channel --rollback
~~~
{: .output}


> ## Notice that we are specifying an --account and a --reservation when interacting with the cluster.
> all jobs executed by the cluster are "billed" towards the fair share of usage across accounts on the system. This being the case all jobs need to be associated with an account to "charge" the usage to (charge as is affect priority.. not $$$). For this training event we placed a reservation on the system such that a set of compute nodes will be kept idle except for jobs being submitted to the specified --reservation. Further, only specific accounts have access to the reservation and the username that is logged into the system submitting the jobs needs ot be associated with the accepted account.
{: .callout}

Now that we are in an interactive session on compute node (notice the prompt change to {username}@gra###) we can check that our modules loaded at the login node followed our session here.

~~~
module list
~~~
{: .source}
~~~
Currently Loaded Modules:
  1) nixpkgs/16.09 (S)   2) imkl/11.3.4.258 (math)   3) icc/.2016.4.258 (H)   4) gcccore/.5.4.0 (H)   5) ifort/.2016.4.258 (H)   6) intel/2016.4 (t)   7) openmpi/2.1.1 (m)   8) StdEnv/2016.4 (S)   9) nix

  Where:
   S:     Module is Sticky, requires --force to unload or purge
   m:     MPI implementations / Implémentations MPI
   math:  Mathematical libraries / Bibliothèques mathématiques
   t:     Tools for development / Outils de développement
   H:                Hidden Module
~~~
{: .output}

With the Nix module loaded we can now start Octave and work within the resources that we requested for this interactive session.

~~~
octave
~~~
{: .source}

~~~
octave: X11 DISPLAY environment variable not set
octave: disabling GUI features
GNU Octave, version 4.2.2
Copyright (C) 2018 John W. Eaton and others.
This is free software; see the source code for copying conditions.
There is ABSOLUTELY NO WARRANTY; not even for MERCHANTABILITY or
FITNESS FOR A PARTICULAR PURPOSE.  For details, type 'warranty'.

Octave was configured for "x86_64-pc-linux-gnu".

Additional information about Octave is available at http://www.octave.org.

Please contribute if you find this software useful.
For more information, visit http://www.octave.org/get-involved.html

Read http://www.octave.org/bugs.html to learn how to submit bug reports.
For information about changes from previous versions, type 'news'.

octave:1>
~~~
{: .output}

> ## Note the the GUI features are disabled.
> Unlike the gra-dvi nodes that are optimized for visualization and graphical usage, compute nodes a set up to focus on calculations and command line interfacing.
{: .callout}

Now that the prompt have changed to "octave:#>" Octave will be interpreting the commands that we ender in the terminal. The terminal is now like the Command Window of our previous local and gra-vdi Octave IDE sessions.

Let’s repeat some of the profiling tests that we performed earlier and explore using a graham compute node from within an interactive job.

~~~
profile clear;
profile on;
load('data/test_data.mat');
tic;
outdata=zeros(size(data));
for c_ind=1:size(data,2);
    outdata(:,c_ind,:)=data(:,c_ind,:)-mean(data,2);
end;
toc
p_info=profile('info');
profile off;
~~~
{: .source}

~~~
Elapsed time is 1.41335 seconds.
~~~
{: .output}

Notice that all of the features that we were using previously to profile the performance on our local machines is available on the compute node as well.

~~~
profshow
~~~
{: .source}

~~~
   #          Function Attr     Time (s)   Time (%)        Calls
----------------------------------------------------------------
  18               sum             1.279      74.70          256
   1              load             0.378      22.10            1
   4             zeros             0.022       1.27            1
   5              mean             0.015       0.87          256
  19          binary /             0.009       0.51          256
  20          binary -             0.006       0.35          256
   3              size             0.001       0.03          514
   6            nargin             0.001       0.03         1026
  11             false             0.000       0.02          257
  10          prefix !             0.000       0.02          512
   9         isnumeric             0.000       0.01          256
  24 __profiler_data__             0.000       0.01            1
  17            strcmp             0.000       0.01          256
  12         binary ==             0.000       0.01          768
  16               fix             0.000       0.01          256
  14             ndims             0.000       0.01          256
  15          isscalar             0.000       0.01          256
   7          binary <             0.000       0.01          256
  13            ischar             0.000       0.01          256
   8          binary >             0.000       0.01          512
~~~
{: .output}

> ## Ideal location and methods for interactive profiling prior to large scale testing
> This is an ideal place to interactively profile the performance of the code as it shares as much in common with the batch job production runs a scale. This is the same compute hardware that the batch jobs will run one, it is using the same software stack, and we are event requesting the resources from the scheduler in the same way that we would be doing for productions runs to sbacth!
{: .callout}


{% include links.md %}


