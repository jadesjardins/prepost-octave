---
title: "Running Octave batch jobs with profiling enabled"
teaching: 0
exercises: 0
questions:
- "How to submit Octave batch jobs to the scheduler?"
- "What types of workloads are appropriate for batch jobs scheduling via sbatch?"
- "How to submit jobs such that performance profiling information will be available at completion"
objectives:
- "Run unsupervised Octave batch jobs via the sbatch command to Slurm scheduler"
- "Explore the benefits of pre-production test jobs"
keypoints:
- "Batch jobs submitted via sbatch to the scheduler are valuable for pre-production code testing"
---

Have repeated the process of running the profiling tests in a couple environments now, it may be getting apparent that by the time we are intending to to start running profiling tests at large scales the interactiveness of the local machine, gra-vdi, and the salloc jobs may not be necessary any more. In fact as we move towards testing large scale performance the operations may start taking longer so holding up interactive resources may become inappropriate. We may also be at a point where we want to start testing various parameters for our job and we may want to run run several of the procedures simultaneously and only work interactively once that the procedures are complete. In this section we explore submitting jobs that collect and save profiling information to be examine later in an interactive environment.

![Resources and tools]({{ page.root }}/fig/resource_tools.png)

For this process of submitting profiling batch jobs to the scheduler we start in the same scenario as when we were requesting an interactive session from the Slurm scheduler on Graham.

We will start again by SSH'ing to the login node at graha.computecanada.ca

~~~
ssh jdesjard@graham.computecanada.ca
~~~
{: .source}

Then we can navigate to our demo directory and load our modules.

~~~
cd ~/SNSS_2019_Octave
module load nix
~~~

Now that we will be submitting unsupervised batch jobs to be executed on compute nodes whenever resources become available we need to state what the job is to perform and make sure that anything that we want to be able to examine later is save to a file. This is important because we do not get access to the Octave workspace with batch jobs, we only get to work which variables that get save to file for latter access.

The various profiling procedures have been saved as functions in the /functions subfolder of our demo directory. Let's take a look at the parts of these function that will allow us to interact with the required information later after the batch jobs complete.

~~~
cat functions/subm_repmat
~~~
{: .source}

~~~
function outdata=subm_repmat(datafname,varargin)

%HANDLE INPUTS
outfname='';
ofi=(find(strcmp('outfname',varargin)));
if ~isempty(ofi);outfname=varargin{ofi+1};end

proffname='';
pfi=(find(strcmp('proffname',varargin)));
if ~isempty(pfi);proffname=varargin{pfi+1};end

tictoc='on';
tti=(find(strcmp('tictoc',varargin)));
if ~isempty(tti);tictoc=varargin{tti+1};end

%START PROFILING
if ~isempty(proffname);
  profile off;
  profile on;
end

if strcmp(tictoc,'on');
  tic;
end


%PERFORM CALCULATIONS
load(datafname);

outdata=data-repmat(mean(data,2),1,size(data,2));


%END PROFILING
if strcmp(tictoc,'on');
  toc
end

if ~isempty(proffname);
  p_info=profile('info');
  profile off;
  save(proffname,'p_info');
end

%WRITE OUTPUT FILE
if ~isempty(outfname);
  save(outfname,'outdata');
end
~~~
{: .output}

> ## Notice the use of optional inputs to control the procedures by output naming.
> In this example profiling procedures are optional and they are enabled by specifying the names of output file that will hold their collected statistics.
{: .callout}

The next step in submitting batch jobs to the scheduler is creating a submission script. The submission script contains all of the information that the scheduler will need in order to run the requested procedure when enough resources become available among the compute nodes. The submit script contains parameters for the "sbatch" function that define how the jobs is intended to run and defines the resources that it will reserve during execution. The submit script also contains the commands to be executed on the resources that are reserved for the job. The execution portion of our profiling jobs will resemble what we have been doing in the interactive environments except they are not contained in functions and save important parts of the workspace to file for later examination.

Let's create a nex submit script with the "nano" text editor while on the Graham login node.

~~~
nano subm_repmat.sh
~~~
{: .source}

> ## "nano" turns the terminal into a plain text editor.
> Notice now that the terminal is ready to create text file content. Many terminal plain text editors have very little affordance (few clues as to how to use them). Nano provides helpful hints along the bottom of the terminal as to navigating this program. For now we can simply past the text for our submit script and the press Ctrl-X before following the prompts to save the file and exit the program.
{: .callout}

![nano submit script edit]({{ page.root }}/fig/bash_nano_sub_script.png)

Since we have stored all of the sbatch parameters in the submit script now it is only a matter of requesting sbatch to interpret the submit script and put this job into the scheduling queue.

~~~
sbatch subm_repmat.sh
~~~
{: .source}
~~~
Submitted batch job 15716041
~~~

> ## If you ever ask for help regarding these profiling jobs the job ID is probably one of the most valuable pieces of information.
> The Compute Canada systemes have a support team that can help debug or optimize your procedures. If you are ever in need of help regarding the execution of a job the staff will be able learn a lot about the process from the job ID.
{: .callout}

Once that the job is submitted we can check the status of pending jobs with a call to "squeue" or jobs in any state with "sacct".

~~~
sacct -aX -u jdesjard -o jobid,submit,start,state
~~~
{: .source}

~~~
       JobID              Submit               Start      State
------------ ------------------- ------------------- ----------
15707003     2019-06-09T13:40:15 2019-06-09T13:40:19  COMPLETED
15711204     2019-06-09T18:12:24 2019-06-09T18:12:25  COMPLETED
...
15716041     2019-06-09T23:04:02 2019-06-09T23:04:04  COMPLETED
~~~
{: .output}

After the jobs run it now a matter of loading the variables that were are interested in and exploring the outcome in one of the various avenues to interactive that we have explored in this lesson.
{% include links.md %}



