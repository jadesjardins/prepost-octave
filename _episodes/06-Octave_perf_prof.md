---
title: "Profiling the performance of function vectorization"
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

## There are many ways to code a single process like the mean subtraction of a multidimensional array


#### **The mean subtraction result**

Remember from the introduction that our goal is to subtract each ROW's mean in the 3 dimensional data array in order to baseline correct the EEG time series. In this section we will use the Octave profiler to guide changes that we make to code in order to improve the execution performance of the operation. We focus here on function vectorization, or put another way, reducing the number of times Octave interprets calls to basic functions.

![Mean subtraction figure comparison]({{ page.root }}/fig/oct_surf_subt_cmp.png)


#### **Element-by-element looping of mean column subtraction**

In this first method for performing the baseline subtraction the minus calculations are called explicitly on each row and page using nested loops.

![Element by element mean subtraction]({{ page.root }}/fig/element_by_element.png)

~~~
load('data/test_data.mat');
for r_ind=1:size(data,1);
    for p_ind=1:size(data,3);
        outdata(r_ind,:,p_ind)=data(r_ind,:,p_ind)-mean(data(r_ind,:,p_ind));
    end;
end;
~~~
{: .source}

#### **The Element-by-element looping of mean column subtraction works, but exactly how long did it take to execute?**

When scaling up the size of a computation problem on shared clusters it is important to consider the efficiency of the code beyond the correctness of the output. Let's add a timer around the code portion that we are interested in optimizing with a "tic" and "toc" pair.

~~~
load('data/test_data.mat');
tic;
for r_ind=1:size(data,1);
    for p_ind=1:size(data,3);
        outdata(r_ind,:,p_ind)=data(r_ind,:,p_ind)-mean(data(r_ind,:,p_ind));
    end;
end;
toc
~~~
{: .source}
 
~~~
Elapsed time is 2.12267 seconds.
~~~
{: .output}

#### **OK, we know exactly how long the code of interest took to execute, but what portions of the code where taking up the time?**

It is one thing to know how long code is taking to run, but in order to optimize the code efficiently we also need to know what portions of the code were taking up the execution time. In order to break down the timing into the execution pieces we can use the Octave "profiler" to generate reports. These reports are valuable in helping us decide where we should be spending development time to increase the performance of the procedure.


~~~
load('data/test_data.mat');
profile clear;
profile on;
tic;
for r_ind=1:size(data,1);
    for p_ind=1:size(data,3);
        outdata(r_ind,:,p_ind)=data(r_ind,:,p_ind)-mean(data(r_ind,:,p_ind));
    end;
end;
toc
profile off;
~~~
{: .source}
~~~
Elapsed time is 2.3558 seconds.
~~~
{: .output}

The profiler does not automatically print to the standard output of the command line, rather it stores information about the execution of the code that occurs between the calls to "profile on;" and "profile off;". Note that it will continue to append profiling information to existing profile data, so we are running "profile clear;" at the beginning of each test to make sure that we are interpreting results from the only the current process execution.

Once that we have profiling information we can examine it with "profshow" or more interactively with "profexplore".

~~~
profshow
~~~
{: .source}
~~~
   #  Function Attr     Time (s)   Time (%)        Calls
--------------------------------------------------------
   3      mean             1.057      77.37        32768
  13      find             0.050       3.69        32768
   4    nargin             0.037       2.68        98305
   2      size             0.034       2.51        65569
  15       sum             0.033       2.43        32768
  11      true             0.023       1.69        32768
   9     false             0.022       1.59        32769
   6  binary >             0.019       1.41        65536
  17  binary -             0.017       1.21        32768
  14    strcmp             0.015       1.07        32768
   7 isnumeric             0.014       1.00        32768
  12     ndims             0.013       0.95        32768
  16  binary /             0.010       0.74        32768
   8  prefix !             0.009       0.64        32768
   5  binary <             0.009       0.63        32768
  10 binary ==             0.005       0.39        32768
  19   profile             0.000       0.00            1
  18       toc             0.000       0.00            1
   1       tic             0.000       0.00            1
  20 binary !=             0.000       0.00            1
~~~
{: .output}

