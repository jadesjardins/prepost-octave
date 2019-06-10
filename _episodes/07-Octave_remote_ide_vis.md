---
title: "Using Octave IDE on the remote machine"
teaching: 0
exercises: 0
questions:
- "How to replicate the interactive Octave IDE work on the Graham cluster?"
- "What performance are we prioritizing by establishing a VNC connection to the gra-vdi node?"
objectives:
- "Work interactively with the Octave IDE from the remote Graham system"
- "Understand the appropriate work load for this access type"
- "Understand the limitations of working from the gra-vdi node "
keypoints:
- "VNC connections to the gra-vdi node are great for interactive development and visualization"
---

At this point we have optimised our code to the point that we could start testing at larger scales. In order to do this we need to find out if the code that we developed on our local machine will run properly on the remote cluster. A convenient way of doing this is by running the Octave IDE on the cluster via the gra-vdi node using a VNC client.

![Resources and tools]({{ page.root }}/fig/resource_tools.png)

Before we get started with accessing the remote system for interactive usage let's send our demo folder over to the Graham /home file system. We should start by Preparing the directory to send to the remote file system via the gra-dtn1 node
~~~
cd ~/Desktop/snss_2019
ls -l
ls -al SNSS_2019_Octave
ls -al SNSS_2019_Octave
rm -rf SNSS_2019_Octave/.git
~~~
{: .source}

Then use scp to perform the transfer.
~~~
scp -r SNSS_2019_Octave jdesjard@gra-dtn1.computecanada.ca:/home/jdesjard
~~~
{: .source}

Now that our demo data files on the Graham /home file system we can start accessing the system to replicate the work that we have been doing locally. Let's launch the VNC viewer and direct it toward the gra-vdi node.

~~~
vncviewer
~~~
{: .source}
 
![vncviewer parameters]({{ page.root }}/fig/vnc_param.png)


Once that the VNC client application reaches the target node it will bring up the node's login page and we can enter out Compute Canada credentials
.
![gra-vdi login]({{ page.root }}/fig/vnc_gra-vdi_login.png)

Once logged into the gra-vdi node we will see an intuitive desktop environment. we can use the top menu to access various resources.
![gra-vdi desktop]({{ page.root }}/fig/vnc_gra-vdi_desktop.png)

Click on the terminal icon in the gra-vdi desktop top menu to launch a new terminal that we will to start setting up our environment.

![gra-vdi term]({{ page.root }}/fig/vnc_gra-vdi_term.png)

The first thing that we want to do now that we have control of our environments on the gra-vdi node is to set up our software environment. In particular we want to be able to load Octave and run it interactively here. We can control what software is loaded in our environment on Graham using modules. The Nix modules allows us to install specific versions of software in our environment, specifically, the Nix installations of Octave are optimized for interactive use. Let's start by loading the the Nix module.

~~~
module load nix
~~~
{: .source}
~~~
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

Now lets see what versions of Octave Nix has available for installation.

~~~
nix search octave
~~~
{: .source}

~~~
warning: using cached results; pass '-u' to update the cache
Attribute name: nixpkgs.octave
Package name: octave
Version: 4.2.2
Description:

Attribute name: nixpkgs.octaveHg
Package name: octave
Version: 4.3.0pre23269
Description:

Attribute name: nixpkgs.octaveFull
Package name: octave
Version: 4.2.2
Description:
~~~
{: .output}


Octave can be installed with several different configurations, many of which are designed to suppress graphical interaction. Let's install the nixpkgs.octaveFull version which includes the full Octave IDE for interactive usage.

~~~
nix-env --install --attr nixpkgs.octaveFull
~~~
{: .source}

Once that the installation is complete we can start Octave on gra-vdi by calling it at the terminal prompt.
 
~~~
octave
~~~
{: .source}

![gra-vdi octave]({{ page.root }}/fig/vnc_gra-vdi_octave.png)

At this point it is a matter of using the gra-vdi instance of Octave to replicate the work that we were doing locally.

Let's use the Octave IDE Command Window to replicate some of the work that we did earlier.
~~~
cd ~/SNSS_2019_Octave
~~~
{: .source}

~~~
load('data/test_data.mat');
for r_ind=1:size(data,1);
    for p_ind=1:size(data,3);
        orig_outdata(r_ind,:,p_ind)=data(r_ind,:,p_ind)-mean(data(r_ind,:,p_ind));
    end;
end;

load('data/test_data.mat');
c_mean=mean(data,2);
for c_ind=1:size(data,2);
    new_outdata(:,c_ind,:)=data(:,c_ind,:)-c_mean;
end;

figure;surf(double(squeeze(orig_outdata(:,:,1))),'linestyle','none');
figure;surf(double(squeeze(new_outdata(:,:,1))),'linestyle','none');
diff_outdata=orig_outdata-new_outdata;
figure;surf(double(squeeze(diff_outdata(:,:,1))),'linestyle','none');
 ~~~
{: .source}

![gra-vdi surface diff]({{ page.root }}/fig/vnc_gra-vdi_surf_diff.png)

> ## Don't forget to logout when you are done!!
> Remember that this is a shared system that is available to all Compute Canada members. Leaving idle sessions connected reduces the availability of the resource to others! Simply use the top menu items System > Log out...
{: .callout}

{% include links.md %}





