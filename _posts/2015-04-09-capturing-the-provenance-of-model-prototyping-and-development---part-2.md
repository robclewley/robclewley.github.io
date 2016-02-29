---
layout: post
title: "Capturing the provenance of model prototyping and development - Part 2"
comments: true
tags: [technical, test-driven-modeling, literate-modeling, python]
redirect_from: ""
permalink: /2015/04/09/capturing-the-provenance-of-model-prototyping-and-development---part-2
references:
---

{% assign ref=page.references %}

**Table of Contents**

- [Steps towards the goal, continued](#head1)
  - [Breaking into Terminal Steps](#head1_1)
  - [When Steps need to be decomposed further](#head1_2)
- [Completing the model development](#head2)
  - [Recap of the linear model](#head2_1)
  - [Major repository changes](#head2_2)
  - [Optimization step](#head2_3)
  - [Visualizations of the results](#head2_4)
- [Conclusions and discussion](#head3)
  - [Summary](#head3_1)
  - [Limitations of the implementation](#head3_2)
  - [Weaving a tangled web of unreadability](#head3_3)
  - [iPython notebooks to the rescue?](#head3_4)
  - [Version control issues](#head3_5)
  - [Further Work](#head3_6)

- [Footnotes](#head4)

------

This is Part 2 of my posts about capturing the provenance of model
prototyping and
development. [Part 1 is here]({% post_url 2015-04-06-capturing-the-provenance-of-model-prototyping-and-development %}).

----

<a name="head1"></a>

## Steps towards the goal, continued

<a name="head1_1"></a>

### Breaking into Terminal Steps

At this point, we switch to the
[GitHub repo](https://github.com/robclewley/literate-modeling-scenario-test1/tree/Stage2),
where I separated the StudyContext (SC) into multiple files describing
Terminal Steps, which now populate a folder named `studycontext1/`
instead of a single file[^1]. This ensures that each file has one
primary purpose. I also added a common library and made the whole file
space into a python package to allow local imports to work. These are
jobs that the hypothetical project manager would do for me.

If you cloned the whole repo locally, you'll have to reset the
project's state to the initial tag for this point in the development:

~~~
git reset --hard "Stage2"
~~~

We now have this file system tree:

~~~
 - project/
   - __init__.py
   - cache_test.py
   - cacher.py
   - common.py    # unused
   - model.py
   - setup.yml
   - studycontext1/
      - __init__.py
	  - common.py
	  - initial_lin_viz.py
	  - initial_test.py
	  - viz_errors1.png
	  - viz_VF_overlay.png
   - Target_VF_and_test_traj.png
~~~
 
We have two study context files with the following Terminal Steps:

  * `initial_test.py`

~~~
studycontext-test:
  tag: initial test that goal fails with a small sample
~~~

  * `initial_lin_viz.py`

~~~
studycontext-step:
  tag: visualize mesh of errors
  notes:
    - figure 1
	
studycontext-step:
  tag: visualize mesh of linear vectors overlaid with actual vectors
  notes:
    - figure 2
~~~

As mentioned before, the `initial_test` script just validates that the
UAC fails, and that the goal test code is set up correctly, before we
begin solving the actual problem.

----

The visualizations produce the following figures:

<a name="Fig1"></a>
 * Figure 1: Initial linear VF overlay (compare [Figure 3](#Fig3))
 
![Initial linear VF overlay](https://github.com/robclewley/literate-modeling-scenario-test1/raw/Stage2/project/studycontext1/viz_VF_overlay1.png)

This figure overlays a 5 x 5 sample of vectors from F (pointing left,
as we saw before) with those from LF (pointing up, arrowheads not shown!).

----

<a name="Fig2"></a>
 * Figure 2: Initial error measure (compare [Figure 4](#Fig4))
 
![Initial error measure](https://github.com/robclewley/literate-modeling-scenario-test1/raw/Stage2/project/studycontext1/viz_errors1.png)

This figure shows a 30 x 30 mesh of absolute errors for the
metric *d(x,y)*, which is implemented as `common.py/metric(x,y)`. The
blob size is proportional to the error magnitude.

<a name="head1_2"></a>

### When Steps need to be decomposed further

The tasks in scope of a StudyContext and its respective input and
output data should be well chosen to involve only a modest number of
files that are easy to manage in one folder. If a task of the SC is
later understood to require much more setup, work steps, or additional
files, then my convention is to make a sub-StudyContext in a new
sub-folder and work in there. If files and data need to be inherited,
then make it a python sub-package and import from the parent's common
library.

The markup of the SC files should indicate where a Step in the parent
SC leads to work in a named sub-SC that probably corresponds to the
sub-folder's name. In lieu of a formal project management system, this
user-enforced rule can work for small sized projects. This idea is not
demonstrated here: I will exemplify it in a future post.

<a name="head2"></a>

## Completing the model development

<a name="head2_1"></a>

### Recap of the linear model

A 2D linear model requires 6 parameters to specify it. In simplest
terms, it would look like:

~~~
dx/dt = a*x + b*y + c
dy/dt = d*x + e*y + f
~~~

However, with a tiny bit of algebra, this can be rewritten in a
convenient form that accentuates the timescales associated with each
variable:

~~~
dx/dt = (x0+yfx*y - x)/taux
dy/dt = (y0+xfy*x - y)/tauy
~~~

This ODE system is implemented in the project as the `PyDSTool.Model`
object `lin`.

<a name="head2_2"></a>

### Major repository changes

We can now switch to
[Stage 3](https://github.com/robclewley/literate-modeling-scenario-test1/tree/Stage3),
which is the HEAD of the GitHub repo, and where the development has
ended.

~~~
git reset HEAD
~~~


We now have this file system tree [^2]:

~~~
 - project/
   - __init__.py
   - cache_test.py
   - cacher.py
   - common.py    # unused
   - model.py
   - setup.yml
   - studycontext1/
      - __init__.py
	  - check_linearization.py
	  - common.py
	  - final_solution_viz.py
	  - initial_lin_viz.py
	  - initial_test.py
	  - optimize.py
	  - viz_errors1.png
	  - viz_VF_overlay1.png
	  - viz_VF_overlay2.png
   - Target_VF_and_test_traj.png
~~~

`optimize.py` is the additional `studycontext1` script that sets up an
optimization process.

<a name="head2_3"></a>

### Optimization step

In order that each script does just one task, some data must be
accrued in the project that can be accessed between scripts. This
involves ['serializing'](http://en.wikipedia.org/wiki/Serialization)
or 'marshalling' data and code objects to the file system for future
use. The [`pyfscache`](https://github.com/robclewley/pyfscache)
package achieves this in a very convenient way, while also avoiding
inefficient recalculation of identical objects that have already been
computed once before.

The prior dependencies of these objects are noted in the metadata for
a SC comment string.

We use the `error_pts` function, and the meshes we set up in
Stage 1, to form the residual error function for a minimization
routine:

~~~
def residual_fn(p):
    lin.set(pars=dict(zip(parnames,p)))
    # fmin will take the norm for us
    return error_pts(mesh_pts_test5)
~~~

This function takes a sequence of parameter values and returns the
vector of the errors at all the mesh sample points.

We create two constraints to see if we can ensure that the two
timescale parameters (`taux` and `tauy`) are both positive
numbers. [^3]

Then, we run the optimizer (`fmin_cobyla`, which can accept simple
inequality constraints). Fortunately, we quickly converge to a useful
solution. If not, we could have created a sub-SC to diagnose and
explore the best parameter choices to make the algorithm
perform. Sometimes, no progress is made at all in a complex
optimization problem without learning some 'magic' parameter
values. Documenting this is not fun, as it is already a tiresome task
to have to tweak an algorithm's parameters in a nonlinear problem.

But, it's also not fun to explain to someone (e.g., your PI/colleague)
how you arrived at a set of choices, and which ones you tried out. You
could write them out in a notebook but you could also commit each
trial (both input and output) in the VCS with one quick keystroke as
you work, and keep all that provenance inherently digital and
alongside the project code.

Furthermore, if you used any cunning logic to figure out the parameter
choices, you might like to markup a discussion of what you did and
share your provenance record with others to help educate them.

<a name="head2_4"></a>

### Visualizations of the results

In Stage 2, we observed that the default state of the linear system's
vector field is almost perpendicular to that of the target vector
field. In comparison, the visualizations of the "optimized" state,
shown below, indicate a great subjective improvement in the alignment
of the vectors and a reduction in the error metric:

<a name="Fig3"></a>
 * Figure 3: Final linear VF overlay (compare [Figure 1](#Fig1))
 
![Final linear VF overlay](https://github.com/robclewley/literate-modeling-scenario-test1/raw/master/project/studycontext1/viz_VF_overlay2.png)

----

<a name="Fig4"></a>
 * Figure 4: Final error measure (compare [Figure 2](#Fig2))
 
![Final error measure](https://github.com/robclewley/literate-modeling-scenario-test1/raw/master/project/studycontext1/viz_errors2.png)

----

The visualizations are grouped into one script, and of course they
depend on previous scripts having been called. Again, these are noted
in the SC comment metadata.

We have not addressed the original UAC, which is to ensure that *trajectories* computed from LF do not diverge more than *L2_tol* from those from F, but we will see that it's not even necessary to assess that to make a conclusion.

<a name="head3"></a>

## Conclusions and discussion

Based on the original error tolerance requested (*L2_tol* = 0.002),
**the scenario appears to be unfeasible** just from looking at the vector field itself!
We achieved a visually similar-fitting vector field with a linear system, but the maximum
error in the vectors across the mesh sample was still 1.0. There is no way such an error will satisfy the
UAC for trajectories across this domain, which will only be compounded given the local vector error.

Without turning this
demonstration into a protracted interlude on numerical analysis, you
can verify that the optimization algorithm had converged to a solution
and was not going to make any large enough improvements to get the
error down another order of magnitude. It is, of course, possible that
we are getting trapped in a local minimum, and ideally we would create
some further tasks to explore this: e.g., repeat over randomized
initial conditions for the optimization in a simple Monte-Carlo
sampling, or perform simulated annealing. We could also have tried to
determine appropriate rescalings of each parameter based on the
sensitivity of the residual error vectors, and provided this rescaling
to the optimizer to help it choose step sizes more effectively.

We may also wish to close this SC and setup a new one that changes the
permitted steps in order to really achieve the requested
tolerance. This could involve sub-dividing the domain further and
using piecewise linear models to fit in each sub-domain.

These ideas are beyond the scope of this demonstration, but it might
be worth revisiting them if the extension of this demo will highlight
more sophisticated and realistic StudyContext usage.

Admittedly, there was nothing so interesting about the linear model we
derived that deserved such detailed attention. It was a pedagogically
convenient example with no surprises along the way and no difficult
choices to explore and resolve. If there were, *and there would be in
more realistic problems*, I argue that this methodology would better
facilitate the planning, organization, execution, and documentation of
work to be done to address those choices and problems.

Later, I will introduce more realistic worked examples that display
such properties. In the meantime, I hope this minimal demonstration
initiates some discussion and maybe even a contribution to creating a
python platform for supporting literate modeling of this kind.

<a name="head3_1"></a>

### Summary

These two posts have been an almost stream-of-consciousness look at a
way to capture the provenance of mathematical model prototyping. We
set out a need, a problem space, and a task, and set up an organized
set of files and workflow conventions to document how we arrived,
piece by piece, to our conclusion. We did derive a *visually* reasonable
linear model for the nonlinear system in the given domain. 

I really don't think it's as difficult to imagine doing literate
modeling at this early model development stage as I've heard people
lament. Although, I haven't demonstrated how to capture aspects of the
discovery and learning process that involve more graphical
interactions. I will return to this.

The essence of my approach to literate modeling is to treat model
development steps as the kinds of task work done during software
development, in particular to apply concepts such as test-driven
development, version control, and highly iterative workflows such as
that espoused in the Agile approach. The version control aspect is
inspired, in part, by previous work on the Sumatra simulation
provenance tool, and simulation execution workflow design tools such
as Taverna and Kepler.

By establishing a simple convention about markup for model development
steps and file system structure, I hope to have made a convincing
argument that such prototyping and early development can realistically
be captured and organized in a useful way.

Note that we could apply this provenance methodology to other forms of
mathematical or computational work, including data analysis.

<a name="head3_2"></a>

### Limitations of the implementation

Several limitations and problems that I encountered are recorded as
comments on the gist github page, but expanded upon here.

Obviously, I need scripts or a proper project management tool to automate git
add/commit/push with automated message headers in the commit
message. Right now, it's all being done by hand, and it's difficult to
keep track of my self-imposed conventions.

How do I add math markup in the documentation of the goals, etc., in a
way that will be transformed to readable math in the browser (online,
I know how to achieve it on localhost)? I *think* this can be done
with github pages and jekyll, but I'm not sure which third party
extension to invoke yet.

What to do when functions defined during development of
`studycontext1.py` need to be pushed to a common library to enable
reuse? How does the provenance markup using the triple-quoted strings
(or any other solution), which shows the order of sub-tasks (Steps) in
that file get affected? Well, for now, a solution is to leave it to
the historical record in the git revision history. But it's not an
entirely satisfactory solution.

Another fundamental problem is, when working interactively, how do we
interpret the SC Step declarations that may be repeated (even out of
order) during pasted-code execution? A simple way to resolve this is
to only retain the changes to the (hypothetical) project manager's
database state at commit time, whenever we are ready to run a SC
script once through and move on.

Another issue is that there is no obvious way to mark when a
StudyContext is complete, except to leave a note in the document or
use a commit message. Presumably, the project management software
would keep metadata in special files to track progress just like
project management tools for software development.

Naturally, though, it makes sense that there should be an executable
script that runs a test on whatever solution to check the UAC have
passed. This is a long-winded way to tell if the SC's work is
complete. The code may still need to be tidied up before committing
and publishing.

<a name="head3_3"></a>

### Weaving a tangled web of unreadability

Finally, as Tony Fast pointed out in our discussion, python code files
are meant for, er, code, and also for direct documentation for
code. They really are not the place for storing metadata for other
systems. I agree, but separating the metadata from the code makes it
more difficult to track where code blocks begin an end as part of Steps.

But, it's true. Triple-quote docstrings belong at the top of the file
and inside function definitions, and I end up repeating some of that
documentation outside for the studycontext metadata.  And I don't want
to keep track of indenting exactly the same amount for the next note
as part of the same Step, several code lines below where it started in
the previous comment (e.g. see line 86 of the gist's
[studycontext1.py](https://gist.github.com/robclewley/0e54e7becf3bb017ea61#file-studycontext1-py)).
It looks *weird*.

The literate programming community have provided some tools, but I am
not excited about working with a setup like
[WEB](http://en.wikipedia.org/wiki/WEB) -- via
[noweb](https://www.cs.tufts.edu/~nr/noweb/), for instance, in which
you write markup for the metadata and put code into special markup tag
environments. On one hand, it's good that code and metadata are all
stored together, but that is not a productive way to write exploratory
code. On most editors, even with noweb markup modes, you'll lose the
code's native syntax highlighting and other editing conveniences
(e.g., auto-indenting blocks). Then, the file is not executable (i.e.,
even testable) until it has been 'tangled'. For my personal view on
rapid development, this situation is inconvenient and
backwards. I want to get code snippets written down and tested ASAP to
build up the pieces I'll need to work on the problem. *Then*, I'll
generate the documented version from the embedded notes and present
the results. I want to keep notes in comments to the side of the
immediately-executable code. Maybe a custom editor or IDE would solve
this problem.

A project management system's GUI editor could use templates to
insert the bulk of the strings for each Step declaration, and precede
them with some kind of coded directive to indicate that the string is
not a regular docstring or comment.

A possible alternative is that, instead of using comment strings
for each Step declaration, there are method calls on a global
"Scenario" object that provides access to the project manager's
API. E.g.:

~~~
Scenario.Goal()
# code block here

Scenario.Step(type='test', tag='check initial model state',
    notes=['use 5x5 grid for now', 'needs optimizing for speed'])
# code block here
~~~

Then, the metadata is made part of the code from the beginning.

What the calls would do: Write a YAML file entry in sequence,
essentially building the docstrings I wrote in. These can be diff'ed
to check for structural changes. The presence of the call in the
source code allows a pre- or post-processor to select the
computational python code to put in a JSON block (like ipynb).

Pasting the SC calls into an interactive script should be innocuous
because the calls may repeat and/or be out of order. We only care
about the results from running the whole script. This is actually easy
to imagine taking care of, because the post-processor would only act
on the ordered declarations when invoked at the end of a script. The
YAML file being formed can safely get messed up during the interaction
session but won't be "committed" until after the whole, completed
script is run to the endpoint. Each run rewrites the whole YAML
record for that script.

An interestingly similar tool concept is
[Watson](http://nhmood.github.io/watson-ruby/), a software development
issue manager that is based on inline code markup. If a python version
matures ([this one](https://github.com/mleonard87/watson-python) is
not there yet), I'd consider basing my project manager on this code.

<a name="head3_4"></a>

### iPython notebooks to the rescue?

However, iPython notebooks, as I currently appreciate them, are also
*not* the solution to this problem of mixing StudyContext metadata and
code. Notebooks do well at separating text and other data formats from
code, and presenting their combination in a clean and pretty way. This
makes a notebook a great presentation tool for a finalized, static
version of the code and output (including intermediate states), but
not for active, rapid development. A notebook is like the result of a
*weave* process.

On the other hand, Tony mentioned that I might 'magic parse' the
SC's YAML metadata on execute in a notebook using
[this fancy snippet](https://gist.github.com/bollwyvl/aa7131d57195f86fb9c0),
although I have yet to understand if that will help or even how to do
it. (There is no documentation provided with that gist.)

Regardless, Ipython notebooks seem inappropriate for my use if I want
to explore interactively in the command line, or especially in a
graphical UI (I don't just mean resizing or zooming plots). I often do
not want my graphs to show up inline, and there may be too many to
conveniently manage in a single window as sub-plots. I may also want
to structure my file system around a local library of common elements
and different scripts for different tasks. These are not so easy to
present as a single notebook document. When I discuss interactive
tools more in future posts, and I learn more about the notebooks, my
arguments will hopefully become clearer one way or another.

<a name="head3_5"></a>

### Version control issues

In terms of version control, the gist system is not sophisticated
enough to support all the present needs. The git history is not as
easily viewable as a regular git project on GitHub (e.g., no network
diagram) as there is no GUI for gist revision history in the same
way. This means that git tags and branches are not as easy to discover
or navigate with online GUI tools. It's not clear to me how they could be
usefully applied to tracking this mini project. A proper model project
manager would have to wrap all the git operations and UI into a
customized interface.

<a name="head3_6"></a>

### Further work

Merging and combining results and artifacts of sub-SCs that resolve
back into the parent SCs to allow them to continue. This poses a data
import problem. For now, we must use a convention. Having committed
the state of the project at the resolution of the sub-SCs, the new
artifacts can be tagged by their original SC file and then merged into
the parent's common library for continued use in the parent SC.

<a name="head4"></a>

### Footnotes

[^1]: We won't bother to record the structure of the study contexts in a metadata file in this example, but a real project management system would be keeping track of the change from a single `studycontext1.py` to the folder `studycontext1/` and a database of all project SCs.

[^2]: Note that the `check_linearization.py` script is a meta-level check for me to verify that the linearization made sense in the context of the original model. This isn't part of the main workflow.

[^3]: They needn't be in the general case, and this is an artificial addition here, although it doesn't affect the result.