Given these profiling results we don't need to dig down deeper to know where we should focus in order to maximize the impact of our code optimization. Almost 80% of the time in this code is spent calling the "mean" function. Further, the issue is not necessarily that the mean function is slow, it is likely just that it is being called too often (32768 times to be exact).

We know that we want to reduce the number of times Octave needs to interpret the call to "mean", but let’s also take a quick look at the information provided by "profexplore".

~~~
profexplore
~~~
{: .source}
~~~
Top
  1) mean: 32768 calls, 1.350 total, 1.057 self
  2) binary -: 32768 calls, 0.017 total, 0.017 self
  3) profile: 1 calls, 0.000 total, 0.000 self
  4) size: 33 calls, 0.000 total, 0.000 self
  5) toc: 1 calls, 0.000 total, 0.000 self
  6) tic: 1 calls, 0.000 total, 0.000 self

profexplore>
~~~
{: .output}

Notice now that not only are we given an initial summary like the one from "profview" but it is reduced to the top priorities and the prompt has changes to "profexplore>". This new prompt indicates that we are now in the profile information explorer where we can navigate around the information while trying to determine where to focus our optimization efforts. Entering "help" provides the options for navigating the profile information and then returns the command line to the profexplore prompt.

~~~
help
~~~
{: .source}
~~~
Commands for profile explorer:

exit   Return to Octave prompt.
quit   Return to Octave prompt.
help   Display this help message.
up [N] Go up N levels, where N is an integer.  Default is 1.
N      Go down a level into option N.

Top
  1) mean: 32768 calls, 1.350 total, 1.057 self
  2) binary -: 32768 calls, 0.017 total, 0.017 self
  3) profile: 1 calls, 0.000 total, 0.000 self
  4) size: 33 calls, 0.000 total, 0.000 self
  5) toc: 1 calls, 0.000 total, 0.000 self
  6) tic: 1 calls, 0.000 total, 0.000 self
~~~
{: .output}

We don't need to dig down into this analysis any further, but later it be helpful to dig deeper down into levels of procedure's function stack.

Let's exit the profile explorer and try another version of the process.

~~~
exit
~~~
{: .source}

#### **Array subtraction with loop through columns**

Now that we know that we need to try to reduce the number of times the "mean" function is interpreted by the Octave lets try applying the mean across and entire dimension of the data array before looping the subtraction across the columns.

![Array subtraction loop]({{ page.root }}/fig/array_subt_loop.png)

~~~
load('data/test_data.mat');
profile clear;
profile on;
tic;
for c_ind=1:size(data,2);
    outdata(:,c_ind,:)=data(:,c_ind,:)-mean(data,2);
end;
toc
profile off;
~~~
{: .source}
~~~
Elapsed time is 1.59288 seconds.
~~~
{: .output}

OK, that is faster, from about 2.35 seconds down to about 1.6 seconds. Lets see why with profshow.

~~~
profshow
~~~
{: .source}
~~~
   #  Function Attr     Time (s)   Time (%)        Calls
--------------------------------------------------------
  16       sum             1.509      98.11          256
   3      mean             0.012       0.78          256
  17  binary /             0.008       0.53          256
  18  binary -             0.007       0.42          256
   4    nargin             0.000       0.03         1025
   2      size             0.000       0.02          513
   9     false             0.000       0.02          257
   8  prefix !             0.000       0.01          512
  15    strcmp             0.000       0.01          256
  14       fix             0.000       0.01          256
   7 isnumeric             0.000       0.01          256
  12     ndims             0.000       0.01          256
  13  isscalar             0.000       0.01          256
  10 binary ==             0.000       0.01          768
   5  binary <             0.000       0.01          256
  11    ischar             0.000       0.01          256
   6  binary >             0.000       0.01          512
  20   profile             0.000       0.00            1
  19       toc             0.000       0.00            1
  21 binary !=             0.000       0.00            1
~~~
{: .output}

Notice now that "sum" is the primary culprit. But we don't call sum in our code. It is time to use profexplore to see where sum is being called from.

