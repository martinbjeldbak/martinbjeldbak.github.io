---
title: "Add pgfplots mark for last point"
date: 2014-05-24 23:12:11 +0200
comments: true
<!-- categories: pgfplots latex -->
---

Marking the last data point in line charts with a specific symbol makes it much easier to distinguish between types of lines on a plot when printing in grayscale, or if the reader is colorblind to the colors you have chosen for your graph.

This can be achieved by defining a new pgfplots style you can mix in whenever you need to mark the last point. It simply checks every data point upon reading from the file and marks it only if it is the last in your data file.

{% highlight latex %}
\pgfplotsset{
  mark end/.style={
    scatter,
    scatter src=x,
    scatter/@pre marker code/.code={
      \pgfmathtruncatemacro\usemark{
        (\coordindex==(1))
      }
      \ifnum\usemark=0
      \pgfplotsset{mark=none}
      \fi
    },
    scatter/@post marker code/.code={}
  },
}
{% endhighlight %}

This works perfectly when importing data from a csv or tsv file with data points, but I haven't yet gotten it to work with functions. Here is an example of using the style on a plot with 4 different lines:

![Plot generated using the mark end style]({{ site.images.post_path }}/mark-end.png)

As you can see, the mark corresponding to each line is also mirrored in the legend. It is even possible to resize the marks to make them larger and more readable on print. The LaTeX code to generate the above graph can be seen on ShareLaTeX [here](https://www.sharelatex.com/project/5381186f40c592fe1409c16b?r=903e7523&rs=ps&rm=d). Hope this helped!

Inspired from [this](https://tex.stackexchange.com/questions/116690/pgfplots-marks-mandatory-for-1st-and-last-point) stackoverflow post showing how to mark every <em>n</em>th point.
