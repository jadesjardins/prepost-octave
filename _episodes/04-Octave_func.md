---
title: "Writing Octave scripts and functions"
teaching: 0
exercises: 0
questions:
- "How does one create their own Octave functions?"
- "what is the difference between a script and a function?"
- "How can we take advantage of the Octave IDE in order to develop functions effectively?"
objectives:
- "Build Octave functions effectively"
- "Understand the difference between the Octave 'base' workspace and the 'function' workspace"
keypoints:
- "Functions are text files that can be created, modified, and tested efficiently with the Octave IDE"
---


## Writing an Octave *., script

#### **An Octave script is a *.m text file containing Octave commands.**

When doing research we often want to execute a specific series of commands over and over and over and over... again. Ideally we do not have to type out the sequence of commands every time. A script allows you save a sequence of Octave commands in a text file and then execute that sequence from the command line by calling the name of the *.m file.

~~~
edit snssmean
~~~
{: .source}

![Octave edit snssmean]({{ page.root }}/fig/oct_edit_snssmean.png)

~~~
sum(indata)/length(indata)
~~~
{: .source}

> ## Scripts and functions need to be in the Octave path in order for them to be interpreted at the command line
> By default new scripts and functions are saved into the Octave working directory (which is in the Octave path by default). If we save the script elsewhere we would also need to make sure that the location of the script is included in the Octave search path. This is a common barrier while learning to write code in Octave.
{: .callout}

> ## Also!.. make sure to save the changes that you make to a script
> When calling a script from the command window Octave does not interpret what is in the Editor window, but rather what is in the save file. Common confusion comes from editing the text in the Editor and then running it again before saving it... resulting in behaviour that does not match the content of the script in the editor!
{: .callout}

Now we can call this script at the command line by entering the name without the *.m extension.

~~~
snssmean
~~~
{: .source}

~~~
error: 'indata' undefined near line 1 column 5
error: called from
    snssmean at line 1 column 12
~~~
{: .output}

Notice that if we simply call the script now we receive an error with helpful information about what went wrong and where. Something names "indata" does not exist when it is called as an input at line one of the "snssmean" script. This is because a script uses variables from the 'base' workspace and returns and output variables back to the 'base' workspace. If the script has a variable that does not exist in the base workspace as an input the script will fail.

Let's create the "indata" variable and then try it again.

~~~
indata=rand(1:10);
snssmean
~~~
{: .source}

~~~
ans =  0.40417
~~~
{: .output}

We will all get different output values each time that we run this because we are creating indata with a random number generator "rand". Also notice that because we did not specify an output name (no leading "=") Octave store the result in the default variable name "ans" in the base workspace.

#### **Converting a script to a function.**

A function is like a script except that it explicitly defines the inputs and outputs that pass between the 'base' workspace and the 'function' workspace.

Lets convert out snssmean script into a function where we explicitly define the inputs and outputs.

~~~
edit snssmean
~~~
{: .source}

![Octave edit snssmean function]({{ page.root }}/fig/oct_edit_snssmean_fcn.png)

~~~
function outdata=snssmean(indata)
outdata=sum(indata)/length(indata)
~~~
{: .source}

> ## Did you save the changes in the Editor?
> Note the star on the Editor tab for the function.
{: .callout}

Now we can call our function passing specific variables into and storing the output in specified variables in the base workspace.

~~~
x=rand(1,10);
y=snssmean(x)
~~~
{: .source}
~~~
outdata =  0.68303
y =  0.68303
~~~
{: .output}

> ## Why is the output printed twice? and why is it named differently each time?
> This is because we did not use a ";" to suppress the output in the base workspace where the output is stored in "y", nor did we suppress the output inside the function when the output is stored in the 'function' workspace as 'outdata'.
{: .callout}

#### **Breakpointing to interact with the "function" workspace**

When developing, debugging or profiling code it can be valuable to have your Command Window pointed to the "function" workspace instead of the 'base' workspace. This can be achieved by adding a "breakpoint' to the Editor margine (by clicking to the left of the line number).

![Octave edit breakpoint]({{ page.root }}/fig/oct_edit_brkpnt.png)

When a breakpoint is enabled in the editor view of a function any execution of the function will pause when the selected line is reached and the Command Window will point to the "function" workspace. Repeat the function call after enabling a breakpoint at line 2, then explore the variables in the function workspace.

~~~
y=snssmean(x)
~~~
{: .source}
~~~
stopped in /home/james/Desktop/snss_2019/SNSS_2019_Octave/snssmean.m at line 2
2: outdata=sum(indata)/length(indata)
debug>
~~~
{: .output}

At this point the standard output in the Command Window gives a summary of where the workspace is pointed and the command prompt changes to "debug>". Notice also that the Editor has an "arrow" indicating the line in the function that the interpreter is about to execute.