~~~
profexplore
~~~
{: .source}
~~~
Top
  1) mean: 256 calls, 1.531 total, 0.012 self
  2) binary -: 256 calls, 0.007 total, 0.007 self
  3) profile: 1 calls, 0.000 total, 0.000 self
  4) size: 1 calls, 0.000 total, 0.000 self
  5) toc: 1 calls, 0.000 total, 0.000 self
  6) tic: 1 calls, 0.000 total, 0.000 self

profexplore>
~~~
{: .output}
~~~
1
~~~
{: .source}
~~~
Top
  mean: 256 calls, 1.531 total, 0.012 self
    1) sum: 256 calls, 1.509 total, 1.509 self
    2) binary /: 256 calls, 0.008 total, 0.008 self
    3) nargin: 1024 calls, 0.000 total, 0.000 self
    4) size: 512 calls, 0.000 total, 0.000 self
    5) false: 256 calls, 0.000 total, 0.000 self
    6) prefix !: 512 calls, 0.000 total, 0.000 self
    7) strcmp: 256 calls, 0.000 total, 0.000 self
    8) fix: 256 calls, 0.000 total, 0.000 self
    9) isnumeric: 256 calls, 0.000 total, 0.000 self
    10) ndims: 256 calls, 0.000 total, 0.000 self
    11) isscalar: 256 calls, 0.000 total, 0.000 self
    12) binary ==: 768 calls, 0.000 total, 0.000 self
    13) binary <: 256 calls, 0.000 total, 0.000 self
    14) ischar: 256 calls, 0.000 total, 0.000 self
    15) binary >: 512 calls, 0.000 total, 0.000 self
~~~
{: .output}

OK, so "mean" is where the "sum" is being called from. We still have the same issue then. We should still focus on reducing the number of times that "mean" is being interpreted by Octave.

Noticing that in the last method we are calculating the exact same "mean" on every itereation we should rather store an intermediate array before the loop begins and then just refer to that array rather than recalculating it redundantly.

~~~
load('data/test_data.mat');
profile clear;
profile on;
tic;
c_mean=mean(data,2);
for c_ind=1:size(data,2);
    outdata(:,c_ind,:)=data(:,c_ind,:)-c_mean;
end;
toc
profile off;
~~~
{: .source}
~~~
Elapsed time is 0.0953429 seconds.
~~~
{: .output}

This change has brought us down from 1.6 seconds to 0.09 seconds! Let's see why.

~~~
profshow
~~~
{: .source}
~~~
   #  Function Attr     Time (s)   Time (%)        Calls
--------------------------------------------------------
  18  binary -             0.010      59.84          256
  16       sum             0.007      38.53            1
   2      mean             0.000       0.55            1
  20   profile             0.000       0.50            1
  17  binary /             0.000       0.26            1
   3    nargin             0.000       0.07            5
   8     false             0.000       0.06            2
  19       toc             0.000       0.05            1
  12      size             0.000       0.04            3
  21 binary !=             0.000       0.03            1
  15    strcmp             0.000       0.02            1
   7  prefix !             0.000       0.01            2
   9 binary ==             0.000       0.01            3
   1       tic             0.000       0.01            1
   4  binary <             0.000       0.01            1
   6 isnumeric             0.000       0.01            1
  10    ischar             0.000       0.01            1
  13  isscalar             0.000       0.01            1
  14       fix             0.000       0.01            1
   5  binary >             0.000       0.00            2
~~~
{: .output}

There is only a single call to mean which was the primary culprit. We don't have to reduce the number of times Octave is interpreting "mean" any more. The next issue is the number of times that "binary -" or "minus" is called. 256 times to be exact.


> ## Notice now that we have changed our focus from the output data results to the output timing of the execution.
> In this section we are only looking at the performance output of execution of the calculation, but it is a good idea to check that the code optimizations are not changing the results. We can always check the results of new code improvements by saving different outputs as different variables and then plotting the differences between them.
> ~~~
> load('data/test_data.mat');
> for r_ind=1:size(data,1);
>     for p_ind=1:size(data,3);
>         orig_outdata(r_ind,:,p_ind)=data(r_ind,:,p_ind)-mean(data(r_ind,:,p_ind));
>     end;
> end;
>
> load('data/test_data.mat');
> c_mean=mean(data,2);
> for c_ind=1:size(data,2);
>     new_outdata(:,c_ind,:)=data(:,c_ind,:)-c_mean;
> end;
>
> figure;surf(double(squeeze(orig_outdata(:,:,1))),'linestyle','none');
> figure;surf(double(squeeze(new_outdata(:,:,1))),'linestyle','none');
> diff_outdata=orig_outdata-new_outdata;
> figure;surf(double(squeeze(diff_outdata(:,:,1))),'linestyle','none');
> ~~~
> {: .source}
> ![subtraction surface difference]({{ page.root }}/fig/oct_surf_subt_diff.png)
{: .callout}

