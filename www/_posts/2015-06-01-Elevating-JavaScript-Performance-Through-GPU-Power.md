---
layout: post
author: Alexander Myltsev
title: "Jetpack-to-CUDA"
subtitle: "Elevating JavaScript Performance Through GPU Power"
date: 2015-06-01 18:37:18 +0400
comments: true
header-img: "img/mozilla_firefox_large.jpeg"
tags: [JavaScript, CUDA, GPGPU, Mozilla, Mozilla-JetPack, Firefox]
---

The article was originally posted at [Mozilla Labs blog](https://blog.mozilla.org/labs/2010/01/elevating-javascript-performance-through-gpu-power/) in January, 2010. Mozilla Labs doesn't store original content of archived articles, so I published it here below as it was back in year 2010.

The article won't be ever composed without significant help of my MIPT fellows: [Andrey Ustyuzhanin](https://events.yandex.ru/lib/people/3361/) and Andrey Nikolaev.

Preface
-----

My name is Alex. I am one of developers who helped make it possible to utilize [CUDA][cuda] from within [Jetpack][jp]. (See [Jetpack-to-CUDA project][jp2c])

Andrey is Ph.D. graduate at Moscow Institute of Physics and Technology. His interests include parallel computing and distributed algorithm development. The goal of this article is to share our ideas on how to elevate JavaScript performance by utilizing the GPU's parallel processing power. Some of these ideas formed the basis of "Jetpack-to-CUDA".


Why Does This Matter?
-----

People are collaborating. People are using the internet to collaborate more than ever before. Collaboration on the internet has been evolving at a rapid pace, and the applications and technology that will drive the next wave of internet collaboration will require even greater technical complexity and more significant computing resources than is currently available through the browser environment today. While text documents, videos, music, and image-base forms of collaboration are now common place, there are many needs require a level of compute performance beyond the web platform as it exists today, such as:

* consumption of high-quality digital video or music streams,
* complex image or speech recognition,
* manipulation and processing large pictures of nature or space,
* processing large sets of tabular data locally in the browser,
* complex animations with DOM elements (via DirectX or OpenGL),
* exploring 3D worlds, such as [SecondLife](http://en.wikipedia.org/wiki/Second_life) or an [OpenSim Grid](http://en.wikipedia.org/wiki/OpenSimulator#Public_grids),
* real-time audio and video editing,
* having an integrated development environment that runs entirely in the browser

There are endless examples of such complex uses of the internet platform that are just not feasible with the status quo web platform. Developers have tried to overcome such barriers in the past with client-side enhancements like ActiveX, Netscape Plugins, Java Applets, but each in its own way was flawed and failed to gain mass adoption. It is possible that the [Native Client][native-client] project will change all this, but standardization of such initiatives across the browser landscape is a lengthy endeavor. For the near future the tools that the developer uses to provide a rich user experience remain JavaScript and ActionScript, plug-ins, such as the ones previously mentioned, are significantly limited by the architectural mismatch of performance requirements they place on the CPU.

Web-developers have the JavaScript and ActionScript skills with which to develop complex applications, but the browser is currently incapable of providing the compute resources needed to run them. Then Jetpack entered the scene. With Jetpack you can easily extend Firefox and enhance JavaScript's abilities and reach on the client. However, this complicated functionality also requires greater speed with which to process it. JavaScript, being an interpreted language, is much slower than a compiled language such as C++, for example.

As we talk about fast processing, let us also think about how fast a uni-processor computer can ever really be, because there are [significant barriers][1] to making uni-processor systems infinitely faster. As a result, software development today is focusing on multi-tasking on multi-processor compute platforms. Developers can end many of their performance and system resource struggles by utilizing the thousands of parallel GPU processors available now on the average client machine with appropriate [CUDA runtime environment][cuda-learn] installed.

GPU-ed JavaScript
-----

By utilizing the power of the GPU, it is possible to create extremely powerful Firefox extensions with only Jetpack and JavaScript, thereby enabling many of the applications we discussed above. There are two possible approaches for bringing GPU power to JetPack:

1. the extension of the Jetpack API
2. and the extension of JavaScript's syntax

Each of these approaches has its pros and cons, which we describe briefly in the rest of the article. We will also focus on possible ways of exploiting these two approaches from a developer's point of view.

### API Extending

Let’s start with something simple. Suppose we’ve got collection of numbers of a significant size we'd like to operate on (example 1):

```javascript
var numbers = new Array(), size = 1000;
for (var i = 0; i < size; ++i)
    numbers[i] = i;
```

Additionally, we want to get a collection of squared numbers (example 2):

```javascript
var sqrFun = function(v) { return v * v; };
for (var i = 0; i < size; ++i)
    numbers[i] = sqrFun(numbers[i]);
```

Depending on complexity of sqrFun and the size of the collection, the same calculation on the CPU could take much time, as the CPU is likely busy with JavaScript interpretation, page rendering, misc. applications, operating system routines, and so on. During all of these CPU cycles, what is the GPU doing? Well, most likely, it is rendering relatively simple GUI associated with operating system and running applications (if you’re not playing games at the time).

### Jetpack.toGPU()

So why just not to give GPU some work while CPU is busy with something else?

Consider the following (example 3):

```javascript
var resNumbers = Jetpack.toGPU( function(nums, numsSize) {
    var sqrFun = function(v) { return v * v; };
    for (var i = 0; i < size; ++i)
        numbers[i] = sqrFun(numbers[i]);
}, numbers, size);
// some job here
document.write(resNumbers);
```

Someday in future it would be nice to have a more effective programming language system capable of analyzing the source code from scratch and making decision what to parallelize and what not. Today program analysis is a complicated task, so programming language systems need hints for effective implementations. Jetpack.toGPU() is that hint. Under the hood of Jetpack.toGPU() there is translation of `function(nums, numsSize) {…}` to CUDA Kernel and a method of sending C-code to the GPU. Jetpack.toGPU() should be non-blocking. It means “some job” will be done while resNumbers is calculating. Of course, a developer could write their own CUDA Kernel in C to be sent to the GPU. This technique is like in similar to [PyCUDA][pycuda] project. Such mechanisms provide developers an interesting opportunity: Wouldn’t it be great to have one of your favorite JavaScript libraries powered by GPU? We think it would only require small source code changes to have a GPU-ed library.

To understand the concept of writing programs which can be parallelized "for free" (auto-parallelizable) one can review the development recommendations for OpenMP and MPI. In reviewing these recommendations, your thoughts while developing might start resembling the following:

"If I put these instructions in that sequence, I will never get parallelization, but if I change the sequence of same instructions, I’ll probably achieve a parallelization of my capability."

Quite a break from the norm, isn’t it?  That's because of the sequential nature of so-called "imperative programming" (JavaScript is imperative). Imperative programming was designed for uni-processor sequential processing, not for multiprocessing. But there are other ways to express the same algorithm which provide parallelization for free.

### Functional, Hence Parallel For Free

We have some ideas on the parallelization of so-called imperative programs. Another paradigm for algorithms expression is functional programming which seems to have more potential to be auto-parallelization friendly. In functional programs you process data by utilizing functions that do not require instructions regarding how to perform their computation. To better understand this paradigm, you can review [Functional JavaScript][2]. Consider the following (example 4):

```javascript
var sqrFun = function(v) { return v * v; };
map(sqrFun, numbers);
```

The function calculates a squared sequence of numbers, without instruction about how to iterate through the collection of numbers (compare with example 2). How the iteration is performed is up to the environment that provides the map function. Such code could be parallelized in the following way (example 5):

```javascript
var resNumbers = Jetpack.toGPU(function(nums, numsSize) {
    return map(sqrFun, numbers); }, numbers, size);
```

Developers can use the functional style while at the same time achieving parallelization for free. Jetpack could provide domain specific language (remembering an SICP course: primitive elements, means of combination, and means of abstraction) for GPU processing. Such DSLs have been worked out in F#, OCaml and Haskell. So, code expressed in the functional style could be parallelized for free more readily than its imperative equivalents.

If you find functional programming puzzling, you can use its interesting offspring LINQ, which is much easier to understand. You might be surprised to find that jQuery is very similar to LINQ in many ways.

### LINQ

LINQ is one particular case of functional programming applied to the imperative programming technique. With LINQ, you are able to write SQL-like queries to different kinds of collections (JavaScript arrays, MySQL data, XML data, JSON data, and so on). See [jLINQ][jlinq] and [JSING][jsinq] projects for further details. Auto-parallelization in LINQ is implemented by Microsoft in [PLINQ][plinq] of [PFX Library][pfx-lib], which is embedded in .NET Framework 4.0.

Consider the following (example 6):

```javascript
var resNumbers = Jetpack.Linq.toGPU().from(numbers).map(sqrFun);
```

### Jetpack GPUs jQuery

Let’s think of [jQuery as LINQ to JavaScript][3]. Jetpack could be used to extend jQuery to include parallelization features. If one had quite a large DOM, one could use these extended parallelization features to elements faster than is presently possible within the browser's JavaScript engine (with thousands of processors processing DOM-objects in parallel).

One possible implementation of this idea is as follows (example 7):

```javascript
$("div.test").add("p.quote").addClass("blue").slideDown("slow").toGPU();
```

With parallelization feature enhancements in place, the average DOM could grow significantly without the traditional impact on performance.

Syntax Extending
-----

There are a lot of interesting projects and ideas on the web in the area of parallelization ([Cω][cw], [BrookGPU][brook-gpu], [JAVAR][javar], [JCSP][jcsp], [Parallel Java][par-java], [Java Titanium][java-titanium], and many others) which could benefit the JavaScript syntax in Jetpack by implementing new paradigms and techniques for code auto-parallelization... which would be a great topic for another article.

Conclusion
-----

We’ve presented some ideas on how to make JavaScript computations faster (in processing and development) by implementing auto-parallelization within Jetpack. We believe that with such advancements in place, developers could begin to write imperative, parallelizable code. Such advancements are also very important for existing imperative JavaScript libraries. With the techniques outlined in this article, developers would be empowered to write very exiting functional code that is parallelized for free. We think functional code has greater potential to be auto-parallelized than its imperative equivalents. LINQ is an excellent case of functional programming utilized for collection processing. jQuery is very similar to LINQ, they share qualities that make them good candidates for auto-parallelization implementations, the results of which would be dramatic.

If you have comments or questions regarding the ideas discussed in this article, please take a minute or two and post a comment. Your feedback is very important to the writers, and the community as a whole.

Have fun, join the Open Source movement, and help make the web better! Best regards and happy elevating!

Thanks
-----

I’d like to thank staff of [Computer Science Chair][cs-mipt] of [MIPT](mipt) for support and friendly environment. Also I’d like thank personally Andrey Nikolaev (head of parallel computations laboratory, chair of informatics, MIPT) for discussions and technical assistance in article preparation.

[mipt]: http://mipt.ru/
[cs-mipt]: http://cs.mipt.ru/
[cuda]: http://en.wikipedia.org/wiki/CUDA
[jp]: https://wiki.mozilla.org/Jetpack
[jp2c]: http://mozillalabs.com/jetpack/2009/11/10/jetpack-0-5-contest-a-winner/
[native-client]: http://en.wikipedia.org/wiki/Google_Native_Client
[cuda-learn]: http://www.nvidia.com/object/cuda_learn.html
[pycuda]: http://mathema.tician.de/software/pycuda
[jlinq]: http://hugoware.net/Projects/jLinq
[jsinq]: http://www.codeplex.com/jsinq
[plinq]: http://en.wikipedia.org/wiki/LINQ#PLINQ
[pfx-lib]: http://en.wikipedia.org/wiki/Parallel_FX_Library
[cw]: http://en.wikipedia.org/wiki/C%CF%89
[brook-gpu]: http://graphics.stanford.edu/projects/brookgpu/
[javar]: http://www.aartbik.com/JAVAR/index.html
[jcsp]: http://www.cs.kent.ac.uk/pubs/2002/1382/
[par-java]: http://www.cs.rit.edu/~ark/pj.shtml
[java-titanium]: http://titanium.cs.berkeley.edu/
[1]: http://microsoft.cs.msu.su/events/hpcschool2009/Documents/materials/mon/20090720_smith.pptx
[2]: http://osteele.com/sources/javascript/functional/
[3]: http://www.robrich.org/archive/2008/05/31/jquery---linq-to-javascript.aspx
