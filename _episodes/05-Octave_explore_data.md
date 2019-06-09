---
title: "Exploring data arrays in Octave"
teaching: 0
exercises: 0
questions:
- "How does one load data into Octave"
- "Index into multidimensional arrays"
- "How can data be visualized"
objectives:
- "Load a demo data set into an Oplore specific portions of the data array and apply function"
- "Visualize aspects of the data array"
keypoints:
- "Get data into the Octave workspace"
- "visualization is an important part of data analytics and procedure development"
---


## Reading data into Octave

#### **Reading in a Octave/Matlab array**

Octave has a flexible toolset for building custom import functions for binary data files and has native tools for load common data formats like *.csv, etc. For simplicity sake in this lesson the demo data has been stored as an Octave/Matlab array in a *.mat files. Octave/Matlab array can be easily read into the Octave workspace using the "load" function by providing the filename as the input.

~~~
load('data/test_data.mat');
~~~
{: .source}

The array saved in the "data/test_data.mat" is stored in the "data" variable once loaded into the base workspace (the variable was saved as "data" at creating time and the variable name is stored in the *.mat file).

#### **Exploring the data array.**

When working with a multi-dimensional numeric array one of first things that we will want to understand is the shape of the matrix. We can find this by calling the "size" function on the "data" array.

~~~
size(data)
~~~
{: .source}

~~~
ans =

     32    256   1024
~~~
{: .output}

> ## The dimension are always in order of ROW, COLUMN, PAGE
> In an EEG data file it is typical that ROWS are recording sites, COLUMNS are time points (about 1 point per 4 milliseconds), and PAGES are time epochs (about one second slices of data). In the example data file the data array has 32 recording sites and 256 time samples per 1024 time epochs.
{: .callout}

We can look at the 10th recording site (ROW 10) time course (all COLUMNS) of the 100th time epoch (100th PAGE) by creating a new figure window and indexing into the data array within a plot call.

~~~
figure;plot(data(10,:,100));
~~~
{: .source}

![Octave plot data index]({{ page.root }}/fig/oct_plot_data_ind.png)

We can also plot all of the recording sites (all ROWS) time course (all COLUMNS) of the 100th time epoch (100th PAGE) on a surface plot by making a new figure window and indexing indo "data" within a call to "surf".

~~~
figure;surf(data(:,:,100),'linestyle','none');
~~~
{: .source}

![Octave surface data index]({{ page.root }}/fig/oct_surf_data_ind.png)

> ## Notice the optional inputs to "surf" for controlling the figure properties.
> There are many figure parameters that can be set via optional inputs to the plotting functions. When looking for the various properties and how they can be set is is best to take to the internet and search for things like "Octave plot surface properties"
{: .callout}

Notice in this surface plot that each of the 32 recording sites (ROWS) appears to be flat even though when we only looked at the 10th recording site (10th ROW) there were clear fluxuations over the time course. The apparent flat recording sites is due to a large amplitude offset across recording sites. Put another way, the voltage variance between recording sites is substantially larger than the voltage variance across recording sites. Because of this variance discrepancy between recording sites it is difficult to see the fluctuations within recording sites.

In order to remove the variance between recording sites we can calculate a baseline subtraction in which each recording site has its mean time point subtracted from each time point. We can use nested functions and a "broadcasted" subtraction to achieve this within the plotting command.

~~~
figure;surf(data(:,:,100)-mean(data(:,:,100),2),'linestyle','none');
~~~
{: .source}

![Octave surface baseline]({{ page.root }}/fig/oct_surf_baseline.png)