> ## Have we done enough?.
> Well, if our test case was at the scale of our production analysis it may not be worth spending more time getting this process to run faster or using up up less resources. But is typically the case that the reason that a research program (and associated code) moves onto an HPC cluster is that the scale of the data is going through a step change. In production our example data may be billions of times this size, so we need to take into account the size of the production run when determining the value of optimization time.
{: .callout}

The scope of the production run is also import to keep in mind when considering how memory is being used. Often saving variables to memory can help improve execution time, but memory usage can also result in expensive execution constraints.

There are memory mistakes to avoid in Octave that do not seem like a problem at small scales, but at large scales result in substantial performance issues. A primary memory issue to avoid is changing the size (growing) an array inside of a loop. Everytime that an array changes size Octave has to re-assign the memory. At a small scale with small data and few iteration this may not be noticable, but as the array becomes larger across more iteration the performance decline increases. This can be resolved by preallocating output array.

~~~
load('data/test_data.mat');
outdata=zeros(size(data));
profile clear;
profile on;
tic;
c_mean=mean(data,2);
for c_ind=1:size(data,2);
    outdata(:,c_ind,:)=data(:,c_ind,:)-c_mean;
end;
toc
profile off;
~~~
{: .source}
~~~
Elapsed time is 0.0978382 seconds.
~~~
{: .output}


#### **Array subtraction with repmat expansion**

![Array subtraction repmat]({{ page.root }}/fig/array_subt_repmat.png)

With this method can remove the large count of "minus" by storing a larger copy of the mean array and subtract the 3 dimensional array from the original data with a single "minus" call. The "repmat" function takes an array and maxes copies of it along a dimension to increase the size.

~~~
load('data/test_data.mat');
profile clear;
profile on;
tic;
outdata=data-repmat(mean(data,2),1,size(data,2));
toc;
profile off;
~~~
{: .source}
~~~
Elapsed time is 0.051177 seconds.
~~~
{: .output}

Now we are down to about 0.05 seconds, but we should also be weary of our memory consumption with this method of creating a full size intermediate array.

#### **Array subtraction by binary expansion or broadcasting**

![Array subtraction binary expansion]({{ page.root }}/fig/array_subt_binary_exp.png)

"bsxfun" binary expansion is a method of applying an operation from one array along a dimension of a larger array without creating a full size intermediate array.
~~~
load('data/test_data.mat');
profile clear;
profile on;
tic;
outdata=bsxfun(@minus,data,mean(data,2));
toc;
profile off;
~~~
{: .source}
~~~
Elapsed time is 0.033097 seconds.
~~~
{: .output}

Another speed improvement and no intermediate array to worry about when considering memory consumption!

> ## "Broadcasting" makes this all seem like a bit of a waste of time.
> in Octave 3.6.0 "broadcasting was introduced to automatically expands binary operations across the dimensions of a larger array. The usage is substantially more simple than "bsxfun" but using it will reduce the backwards compatibility of code and produce errors. Perhaps a good opportunity to introduce "try" and "catch" pairs!
> ~~~
> load('data/test_data.mat');
> profile clear;
> profile on;
> tic;
> outdata=data-mean(data,2);
> toc;
> profile off;
> ~~~
> {: .source}
> ~~~
> Elapsed time is 0.0330579 seconds.
> ~~~
> {: .output}
{: .callout}

> ## Always run multiple samples of the profiling tests.
> You may have noticed that is you run the exact same code multiple times while profiling that the time results could be quite different. The executing speed of a given procedure can vary for several reasons across apparently identical operation (e.g. different things may always be happening in the background on the hardware, etc). This is why it is also very important to also re-examine performance tests across systems!
{: .callout}

