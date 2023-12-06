---
description: 'The Business Challenge: Analyzing Big Data Quickly.'
---

# Apache® DataSketches™

A software library of [_stochastic_](https://en.wikipedia.org/wiki/Stochastic) [_streaming algorithms_](https://en.wikipedia.org/wiki/Streaming\_algorithm)

_"A truly excellent example of theoretically-informed algorithm engineering"_ &#x20;

\- Prof. Graham Cormode, University of Warwick



### What is a Sketching Algorithm?

In the analysis of **big data** there are often problem queries that **don’t scale** because they require **huge compute resources** and time to generate exact results. Examples include **count distinct**, **quantiles**, **most-frequent** items, **joins**, **matrix computations**, and **graph analysis**.

If approximate results are acceptable, there is a class of specialized algorithms, called streaming algorithms, or [sketches](docs/Background/SketchOrigins.html) that can produce results orders-of magnitude faster and with mathematically proven error bounds. For interactive queries there may not be other viable alternatives, and in the case of real-time analysis, sketches are the only known solution.

For any system that needs to extract useful information from big data these sketches are a required toolkit that should be tightly integrated into their analysis capabilities. This technology has helped Yahoo successfully reduce data processing times from days or hours to minutes or seconds on a number of its internal platforms.



### Getting Started

You can try the library in python as follows:

```python
from datasketches import hll_sketch, tgt_hll_type


n = 1 << 18
sk = hll_sketch(lg_k = 12)
for x in range(n):
    sk.update(x)
print(sk.get_estimate())
```

```
260297.46789489232
```

This project is dedicated to providing a broad selection of sketch algorithms of production quality. Contributions are welcome from those interested in further development of this science and art.

[Fast](docs/Architecture/LargeScale.html#speed)

[Sketches](docs/Background/SketchOrigins.html) are _fast_. The sketch algorithms in this library process data in a single pass and are suitable for both real-time and batch. Sketches enable streaming computation of set expression cardinalities, quantiles, frequency estimation and more. In addition, designing a system around sketching allows simplification of system's architecture and reduction in overall compute resources required for these heretofore difficult computational tasks.\
\
\
\
\
\
\
This is the DataSketches website source. Please visit the main [DataSketches website](https://datasketches.apache.org) for more information.

If you are interested in making contributions to this site please see our [Community](https://datasketches.apache.org/docs/Community/) page for how to contact us.

### How the website works

* The website is published directly from a specially named _asf-site_ branch. The content of this branch is generated automatically by _Jekyll_ from the _master_ branch whenever changes are detected in the _master_ branch. One should never modify the content of the _asf-site_ directly.
* The _master_ branch consists primarily of GitHub compatible _MarkDown_ documents, which hold all the written content.
* There are two navigation mechanisms on the site to help the user find documents: the _nav\_bar_ at the top of each page and the table-of-contents _toc_ drop-down menus on the left of each page. Individual pages can link to each other using standard MarkDown links.

### How to contribute content to the website

In order to contribute changes to the website content, you will need to fork this repository to your own GitHub profile.

If you only need to change an existing page, edit the relevant MarkDown page locally and submit a pull-request to _master_.

However, if you need to add a new page to the website, you may need to modify the _toc_ to enable users to find it:

* Create the new MarkDown document with the appropriate layout definition, and copyright notice. This can be copied from any of the existing pages. The types of available layouts can be found in the _/\_layouts/_ directory. Almost all site documents use the _doc\_page_ layout. Place the new page in an appropriate subdirectory in _master_.
* The _toc_ is generated statically by the developer/author, when it needs to be updated, by running a small Java program called `TocGenerator.java` located in _/src/main/java/org/datasketches/docgen/_. The TocGenerator takes input from the _/src/main/resources/docgen/toc.json_ file and save the output html in _/\_includes/toc.html_ in _master_. Please do not edit the _toc.html_ file directly.
* The _toc.json_ file is pretty easy to figure out. It is a tree structure of two types of elements, _Dropdown_ and _Doc_. Each element has 4 or 5 _key:valu_e pairs. Make sure you structure the JSON correctly with matching braces and brackets, and with commas between tree elements.
* Run the table of contents generator. The `runTocGenerator` method is a static member of `TocGenerator.java`. You can run this from your preferred IDE or from the command line. You should see the genenerated HTML as output to the console.
* Once you have run the generator, ensure that your new entry is included in the `toc.html` file under the `_includes` subdirectory.
* If you have Jekyll installed on your computer you can visually check the _toc_ for proper operation before submitting your PR.
* Lastly, submit your pull request for review!
