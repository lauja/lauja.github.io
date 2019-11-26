---
layout: post
title:  "Optimising Vectors"
date:   2019-11-14
---
So vector drawables have been around since Android 5.0. They tools provided in Android Studio make them easy to work with. You can easily right click on the **drawable** folder and go to **New -> Vector Asset** and easily import an SVG file. I recently came across examples where I was seeing warnings about **Very long vector path**:
<br/><br/>
![very long vector path](/images/optimising_vectors/very_long_vector_path_large.png)
<br/><br/>
I was intrigued so I tracked down the issue that added this lint check and it seemed to have used an arbitrary value of over 800 to be deemed as “very long”:
<br/><br/>
![lint warning issue](/images/optimising_vectors/lint_warning_issue.png)
<br/><br/>
It also mentioned that other tools can be used like SVGO to optimise the vector. See https://github.com/svg/svgo for more details. There is also an SVGO Compressor for Sketch for those who get to use it: [https://github.com/sketch-hq/svgo-compressor](https://github.com/sketch-hq/svgo-compressor), which will run it automatically as you export your SVG.

I also found a tool written by Alex Lockwood (of [https://shapeshifter.design](https://shapeshifter.design) fame) called Avocado: [https://github.com/alexjlockwood/avocado](https://github.com/alexjlockwood/avocado) that does a similar optimisation on your vector drawable xml files. This is a Node.js app so you can install it via:

{% highlight shell %}
npm install -g avocado
{% endhighlight %}

You can then run something like:

{% highlight shell %}
avocado ic_logo_visa.xml -o ic_logo_visa_min.xml
{% endhighlight %}

or omit the -o and it will replace the existing file
<br/><br/>
![avocado results](/images/optimising_vectors/avocado_results.png)
<br/><br/>
So this cut down my original file significantly, but it was still a touch over at 807 characters.

The problem is caused by the one that is used for the whole SVG, when it can effectively be split up. This was possibly caused by running SVGO which had the “mergePaths” setting on by default, or if in Sketch/Illustrator the layers are flattened before exporting. I found that leaving the paths separate and then running Avocado on it, seemed to solve the problem.

One path (1244 chars):
<br/><br/>
![one path](/images/optimising_vectors/one_path.png)
<br/><br/>
Broken down in to multiple paths (longest path 832 chars):
<br/><br/>
![multiple paths](/images/optimising_vectors/multiple_paths.png)
<br/><br/>
Multiple paths after Avocado (longest path 732 chars):
<br/><br/>
![multiple paths after avocado](/images/optimising_vectors/multiple_paths_after_avocado.png)
<br/><br/>
So it fixed the problem, but I leave you with these thoughts from Florina Muntenescu ([https://upday.github.io/blog/vector_drawables_optimisation](https://upday.github.io/blog/vector_drawables_optimisation))

> The VectorDrawable rendering duration is influenced by the size of the image, and by its complexity. If the drawable contains complicated paths then it will take longer to draw. <br/><br/> …. <br/><br/> Is your VectorDrawable really complex, big, and with complicated paths? Then it’s just better to use a raster image. The size of the APK will grow, but the performance of your app won’t be decreased by the VectorDrawable render time. Instead of using PNG files, start using [WebP images](https://developers.google.com/speed/webp/).


And again, it’s super easy to convert to WebP in Android Studio. You can right click a PNG or JPEG and convert to WebP.