![Octave edit breakpoint indicator]({{ page.root }}/fig/oct_edit_brkpnt_ind.png)
 
We can now query the variables in the function workspace with a call to "whos".

~~~
whos
~~~
{: .source}
~~~
Variables in the current scope:

   Attr Name        Size                     Bytes  Class
   ==== ====        ====                     =====  =====
  a     argn        1x1                          1  char
    f   indata      1x10                        80  double

Total is 11 elements using 81 bytes
~~~
{: .output}
 
Calling "return" at the command line will release the active breakpoint and continue the execution of the function normally.

~~~
return
~~~
{: .source}
~~~
outdata =  0.68303
y =  0.68303
>>
~~~
{: .output}

In this case the snssmean functions terminates normally, return the variable values to the standard output at the command line, then returns to the ">>" base workspace prompt.

> ## Interacting with the breakpoint state
> Notice the navigation buttons in the toolbar of the Editor while you are in an active breakpoint. these tools allow you to move around the execution steps of the function or exit the debug mode.
> ![Octave edit breakpoint tools]({{ page.root }}/fig/oct_edit_brkpnt_tool.png)
{: .callout}

#### **Adding a special kind of input to the function: VARARGIN**

It can be valuable to allow a function to accept an unlimited number of optional inputs. This can be achieved with the special kind of input named "varargin". varargin passes a cell array to the function that can be parsed to obtain optional inputs that are defined within the function.

Let's provide an optional input with which we can provide a file name that will trigger the output to be saved to file. This will also give us a chance to implement a couple "if" conditions!

~~~
edit snssmean
~~~
{: .source}

![Octave edit varargin]({{ page.root }}/fig/oct_edit_varargin.png)

~~~
function outdata=snssmean(indata,varargin)
outfname='';
ofi=(find(strcmp('outfname',varargin)));
if ~isempty(ofi);outfname=varargin{ofi+1};end
outdata=sum(indata)/length(indata)
if ~isempty(outfname);
   save(outfname,'outdata');
end
~~~
{: .source}

> ## Sorry for the brutally "efficient" variable name "ofi"!
> When using varagin we have to parse the input to find where the optional values are. In this case "ofi" stands for "Output Filename Index", which hold the index into the varagin cell that reads "outfname".
{: .callout}

Now when we call our snssmean function we have the option of saving our output variable to a file. We accomplish this by providing a "key and value pair" as inputs to the function as follows:

~~~
snssmean(x,'outfname','out.mat');
~~~
{: .source}

Notice now, that after running this call of the snssmean function, not only was the value returned to the variable and printed to the standard output, but there is also a new file listed in the File Bowser under the current directory named "out.mat". This file contains the output variable in the *.mat format.

You can also use the Bash like "ls" command to list the contents of the working directory.

~~~
ls -l
~~~
{: .source}
~~~
total 232
drwxr-xr-x 2 james james   4096 Jun  8 12:48 data
drwxr-xr-x 2 james james   4096 Jun  8 12:48 functions
-rw-r--r-- 1 james james    122 Jun  8 19:27 out.mat
-rw-r--r-- 1 james james 158267 Jun  8 12:48 parallel-3.1.2.tar.gz
-rw-r--r-- 1 james james    233 Jun  8 19:16 snssmean.m
-rw-r--r-- 1 james james  61073 Jun  8 12:48 struct-1.0.15.tar.gz
~~~
{: .output}

> ## Did you know that Octave is highly compatible with Matlab?
> Yes it is! .. but with some important differences. For example, not all of the *.mat file formats are compatible across the two platforms. In fact the default *.mat format save by Octave is not compatible with Matlab.
{: .callout}

Lets add a "try catch" flow to snssmean make sure that the files written by this function will be compatible with both Octave and Matlab.

![Octave edit try catch]({{ page.root }}/fig/oct_edit_try_catch.png)

~~~
function outdata=snssmean(indata,varargin)
outfname='';
ofi=(find(strcmp('outfname',varargin)));
if ~isempty(ofi);outfname=varargin{ofi+1};end
outdata=sum(indata)/length(indata)
if ~isempty(outfname);
    try OCTAVE_VERSION;
    save('-mat7-binary',outfname,'outdata');
    catch,
    save(outfname,'outdata');
    end
end
~~~
{: .source}

> ## There are other Octave/Matlab differences that you may want to watch out for while crossing platforms
> Here are some hints:
> - OCTAVE_VERSION
> - save('-mat7-binary','fname','var');
> - ! system
> - load empty file
> - ...
> - %#
> - ^ **
> - '"
> - end endif endfor
> - as of Octave 4.0.0 GUI
> - graphics handles
> - && \|\|
> - JIT (Just In Time compiler)
> - compiler
{: .callout}

