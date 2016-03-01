---
layout: post
title: "Google Summer of Code 2016 project internships in Literate Modeling and Diagnostics"
comments: true
tags: ['education', 'discussion']
redirect_from: ""
permalink: /2016/03/01/google-summer-of-code-2016-project-internships-in-literate-modeling-and-diagnostics
references:
---

I am mentoring up to two projects this summer with Google Summer of Code (GSoC16), following on from the success of last year's projects. They are described in section 5 of [this page](https://incf.org/gsoc/2016) on the sponsoring organization's web site. Thanks to the [INCF](http:/incf.org) for their support.

A brief summary of the projects, reproduced from the INCF page:

### Next-generation neuroscience data and model exploration tools

#### 1. Improving the graphical exploration and analysis environment, Fovea.

[Fovea](https://github.com/robclewley/fovea) is a user-extensible graphical diagnostic tool for working with complex data and models. It is a library of Python classes that are built over components of [PyDSTool](http://pydstool.sf.net) and Matplotlib. Fovea’s present capabilities and applications to neural data and models were developed significantly in GSoC 2015 (see [previous blog posts](http://robclewley.github.io/spike_detection_with_fovea/)), but there are several directions remaining to develop and explore to maximize the utility and accessibility of this package to less specialized users. 

A primary capability of Fovea is to assist the creation of organizing "layers" of graphic data in 2D or 3D sub-plots that are associated with specific calculations. These layers can be dynamically updated as parameters are changed or as time advances during a time series, and they can be grouped and hierarchically construced to ease navigation of complex data. Examples of layer usage is to display:

* Feature representations, such as characteristic threshold crossings
of time series, clusters or principal component projections, domains
of an approximation’s error bound satisfaction
* Augmenting meta-data, such as vectors showing velocity or
acceleration at a specified position or time point during a simulation
* Other diagnostic quantities that can visually guide parameter
  fitting of a model or algorithm.
  
In addition, GUI buttons and dialog boxes, and command-line
interfacing can provide additional interactivity with the graphical
views.

**Aims**: Depending on student interest and skills, there are three
  possible directions this project could usefully take:
  
  1. Redevelop  the existing prototype “application” into an actual Django or flask  web-based app running over a local server (maybe using Bokeh);
  
  2. Extend the development of the existing core tools to work  better for dynamical systems applications(especially biophysical models of neural excitability);
  
  3. Continue to streamline the workflow for  constructing new applications by improving core design and adding  convenient utilities;
  
  4. Integrate existing functionality with the  literate modeling prototype discussed in Project 2 (maybe in  collaboration with a student working directly on that project, if both projects run).
  
As part of any direction of this project, there might also be
opportunity to create innovative new forms of plug-and-play UI
component that will assist in visualization and diagnostics of neural
(or similar) data.

**Skills**: _Required_: Python, Matplotlib, GUI design, documentation
tools, working knowledge of calculus and geometry or statistics and
data science

_Desirable_: git, general experience with either model simulation or data science techniques; agile development methodology; technical writing; Django or similar, where applicable, dynamical systems theory, where applicable.

**Keywords**: Python, data science, GUI, diagnostics

#### 2. “Literate modeling” capabilities for data analysis and model development
  
A basic workflow and loosely structured package for "literate modeling" [has recently been explored](http://robclewley.github.io/ipython-notebooks-for-literate-modeling/). This prototype reuses several PyDSTool classes and data structures, but is intended to work mostly standalone within other computational or analytical frameworks. It is a part-graphical, part-command-line tool to explore and test model behavior and parameter variations against quantitative objective data or qualitative features while working inside a core computational framework (e.g. simulator), and could work well as an integration with the Fovea package mentioned in Project 1.

“Literate modeling” is a natural extension to “literate programming” and reproducible research practices that is intended to create a rich audit trail of the model development process itself (in addition to recording the provenance of code versions and parameter choices for simulation runs once a model is developed). Literate modeling aims to add structured, interpretive metadata to version-controlled models or analysis workflows through specially structured “active documents” (similar to the Jupyter notebook). Examples of such metadata include validatory regression tests, exploratory hypotheses (as sets of expected features and data-driven constraints), and data-driven contexts for parameter search or optimization. There is presently no other software tool that aims to provide this advanced support for hypothesis generation and testing in a computational setting!
 
**Aims**: Refine, improve, or redesign existing prototype classes, database structure, and user workflow for the core functions of literate modeling. Add other core functionality to support other workflows. Option to design a browser-based interface to this system using Django or similar technology or to collaborate on integration with Fovea (see Project 5.1). Document the tools, including creating a tutorial for the website. Test according to predefined examples that will include manipulations of the Hodgkin Huxley neural model and other simple dynamical systems.

**Skills**: _Required_: Python, git, documentation tools, automatic
code generation principles, agile development methodology

_Desirable_: Pweave/Ptangle or similar; Django, flask, or similar; SQL; XML; general experience with either model simulation or data science techniques; technical writing; working knowledge of calculus and geometry or statistics and data science

**Keywords**: Python, data science, literate modeling, productivity

### Inquiries and Applications for GSoC 2016

Please contact INCF at `gsoc@incf.org` so they can give you access to our discussion group for INCF mentors and students. Please prepare a resume and 1-2 page statement about your specific interests in the work and why you'd make a good fit with the project. You can share that within the discussion group once you're logged in.

{% assign ref=page.references %}

