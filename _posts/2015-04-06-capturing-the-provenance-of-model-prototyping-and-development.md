---
layout: post
title: "Capturing the provenance of model prototyping and development"
comments: true
tags: [technical, test-driven-modeling, literate-modeling, python]
redirect_from: ""
permalink: capturing-the-provenance-of-model-prototyping-and-development
references:
---

{% assign ref=page.references %}


**Table of Contents**

- [Dependencies](#head1)
- [Viewing and understanding the example](#head2)
- [The math modeling problem](#head3)
  - [Implementation](#head3_1)
- [Other project preparations](#head4)
- [Steps towards a goal: a workflow of test-driven development](#head5)
  - [The 'StudyContext'](#head5_1)
  - [Flexible workflow](#head5_2)
  - [Open ontology](#head5_3)
  - [Example Step types](#head5_4)
  - [StudyContext object declaration syntax](#head5_5)
  - [Commit the project when you'd commit code](#head5_6)
  - [Bifurcate the StudyContext when it gets large or complex](#head5_7)
- [Summary so far](#head6)

------

This is Part 1 of a series of posts.

As I discussed
[recently](http://robclewley.github.io/sumatra-and-simulation-management-or-literate-modeling/)
in a blog post and comments, I want to see the early development
process for mathematical models deconstructed into somewhat formalized
workflows that can be meaningfully captured by version control systems
([VCS](http://en.wikipedia.org/wiki/Revision_control)). To be precise,
I want to structure my workflow and filesystem and add structured
metadata using a project management system. Unlike existing focus on
the simulation of existing models, I want to capture provenance of
model development 'steps,' their diagnostics, and their results, not
just of the primary data (model code, final simulation results,
parameter sets), and not just of unstructuted text descriptions of
notes that might accompany that primary data (like a simple journal).

In this, my first technical and tutorial-like post, I'll share a
prototype workflow for achieving this. I will use a contrived and
minimal model project example and a mostly aspirational set of
management tools (you'll see what I mean).

My literate modeling "solution" embodies some compromises and has many
shortcomings that we can discuss and hopefully build from. But, I also
think it effectively demonstrates the principles. It is also
executable and already version controlled for posterity as a GitHub
'gist'.

<a name="head1"></a>
## Dependencies

I have to assume you know something about applied math, differential
equations, optimization, and python programming, otherwise we'll be
here all month.

You can execute the gist files provided you have installed my
[PyDSTool](http://pydstool.sourceforge.net) package for simulating
dynamical systems models. The Stage 3 revisions of the GitHub repo are
immiediately Python 3 compatible. I forgot in the earlier stages to
ensure that all `print` statements were written as function calls. If
you correct those then all the stages are Python 3 compatible.

The simple and cool
[pyfscache](https://pypi.python.org/pypi/pyfscache) file system object
cache manager is an optional dependency. This package minimizes code
re-execution to build objects that can be reused across scripts in a
common folder. Such a facility would be important in the kind of
workflow that I'm proposing, but in this simple example you can get
away without it. My code runs either way. (*Edit*: to use this package
on my system and with numpy objects, I needed to alter some things in
this package and add some convenience
features. [This is my forked version](https://github.com/robclewley/pyfscache)
for use with this project.)

All of the development was done in a python-friendly IDE
([Wing IDE](https://wingware.com/)) alongside an interactive iPython
session. Interactive sessions are key, here.  Running pre-written
scripts from the OS prompt is only useful when you have everything
already figured out and ready to go. With iPython, I can mock up and
test functions or arrays quickly, and introspect objects to figure out
what to do next. Then I can copy snippets back to the IDE and compose
a clean version for future reference and reuse. Or I edit such
snippets in the IDE and `%paste` them into the session to update its
state. iPython has color syntax highlighting, special meta-commands,
and extensive command history tools. Wing IDE lets me interactively
debug problem elements in my code as I develop it. When I make more
fundamental changes to my code or pollute my session's memory, I just
restart the session and re-run whichever script I'm currently working
on. The combination of these two tools make me very productive.

If you were actually performing the modeling steps yourself, you'd
obviously want to have Git locally installed, and a GitHub account as
an online repository.

<a name="head2"></a>
## Viewing and understanding the example

Ideally, the gist would be presented online in a convenient fashion
that makes it self-documenting and amenable to easy sharing,
discussion, and collaboration, in the same way that the
[notebook viewer](http://ipython.org/notebook.html) changed how we can
share iPython notebooks online. However, this will require some
additional technology that doesn't exist yet. So, for now, you'll have
to dig into the gist files individually and reconstruct some of the
view of the project by yourselves. This post will help to guide you
through it, in an admittedly inadequate way.

For lack of the appropriate tool chain to present the major stages of
model development within the revision history of a single gist, I
moved over to a regular GitHub project and tagged major stages:

 * [Stage 1](https://gist.github.com/robclewley/0e54e7becf3bb017ea61)
    * [alternative version of Stage 1 on GitHub repo](https://github.com/robclewley/literate-modeling-scenario-test1/tree/Stage1)
 * [Stage 2](https://github.com/robclewley/literate-modeling-scenario-test1/tree/Stage2)
 * [Stage 3](https://github.com/robclewley/literate-modeling-scenario-test1/tree/Stage3)


The primary files to take a look at the beginning are in the gist of
Stage 1:

 * [Scenario-local-linear.yml](https://gist.github.com/robclewley/0e54e7becf3bb017ea61#file-scenario-local-linear-yml)
 * [model.py](https://gist.github.com/robclewley/0e54e7becf3bb017ea61#file-model-py)
 * [setup.yml](https://gist.github.com/robclewley/0e54e7becf3bb017ea61#file-setup-yml)
 * [studycontext1.py](https://gist.github.com/robclewley/0e54e7becf3bb017ea61#file-studycontext1-py)

These already reflect a few commits, partly to assist in creating backups in
case I messed something up. (See later.)

<a name="head3"></a>
## The math modeling problem

Suppose we are given experimental details of an unknown Black Box
containing a nonlinear dynamical system. We are asked to approximate
its dynamics in a region where its behavior is only weakly nonlinear
(phew). Therefore, we *do not* have access to the equations of motion
or other background information, only raw sample data. In the real
world, the sample data could be non-stationary and noisy, but let's
start with stationary (repeated samples over time yield identical
results) and deterministic data.

For simplicity, let's assume that we know we are working in two
dimensions only, and that a suitable sub-domain *D* of two-dimensional
space is given to us. We are given a black box function from which we
can request spatial samples of the vector field *F* (measuring flow at
a location), i.e.

<code>
\\[ F: (x,y) \mapsto ( F_x(x,y), F_y(x,y) ) \\]
</code>

We will find a 2D linear system of ordinary differential equations
(ODEs) that matches this flow as best as possible. Its vector field
will be named *LF*. How well should the match be? We must
be given **User Acceptance Criteria** (UAC): let's assume we're given a maximum
error tolerance *L2_tol* that our model must achieve in the standard Euclidean
(L2) metric *d(x,y)* applied to any sampled vectors in *D* compared to the linear
model.

<code>
\\[ \forall (x,y) \in D, \quad d(x,y) = || F(x,y) - LF(x,y) || < L2tol \\]
</code>

Without getting too technical, if we can make guarantees about the
continuity of *F* and the amount of nonlinearity that it may possess
in *D* (i.e. about its local variance, or bounds on its derivatives),
then it is reasonable to only require that this condition is met on a
finite sample of points in order to guarantee it will work for all
points in *D*. Let's keep this simple for the sake of exposition.

These details constitute a problem **scenario**. It is easy to write
this up in [YAML](http://en.wikipedia.org/wiki/YAML) imagining what a
hypothetical project management program might automatically process
for me. YAML is a simple and great markup language for easily creating
structure out of unstructured text.

~~~
scenario-metadata:
  name: local linear fit
  difficulty: easy
  time: short
  tags: graphical, ODE
 
Given:
 - 2D, weakly nonlinear, determininstic vector field given as a black box function of (x,y) -> (f_x(x,y), f_y(x,y))
 - finite, rectangular domain [x0,x1] X [y0,y1]
 - Euclidean error tolerance of forward trajectories (chosen to be feasible for the given function f, unless task scenario is open to being proven inconsistent)
 
Goal:
  - Derive a UAC-satisfying local linear ODE approximation in the domain (where nonlinearity is fairly weak)
 
UAC:
 - Predict all forward trajectories up to given max tolerance of Euclidean error 
~~~

These seem to be the minimal, necessary elements to make a modeling
problem well posed.

<a name="head3_1"></a>
### Implementation

For the sake of an easy example, I reused code for the classic
[van der Pol](http://en.wikipedia.org/wiki/Van_der_Pol_oscillator)
nonlinear oscillator to create the model in
[model.py](https://gist.github.com/robclewley/0e54e7becf3bb017ea61#file-model-py). Since
the internal details are meant to be hidden from the user, I simulate
this by being clear about what is publically exported from the script
according to the list of givens in the scenario YAML file.

In particular, the exports are:

~~~
__all__ = ['F', 'target_dom', 'compute', 'L2_tol']
~~~

For easy reconfiguration of the test problem, the model setup loads
values for the ODEs' free parameters and domain *D* from `setup.yml`:

~~~
pars:
    eps: 0.1
    a: 0.5
target_dom:
    x: [1, 1.3]
    y: [-0.9, -0.75]
error_tol: 0.002
~~~

----

![Target Vector Field](https://github.com/robclewley/literate-modeling-scenario-test1/raw/Stage2/project/Target_VF_and_test_traj.png "Target Vector Field")

I wrote a couple of sanity check tests and demonstrations in that
module to check that I had everything set up right before attempting
the model development scenario. Those were used to generate the above
diagram of the local, nonlinear vector field in the chosen
sub-domain. The diagram doesn't show arrow heads right now, but they
are pointing to the left. The thick line shows a numerically
integrated trajectory that starts from the center of the domain, with
the individual sample points shown as black dots.

<a name="head4"></a>
## Other project preparations

At this point, it was time to initialize a new Wing IDE python
project in a new folder, and clone the gist I began online into
it. I then configured the Git integration tool within Wing and
synchronized all the files between the local folder and GitHub.

<a name="head5"></a>
## Steps towards a goal: a workflow of test-driven development


<a name="head5_1"></a>
### The 'StudyContext'

For want of a better name (I've considered several), a
**StudyContext** (SC) is what I call the primary unit block of
activity in my workflow. It is meant to represent a frame of
reference, a situation, scene, or setting for studying a given problem
or question. Every SC wraps up a **goal-directed project activity**,
and I organize my file system around the SC tasks towards achieving
whatever goal. The SC goal could be as high level as "learning" or
"discovery" providing the endpoint criteria can be specified in some
way as a UAC. Thus, in some cases, it may be qualitative rather than
quantitative. This follows the same principle as a "sprint" in Agile
software development. Equally, an SC should be chosen to be as
concrete as possible, and achievable in a reasonably short time. If
the problem will take much longer to solve, then multiple SCs are
recommended. A set of unresolved SCs become the project's 'backlog' of
work to be allocated to investigators!

<a name="head5_2"></a>
### Flexible workflow

Since, by the nature of early model development, a lot is unknown, it
makes sense not to enforce a rigid workflow. The only rules that I
need are 1) to set up the goals of an SC before starting, and 2) to
immediately code representations of the goals and the objects needed
to create an executable test of the goals. As with test-driven software
development, the test is expected to fail before the actual model
development work is done. But the existence of the test is an
objective goal to define the UAC of the SC.

<a name="head5_3"></a>
### Open ontology

By the same token, it also makes sense not to limit the user to a
small set of workflow object types. Activities, inputs, and outputs
will vary greatly depending on the problem and the user. Thus, the
ontology for demarking these is kept general and simple.

To get into the right frame of mind, let's quickly recap the
stereotypical structure of an academic mathematics article. There is
an introduction that sets up a problem that will be addressed and its
background context. This provides the 'given' data for the problem and
allows assumptions and definitions to be made. From there on, it's
very context dependent, but we typically see a logical progression of
hypotheses, questions, remarks, calculations, validations, lemmas,
theorems, and their proofs. Hopefully, at the end of the article,
conclusions can be drawn from all of this work that answers the
original question. Unlike most articles in science, a math article
unashamedly uses structured markup throughout the document to allow
the reader to keep track of the beginning and end of each logical
step, and what its role is.

With this presentation model in mind, a SC script should have the
following structure:

 * Metadata header (including a unique StudyContext ID tag)
 * Definitions / declarations
	 - Given data or functions (probably a necessity)
	 - (optional) Assumptions
	 - (optional) Hypothesis, Question, or Need -- if not obvious from
       Goal statement
     - Goal or Target
	 - (optional) Constraints
     - UAC statement
 * (one or more) work **Steps** of various types ending in a Terminal
   Step for the script

*This list is a work in progress. Discussion is welcome.*

Further definitions that are marked 'optional' can be made, where
appropriate, later in the document, but the others must be made once
and at the top of the file.

Thus, the **primary, ongoing project activity** is to iteratively
convert as much of the plain-text definition of the scenario to
executable code blocks associated with each part of the
definition. The success of the code itself (since it is a formal
system) can be a validation of the logic behind the scenario's
original definition.

At the beginning, all we have is the the scenario metadata YAML, which
is pre-pseudo code and hardly a formal specification. As the project
develops, the mapping to the code blocks becomes more concrete as we
lay down declarations of Steps that we will take to achieve the goal,
and write more code to fill in the blocks we need.

<a name="head5_4"></a>
### Example Step types

Types of 'Step' are entirely user-defined, depending on
need. Here are some useful ideas of common types:

 * Validation Test (expect a result): graphic visualization or text output
 * Diagnostic Test (no expectations): graphic visualization or text output
 * Calculation
 * Header-like preamble and boilerplate: package imports, etc.
 * Footer-like clean-up or admin Step at very end of file,
   e.g. matplotlib's `show()` command

In the true CS tradition of "divide and conquer", Steps can be defined
hierarchically, if needed. Steps that turn into research problems of
their own should spawn a new StudyContext to address it (see more
about this below). These should be broken out into sub-folders in the
file system, and linked to from the metadata in the parent document,
and from the header metadata in the child documents back to the
parent. Again, management software should assist in tracking and
validating such links. For now, we'll enforce it as a convention.

<a name="head5_5"></a>
### StudyContext object declaration syntax

As there is no actual project management program visiting and parsing
the SC, there is no need for a formal syntax yet. Using YAML, I have
suggested one way to make the declarations using global-level
triple-quoted comment strings to begin a Step's code block. This is
just a convention that I'm exploring. In the earlier versions, I
referred separately to tests, etc. in the YAML declarations. Later,
they are merged as `step-test`, and so on.  In the files presented
here, I didn't attempt to be consistent with how to specify Step
types.

~~~
"""
studycontext-goal:
  tag: define goal
  notes: functions with richer diagnostic feedback about spatial errors
"""
# python code block here

"""
studycontext-step:
  tag: visualize mesh of errors
  notes:
    - figure 1
"""
# python code block here
~~~


<a name="head5_6"></a>
### Commit the project when you'd commit code

As in a software development project, I committed the project code
whenever I had reached a local "stopping point" in my day's work. For
instance, when I had cleaned up a script that now runs successfullly,
and I'm ready to move on to other tasks; or, when I had finished
refactoring some of the library code and everything is back to normal
in the status quo.

Neglecting some initial commits where I was fiddling about and syncing
with GitHub, the terminal command `git log --oneline -10` produces:

~~~
4e21abf Step 3: fix files created
423b48e Step 3: visualizations part 1
b854606 Step 2: setup linear model spec, goal, and test conditions; verify initial failure
2215d75 Step 1: scenario implementation setup
~~~

Alas, at the time I was doing this, I hadn't yet chosen the term 'Step' to
generically refer to all the different blocks of the SC script. The
numbered 'Steps' in this revision history refer to time checkpoints in
the progression of the project overall. My apologies! This is very
complicated to present as a rapid communication with full disclosure
of my thought process for implementing a first working example.

You can see what changed between commits using commands like:

~~~
git diff b854606 2215d75
~~~


<a name="head5_7"></a>
### Bifurcate the StudyContext when it gets large or complex

During rapid prototyping, I want to use meaningful script file names
and have all my figure output data and model input data in the same
working directory. This avoids the overhead in coding up systematic
access to sub-folders for each of the relevant data types (logs,
parameters, figures, raw data, etc.), and avoids needing to track file
names through formal file system naming and organization rules. While
the argument for *a priori* organization of the local file system is
about avoiding difficulty in search or intuitive comprehension of the
filespace, my alternative solution is that we have 1) great
algorithmic search capabilities locally available on our computers and
2) good visual search of folders if we KISS: "keep it simple, stupid".

The SC file in the gist is a single script. But, by the time I finished
setting it up, I realized that it really was achieving several
different sub-goals, such as diagnostic visualization and initial
testing.

That might not matter a lot, but during debugging or tweaking of
reusable function definitions, I wanted to re-run parts (Steps) of the
script again. I didn't want to re-run other Steps, some of which are
more computationally costly or generate unnecessary output. I have
always found myself commenting out parts of scripts while I work on
other parts. Instead of doing that, I can recognize that some of the
Steps are really Terminal Steps in their own right, and 'parallel' to
each other as elements of the same StudyContext. It's easier to break
the file into many.

One reason I *hadn't* split up files in the past is the
proliferation of boilerplate code that must be run to set up *any*
Step. Copying this into each file is not a good solution. Code should
be re-used *not* duplicated. Thus, the next step was to use an object
cache and a common library convention in order to move reusable
objects and functions to a module from which I can `import *`. A
drawback is that I lose the record of the order in which I created
those Steps in the past, but the revision history (and branch tags,
for instance) at least retains their provenance.

<a name="head6"></a>
## Summary so far

We are nearly half-way done, but we haven't even tackled the
mathematical problem yet. Summarizing so far, you've seen a mock up of
some project metadata attached to code blocks that uses an open
ontology for tracking the major elements of a task. Tasks are executed
by Steps, and organized into common StudyContexts, which are scripts
or folders of scripts, like a python package's organization. We used
Git behind the scenes to commit intuitively meaningful units of work
and to help create the "origin story" of the Steps taken. We have
mentioned a hypothetical project management software package that
would take care of the administration of some of these tasks and
enforcement of the logical operations that modify and track the
project elements through the project's lifetime.

Come back soon!
