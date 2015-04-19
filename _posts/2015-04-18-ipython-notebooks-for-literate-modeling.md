---
layout: post
title: "ipython notebooks for literate modeling"
comments: true
tags: ['technical', 'python', 'web-technology']
redirect_from: ""
permalink: ipython-notebooks-for-literate-modeling
references:
---

{% assign ref=page.references %}

In my previous post, I complained a bit about ipython notebooks not
being the solution to mixing metadata, pre- or post-processor
directives, and code. But, I mentioned an intriguing snippet of code
that I would look into, and indeed this has led to some progress for
me in picturing the use of ipython notebooks for literate modeling.

## Extensions

[YAML_magic](https://gist.github.com/bollwyvl/aa7131d57195f86fb9c0) is
an ipython notebook extension by 'bollwyvl' that enables raw YAML to
be interpreted in a code cell when prefixed by `%%yaml`. This
extension idea opened my eyes to simple ways in which an extension can
enable literate modeling. For one thing, once I understood what to do
with the file (no documentation was provided), I realized how easy it
is to write custom extensions to hook into other kernel processes that
could help me achieve my ideal exploratory modeling setup. But first,
the instructions...

I want to auto-load my extensions and modules for my notebooks but not
for my regular ipython sessions (partly because those loads create
warnings and pollute my default namespace).

Write these config options (based on uncommenting the appropriate
lines in the config file) into
`.ipython/profile_default/ipython_kernel_config.py`, which is probably
in your home directory:

    c.InteractiveShellApp.extensions = ['yaml_magic', 'autoreload']

You can make a copy of an existing config file in that directory if
the file doesn't already exist, and just make sure only kernel-focused
config options are uncommented. The precedence rule is for ipython to
start by interpreting `ipython_config.py` and then the individual
application components read their specific config file and overwrite
any overlapping options.


### Using the extensions

Anyway, configuration aside, I can get 'autoreload' and 'yaml\_magic'
to work and the results are quite
inspirational. [Autoreload](https://ipython.org/ipython-doc/dev/config/extensions/autoreload.html)
simply reloads any module dependencies if they are edited outside of
the session. This is great for exploratory workflows.

yaml\_magic is not fully documented but seems to allow you to write
YAML directly into a code cell, prefix it with `%%yaml` and have the
YAML interpreted into a dictionary (as one might expect). Also, adding
a name after the directive assigns this dictionary to a variable with
that name in the local scope. For instance, I defined the cell:

    %%yaml x
	studycontext-metadata:
	ID: 3
	tag: initial visualization of linearized model's performance

	studycontext-header:
	tag: SC_imports


which displays nothing. But introspecting `x` yields the dictionary

    {'studycontext-header': {'tag': 'SC_imports'},
     'studycontext-metadata': {'ID': 3,
      'tag': "initial visualization of linearized model's performance"}}


I see from the Gist that getting behavior like this is as simple as
adding a couple of decorators to a class that invokes a little
Javascript. Based on this example, I can see a way to implement a
backend project management system that I alluded to in my previous
post.

## Importing from other notebooks

A big issue for me is that my modeling work
involves multiple files and the continued editing of common library
files in the local project. If I'm going to adopt ipython notebooks as
the basis for my workflow then I need to make most, or all, of these
files into `.ipynb` notebook files so that they can all be marked up
and processed similarly. This would normally prohibit importing them
directly into other notebooks when they are needed.

However, I found
[this snippet](http://nbviewer.ipython.org/github/adrn/ipython/blob/1.x/examples/notebooks/Importing%20Notebooks.ipynb),
which I turned into a
[module I named `notebook_importing.py`](https://gist.github.com/robclewley/75b7719119892b99d73b). I
placed it in a local library folder on my python path for safe
keeping. I then edited my kernel config file's option to execute commands on
launch:

    c.InteractiveShellApp.exec_lines = [
        'import notebook_importing'
    ]

This ensures that 'import' is overloaded to check for the `ipynb` file extension and pre-process it appropriately.

## Debugging

One of the most useful things I can do in my IDE is to graphically set breakpoints and trace bugs using the live interpreter. There is the opportunity to interact post-mortem after a bug using `%debug` in the next cell after the traceback, and it works well enough for that aspect of tracing, at least.

## Remaining issues

* I can't easily see contents of a notebook-based script once outside
  of the notebook. I'm not *always* online to use nbviewer, and a
  [local installation](http://stackoverflow.com/questions/18572423/running-ipython-notebook-viewer-locally)
  is not the easiest thing to do for an end user (it's not packaged,
  per se). [This approach](https://github.com/jupyter/nbviewer)
  requires docker but looks relatively simple. Anyway, this is an
  inconvenient extra step to quickly get a view of the code.
* There are no line numbers for reference, and if I can't edit
  natively in my IDE then I can't use breakpoints so easily.
* Doing a `diff` on ipynb-formatted files makes it difficult for the
  end user to track changes with git. For my literate modeling
  workflow idea, there is therefore a need to flatten the json to
  create a regular `.py` file before doing the `diff` -- but to also
  include the markup somehow, in case of comment/tag changes in the
  YAML!


Nonetheless, with some caveats, I now see how I could use ipython
notebooks for the interactive command line workflow along the same
lines that I like to do in regular ipython.
