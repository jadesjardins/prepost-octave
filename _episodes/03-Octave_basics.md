---
title: "Basics of using Octave"
teaching: 0
exercises: 0
questions:
- "How does one work with data in Octave?"
- "How to run a typical performance prototyping exploration in Octave on the local machine?"
- "What are some of the key issues to consider when assessing and improving performance of computations in Octave?"
objectives:
- "Run test calculations in the Octave IDE on the local machine"
- "Examine the timing of Octave code execution"
- "Prioritise development using the code profiler in Octave"
keypoints:
- "There are many ways to write even simple calculations on multi-dimensional arrays in Octave"
- "Some ways of writing the code perform substantially better than others"
- "Code should be profiled and optimized as much as possible before starting a production run of batch jobs on the cluster."
---


## The basics of working with data in Octave

Before we get to profiling code performance lets take a look at how the functions operate in the Octave environment. Octave is and interpreted language. This means that it will read some text, interpret it in a very specific way and then ask the cpu(s) to perform the operations that are defined by the text.

For our purposes we can think of Octave doing two very important things:
1. it stores information in memory as different kinds of variables
2. it can perform operations (or functions) on those variable

Lets open the Octave Integrated Development Environment (IDE) and take a look at these capabilities.

![Octave Integrated Development Environment]({{ page.root }}/fig/oct_ide.png)

> ## Anatomy of the Octave Integrated Development Environment (IDE)
> The Octave IDE is made up of several sections in the Graphical User Interface (GUI). Each of these are important to understand as we move towards interacting with data building/optimizing functions.
> 1. Command Window: This is where we interact with Octave using the Command Line Interface (CLI) creating variables and performing operations on them.
> 2. Current Folder: This is the directory that Matlab is pointing to. This is the part of the file system that Matlab sees and is its starting point for relative paths.
> 3. Workspace: this is a summary of the variables that are accessible to the Command Window
> 4. Command History: This is the interactive list of operations that have been called from the Command Window.
> 5. Editor: This is a text editor with added features for modifying Octave interpreted text files (*.m files). 
>
> {: .source}
{: .callout}

#### **Creating and indexing into variables in the Octave CLI**
Making new variables (or modifying existing variables) is accomplished using the "=" character.

We can create a new variable named "x" and make it equal to a series of numbers:
~~~
x=[1:10]
~~~
{: .source}

> ## There are several ways to store information in the workspace
>There are several types of variables in Octave, such as the numeric array above, but there are also strings, cell arrays and structures.
{: .callout}

Note that a semi-colon ends a statement and suppresses the result from being printed to the command line:
~~~
x=[1:10];
~~~
{: .source}

Once variables exist in the Workspace we can index into them using brakets:
~~~
x(5)
~~~
{: .source}

When creating variables or indexing into them punctuation takes on several meaning.
The following reads as: variable "y" contains ("=") an array ("[]") of two rows (";" separates rows) where the first is 1 to 10 (":" count consecutively) and the segond is 1 every 2 to 20 (":2:" count by two):
~~~
y=[1:10;5:5:50];
~~~
{: .source}

We can perform calculations across arrays by indexing portions of array and calling operations across them.
The following reads as: variable "z" contains the first row of "y" and all the columns ("y(1,:)) minus the content of "x".
~~~
z=y(1,:)-x;
~~~
{: .source}
Note that in this case the "," separates the dimensions in the indexing of "Y" and the ":" means all of the indexes of the dimension.


#### **Executing functions on variables in the Octave CLI**

We can also perform operations on variables by passing them to functions and have the results stored in a new variable:
~~~
z=mean(x);
~~~
{: .source}

> ## Functions are text files (*.m) that are stored somewhere in the Octave search path.
> All functions that are called from the Octave command line have a text file (*.m) that Octave is able to find. The directories that Octave will find functions in is called its "search path" or "path". New directories can be added to the "path" using the "addpath" function at the command line or several other interactive methods from the IDE.
> The function "which [fcn_name]" will tell us what file is is being interpreted in the function call.
> ~~~
> which mean
> ~~~
> {: .source}
> ~~~
> 'mean' is a function from the file /usr/share/octave/4.2.2/m/statistics/base/mean.m
> ~~~
> {: .output}
> The function "edit [fcn_name]" will open the associated text (*.m) file in the Octave Editor.
> ~~~
> edit mean
> ~~~
> {: .source}
> ![Octave edit mean]({{ page.root }}/fig/oct_edit_mean.png)
{: .callout}

Operations can include making new figure windows and plotting variables.
~~~
figure;plot(x);
~~~
{: .source}

![Octave simple plot]({{ page.root }}/fig/oct_plot.png)

#### **Common flow operation in Octave**
Beyond performing operations on arrays Octave contains many of the flow operation typical to programming languages. These flow operations control the squential operation of a series of comands. Here we describe the Octave venacular for some of the flow language that we will use later in our function development.

#### ***"if" statements***
"Conditional" flow allows you to have code that will only operate if specified conditions are true.
~~~
x=1;
if x;
   disp('true');
else
   disp('false');
end
~~~
{: .source}
~~~
true
~~~
{: .output}

> ## 1 and 0 are typically synonymous with TRUE and FALSE boolean states.
> What happens if "x" is set to some other value than 1 or 0? What about negative values?
{: .callout}

#### ***"catch" control***
Sometimes there are states in functions that can cause errors resulting in the termination of the process. Sometimes these error states should be allowed to exist, but rather than exiting the function the code should just perform a dirrent section of code. The "try - catch" flow allow for error conditions to exist in code and rather than exiting the function a differnet part of the function is executed before continuing.
 
~~~
try junk;
   disp('ok');
catch,
   disp('not ok');
end
~~~
{: .source}
~~~
not ok
~~~
{: .output}
> ## "try" captures erros and exectutes the "catch" block before continuing on.
> What happens if you simple type call "junk" at the command line? what about if you repeat this "try - catch" example after "junk=1"?
{: .callout}

#### ***"for" loop***
In programming we frequently want to do something over and over and over and over.. again while changing some parameters on each itteration. This can be accomplished in most (all?) programming languages with "for" loops. The way to state a for loop in Octave is as follows.

~~~
for i=1:10;
   disp(['now on number: ',num2str(i)]);
end
~~~
{: .source}
~~~
now on number: 1
now on number: 2
now on number: 3
now on number: 4
now on number: 5
now on number: 6
now on number: 7
now on number: 8
now on number: 9
now on number: 10
~~~
{: .output}

... where "i" is variable that contains the collection "1:10". The code between the "for" statement and "end" statement is executed for each item in the "collection".
