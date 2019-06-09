---
title: "Introduction to the data, resources and strategy"
teaching: 0
exercises: 0
questions:
- "Where did the demo data come from and why does it required Advanced Research Computing (ARC) and High Performance Computing (HPC) resources?"
- "what are the tools that are used to solve some of the data handling problems and how do they fit together in the project workflow?"
objectives:
- "Understand the properties of the data and requirements involved in scaling up the size of samples"
- "Understand what procedures are required to migrate from working on a local computer to working on a remote cluster"
- "Understand what tools can be used to accomplish specific tasks of ARC projects that span local and remote HPC systems"
keypoints:
- "The remote clusters are not only for unsupervised batch production processing but can be part of several stages of a research project lifecycle"
---

## Sander's Parallelogram and the problem of interpreted perception

![Sander's parallelogram]({{ page.root }}/fig/sander_parallel.png)

Lines AE and EC are the same length when measured with a ruler, although we tend to perceive them as different lengths. This is likely due to our uncontrolled interpreted perceptual processing that "simplifies" the relationship between the lines as a rectangle that is leaning into the third dimension.

## Electroencephalography (EEG) and the study of brain activity dynamics

![EEG sample]({{ page.root }}/fig/eeg_sample.png)

Parts of the brain are capable of generating electrical fields that are large enough to pass through the skull and be picked up by electrodes on the surface of the scalp. By recording these voltages at a typical sampling rate around 500Hz and with a dense montage of recording site locations (e.g. 128 electrodes) several aspects of brain region activation and interaction can be examined.
 
## The resources in a local workstation and remote computing cluster environment.

![Resources and tools]({{ page.root }}/fig/resource_tools.png)

{% include links.md %}

When scaling up the scope of research computing it not necessarily a one step process from the work that was being done on the local workstation to submitting jobs. There are several resources available for transitioning and exploring the code execution as it migrates to the cluster computing environment.

## The strategy for migrating from the local workstation to production job batching typically involves profiling of code performance.

With high level interpreted languages like Octave a simple process like baseline correction (removing the mean from an array) can be stated in many ways, particularly when working with multidimensional matrices. Further, different ways of executing the process (that result in identical results) can have substantially different performance and resource requirements.

In the lesson we profile an example of "vectorizing" a mean subtraction procedure on a multidimensional array.

#### **Element-by-element looping of mean column subtraction**

![Element by element mean subtraction]({{ page.root }}/fig/element_by_element.png)

~~~
for r_ind=1:size(data,1);
   for p_ind=1:size(data,3);
      outdata(r_ind,:,p_ind)=data(r_ind,:,p_ind)-mean(data(r_ind,:,p_ind));
   end;
end;
~~~
{: .source}

#### **Array subtraction with loop through columns**

![Array subtraction loop]({{ page.root }}/fig/array_subt_loop.png)

~~~
for c_ind=1:size(data,2);
outdata(:,c_ind,:)=data(:,c_ind,:)-mean(data,2);
end;
~~~
{: .source}
... or better, just calculate the identical mean once.
~~~
c_mean=mean(data,2);
for c_ind=1:size(data,2);
outdata(:,c_ind,:)=data(:,c_ind,:)-c_mean;
end;
~~~
{: .source}

#### **Array subtraction with repmat expansion**

![Array subtraction repmat]({{ page.root }}/fig/array_subt_repmat.png)

~~~
outdata=data-repmat(mean(data,2),1,size(data,2));
~~~
{: .source}

#### **Array subtraction by binary expansion or broadcasting**

![Array subtraction binary expansion]({{ page.root }}/fig/array_subt_binary_exp.png)

"bsxfun" binary expansion
~~~
outdata=bsxfun(@minus,data,mean(data,2));
~~~
{: .source}

broadcasting
~~~
outdata=data-mean(data,2);
~~~
{: .source}


## The strategy for migrating from the local workstation to production job batching may also involve data management, interactive development/visualization on the remote system and interactive profiling on the compute nodes.

#### **Intuitive data management with SSHFS and Meld**

#### **Interactive IDE development and data visualization on the VDI node via TigerVNC**

#### **Interactive performance profiling on the compute nodes via "salloc"**



