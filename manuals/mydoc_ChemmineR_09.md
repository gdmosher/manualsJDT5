---
title: Streaming Through Large SD Files
keywords: 
last_updated: Fri Jul  1 13:53:21 2016
---

The `sdfStream` function allows to stream through SD
Files with millions of molecules without consuming much memory. During
this process any set of descriptors, supported by
`ChemmineR`, can be computed (*e.g.* atom pairs,
molecular properties, etc.), as long as they can be returned in tabular
format. In addition to descriptor values, the function returns a line
index that gives the start and end positions of each molecule in the
source SD File. This line index can be used by the downstream
`read.SDFindex` function to retrieve specific molecules
of interest from the source SD File without reading the entire file into
R. The following outlines the typical workflow of this streaming
functionality in `ChemmineR`.  

Create sample SD File with 100 molecules: 

{% highlight r %}
 write.SDF(sdfset, "test.sdf") 
{% endhighlight %}


Define descriptor set in a simple function: 

{% highlight r %}
 desc <- function(sdfset) 
 cbind(SDFID=sdfid(sdfset), 
	# datablock2ma(datablocklist=datablock(sdfset)), 
	 MW=MW(sdfset),
	groups(sdfset), APFP=desc2fp(x=sdf2ap(sdfset), descnames=1024,
	type="character"), AP=sdf2ap(sdfset, type="character"), rings(sdfset,
	type="count", upper=6, arom=TRUE) )  
{% endhighlight %}


Run `sdfStream` with `desc` function and
write results to a file called `matrix.xls`:


{% highlight r %}
 sdfStream(input="test.sdf", output="matrix.xls", fct=desc, Nlines=1000) # 'Nlines': number of lines to read from input SD File at a time 
{% endhighlight %}


One can also start reading from a specific line number in the SD file.
The following example starts at line number 950. This is useful for
restarting and debugging the process. With `append=TRUE`
the result can be appended to an existing file. 

{% highlight r %}
 sdfStream(input="test.sdf", output="matrix2.xls", append=FALSE, fct=desc, Nlines=1000, startline=950) 
{% endhighlight %}


Select molecules meeting certain property criteria from SD File using
line index generated by previous `sdfStream` step:


{% highlight r %}
 indexDF <- read.delim("matrix.xls", row.names=1)[,1:4] 
 indexDFsub <- indexDF[indexDF$MW < 400, ] # Selects molecules with MW < 400 
 sdfset <- read.SDFindex(file="test.sdf", index=indexDFsub, type="SDFset") # Collects results in 'SDFset' container 
{% endhighlight %}


Write results directly to SD file without storing larger numbers of
molecules in memory: 

{% highlight r %}
 read.SDFindex(file="test.sdf", index=indexDFsub, type="file",
 outfile="sub.sdf") 
{% endhighlight %}


Read AP/APFP strings from file into `APset` or
`FP` object: 

{% highlight r %}
 apset <- read.AP(x="matrix.xls", type="ap", colid="AP") 
 apfp <- read.AP(x="matrix.xls", type="fp", colid="APFP") 
{% endhighlight %}


Alternatively, one can provide the AP/APFP strings in a named character
vector: 

{% highlight r %}
 apset <- read.AP(x=sdf2ap(sdfset[1:20], type="character"), type="ap") 
 fpchar <- desc2fp(sdf2ap(sdfset[1:20]), descnames=1024, type="character")
 fpset <- as(fpchar, "FPset") 
{% endhighlight %}


