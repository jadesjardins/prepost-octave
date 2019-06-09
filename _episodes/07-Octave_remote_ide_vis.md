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


Prepare the directory to send to the remote file system via the gra-dtn1 node
~~~
cd ~/Desktop/snss_2019
ls -l
ls -al SNSS_2019_Octave
ls -al SNSS_2019_Octave
rm -rf SNSS_2019_Octave/.git
~~~
{: .source}

Use scp to perform the transfer
~~~
scp -r SNSS_2019_Octave jdesjard@gra-dtn1.computecanada.ca:/home/jdesjard
~~~
{: .source}

launch the VNC viewer and direct it toward the gra-vdi node
~~~
vncviewer
~~~
{: .source}
 
![vncviewer parameters]({{ page.root }}/fig/vnc_param.png)


Enter guest credentials at the gra-vdi login...
![gra-vdi login]({{ page.root }}/fig/vnc_gra-vdi_login.png)


![gra-vdi desktop]({{ page.root }}/fig/vnc_gra-vdi_desktop.png)

![gra-vdi term]({{ page.root }}/fig/vnc_gra-vdi_term.png)

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

~~~
nix-env --install --attr nixpkgs.octaveFull
~~~
{: .source}

~~~
octave
~~~
{: .source}

![gra-vdi octave]({{ page.root }}/fig/vnc_gra-vdi_octave.png)

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
> Remember that this is a shared system that is available to all Compute Canada members. Leaving idle sesisons connected reduces the availablility of the resource to others! Simply use the top menu items System > Log out...
{: .callout}

{% include links.md %}



