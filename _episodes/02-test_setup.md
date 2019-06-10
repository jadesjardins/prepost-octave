---
title: "Test setup"
teaching: 0
exercises: 0
questions:
- "Will any of these tools work on my laptop?"
- "What are the roles of the various software tools for remote computing and Advanced Research Computing (ARC)?"
objectives:
- "Test that the specific tools using in this lesson work on each laptop in the room."
- "Understand why each of the tools is recommended for various portions of an ARC project lifecycle."
keypoints:
- "There are various ways to use remote computing resources from a local machine"
- "The various access method serve various aspects of an ARC project"
- "free software is available for cross platform remote access but some operating systems are more compatible than others"
---

## Octave IDE

We will be doing most of the development work today in a local installation of the Octave Integrated Development Environment (IDE). At this point we want to test that everybody can open the Octave IDE on their laptops. Once you start Octave it should look like the following.

![Octave IDE]({{ page.root }}/fig/octave_ide.png)


## Bash terminal

The Bash terminal is used to interact with the computer from the Command Line Interface (CLI). This is where we will be doing some administrative tasks and initiating accessing the remote clusters. Terminal will look different across computers but once a terminal is open on your computer it should look something like the following.

![Bash terminal]({{ page.root }}/fig/bash_term.png)

## ssh from Bash terminal

When accessing the remote clusters we will be using the "ssh" program form the terminal. To check that it is installed and available to our terminals let's see what version we have from the terminal.

~~~
ssh -V
~~~
{: .source}

![ssh version]({{ page.root }}/fig/ssh_ver.png)


## Course material download

Make a folder on the Desktop named snss_2019. Inside this folder either clone the demo folder from github or download the zip file and extract it to this snss_2019 folder. From the Bash terminal we can accomplish this as follows.

~~~
cd ~/Desktop
mkdir snss_2019
cd snss_2019
git clone https://github.com/jadesjardins/SNSS_2019_Octave.git
~~~
{: source}

Then navigate into the course directory and list that everything is present.

~~~
cd SNSS_2019_Octave
ls -l
~~~
{: .source}
~~~
total 224
drwxr-xr-x 2 james james   4096 Jun 10 06:52 data
drwxr-xr-x 2 james james   4096 Jun 10 06:52 functions
-rw-r--r-- 1 james james 158267 Jun 10 06:52 parallel-3.1.2.tar.gz
-rw-r--r-- 1 james james  61073 Jun 10 06:52 struct-1.0.15.tar.gz
~~~
{: .output}



## VCN viewer
One of the ways that we will be accessing the remote systems is through Virtual Network Computing (VNC). A VNC connection allows for graphical usage of a remote computer rather than being limited to the CLI. There are instructions for installing TigerVNC on your laptop in the setup page of this event. At this point we just want to check that our laptops can access the login page of the gra-dtn node. We can start TigerVNC by calling "vncviewer" from the Bash terminal.

~~~
vncviewer
~~~
{: .source}

This should bring up a gui for the connection details. We want to connect to gra-vdi.computecanada.ca.

![ssh version]({{ page.root }}/fig/ssh_ver.png)

After pressing "connect" a new window should appear requesting login credentials. This is all that we need for now. We will login to this node later in the lesson.

![vncviewer parameters]({{ page.root }}/fig/vnc_param.png)

![gra-vdi login]({{ page.root }}/fig/vnc_gra-vdi_login.png)

> ## OK! The hardest part is done!
{: .callout}

{% include links.md %}


