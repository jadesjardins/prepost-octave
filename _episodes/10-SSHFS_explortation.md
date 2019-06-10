---
title: "Exploring job output from an SSHFS mount to the local machine"
teaching: 0
exercises: 0
questions:
- "How to explore output of batch jobs interactively from the local machine"
- "How can we explore and compare the local file system to the remote file system in a difference viewer"
objectives:
- "establish a mount of the remote Graham file system on the local machine"
- "Use software on the local machine to interact with files on the remote file system"
keypoints:
- "While there are options of running interactive software on the system, it may be more convenient to run the local software accessing remote files via SSHFS mount."
---

Now that batch jobs have run and the relevant information has been stored to Graham's file system we want to be able to explore these files much in the way that we had been exploring the procedures interactively through various methods during this lesson. While there are several options that we have explored so far it is worth exploring the SSHFS mount of the remote file system to the local machine at this point. This strategy allows us to use the software on the local machine to interact with the files on the remote file system.

![Resources and tools]({{ page.root }}/fig/resource_tools.png)

Let's start by creating a "remote_mnt" empty folder next to the SNSS_2019_Octave folder to use as a mount point for the remote copy of our demo directory.

~~~
cd ~/Desktop/snss_2019
~~~
{: .source}

I am am going to use bash to create my empty folder but it can be created with any method that you prefer.

~~~
mkdir remote_mnt
~~~
{: .source}

Now that an empty directory exist for us to use as mount point we can call sshfs to mount the remote content into the local files system at the target directory.

~~~
sshfs jdesjard@gra-dtn1.computecanada.ca:/home/jdesjard/SNSS_2019_Octave remote_mnt
~~~
{: .source}

Now that the remote directory has been mounted to the target remote_mnt folder we can use the local software to interact with the remote files. In the simplest form we can use the bash "ls" in the local terminal to list the content of the remote_mnt folder.

~~~
ls -l remote_mnt
~~~
{: .source}

![list remote_mnt]({{ page.root }}/fig/remote_mnt_ls.png)

This scenario is where difference viewers can be extremely valuable. A difference viewer compares files and or folders side by side highlighting difference between them, often providing tools to transfer, merge and save manipulations. Meld is a powerful and intuitive interactive difference viewer that is capable of difference viewing directory tee and the contained files across three versions.

Let's take a look at the difference between our local and remote demo directory in Meld.

~~~
meld SNSS_2019_Octave remote_mnt
~~~
{: .source}

![sshfs meld]({{ page.root }}/fig/sshfs_meld.png)

Now at a glance we can see which files exist in which location and we can interactively examine the comparisons between files in the directory.

If we can now examine the remote files with local software it is probably a good time to check and make sure that the end result of our batch jobs are producing the same results as our original procedures executed on the local compute.

Let’s open a local instance of Octave and navigate to the folder just below "SNSS_2019_Octave" and "remote_mnt". Then we can load the remote output file and compare it to one of our original local procedures.

~~~
octave
~~~
{: .source}
~~~
cd ~.Desktop/snss_2019
~~~
{: .source}

Now load the output file generated from the remote batch job and rename the array.

~~~
load remote_mnt/out_repmat.mat
remote_outdata=outdata;
~~~
{: .source}

Replicate one of the original local procedures that we ran originally.

~~~
load('SNSS_2019_Octave/data/test_data.mat');
for r_ind=1:size(data,1);
    for p_ind=1:size(data,3);
        orig_outdata(r_ind,:,p_ind)=data(r_ind,:,p_ind)-mean(data(r_ind,:,p_ind));
    end;
end;
~~~
{: .source}

Then, finally, compare the differences of the outputs from the original local process that we started with and the results from the optimized remote batch job execution.

~~~
figure;surf(double(squeeze(orig_outdata(:,:,1))),'linestyle','none');
figure;surf(double(squeeze(remote_outdata(:,:,1))),'linestyle','none');
diff_outdata=orig_outdata-remote_outdata;
figure;surf(double(squeeze(diff_outdata(:,:,1))),'linestyle','none');
~~~
{: .source}

![remote_mnt surface diff]({{ page.root }}/fig/remote_mnt_surf_diff.png)

Knowing that the optimized procedures from the remote batch job did not change the output results we can now also examine the performance profiling information that was saved during the run by loading the profile output into the load Octave instance and exploring it with profshow and profexplore.

~~~
load remote_mnt/prof_repmat.mat
profshow(p_info)
~~~
{: .source}
~~~
   #  Function Attr     Time (s)   Time (%)        Calls
--------------------------------------------------------
   3      load             0.297      89.57            1
  32  binary -             0.016       4.77            1
  19    repmat             0.013       3.80            1
  17       sum             0.006       1.72            1
  18  binary /             0.000       0.03            1
   4      mean             0.000       0.03            1
  33       toc             0.000       0.02            1
  34   profile             0.000       0.01            1
  23       max             0.000       0.00            1
  30   reshape             0.000       0.00            2
   7  binary >             0.000       0.00            4
   1    strcmp             0.000       0.00            3
  14      size             0.000       0.00            6
   6  binary <             0.000       0.00            3
  29      ones             0.000       0.00            3
   9  prefix !             0.000       0.00            4
  31 binary .*             0.000       0.00            1
  21       all             0.000       0.00            3
  20   isempty             0.000       0.00            5
   5    nargin             0.000       0.00            7
~~~
{: .output}

~~~
profexplore(p_info)
~~~
{: .source}
~~~
Top
  1) load: 1 calls, 0.297 total, 0.297 self
  2) binary -: 1 calls, 0.016 total, 0.016 self
  3) repmat: 1 calls, 0.013 total, 0.013 self
  4) mean: 1 calls, 0.006 total, 0.000 self
  5) toc: 1 calls, 0.000 total, 0.000 self
  6) profile: 1 calls, 0.000 total, 0.000 self
  7) strcmp: 2 calls, 0.000 total, 0.000 self
  8) tic: 1 calls, 0.000 total, 0.000 self
  9) prefix !: 1 calls, 0.000 total, 0.000 self
  10) isempty: 1 calls, 0.000 total, 0.000 self
  11) size: 1 calls, 0.000 total, 0.000 self

profexplore> exit
~~~
{: .output}


{% include links.md %}

