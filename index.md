---
layout: lesson
root: .  # Is the only page that doesn't follow the pattern /:path/index.html
permalink: index.html  # Is the only page that doesn't follow the pattern /:path/index.html
---

This is the workshop material for a Molecular Dynamics course that is being developed and delivered by the Molecular Modelling and Simulation Team
of the Digital Research Alliance of Canada.

<!-- this is an html comment -->

{% comment %} This is a comment in Liquid {% endcomment %}

> ## Prerequisites
>
> This is a hands-on lesson using [GROMACS](https://gromacs.org)
> for running molecular dynamics simulations.
>
> Familiarity with terms and concepts of Molecular Mechanics (MM), Newtonian
> mechanics, Force Fields and statistical ensembles are expected.  
> Our [Molecular Dynamics Theory lesson][molmodsim-md-theory-lesson] covers
> all these theoretical prerequisites.
>
> In addition, this lesson expects familiarity with the Linux command line.
> A great way to learn this is the [Software Carpentry][{{ site.swc_site }}] 
> lesson [The Unix Shell]({{site.swc_pages}}/shell-novice/).
> Using the Linux command-line is also a common component
> of the introductory workshops for using Alliance systems that are delivered
> by the regional consortia.
{: .prereq}

{% include links.md %}

[molmodsim-md-theory-lesson]: https://computecanada.github.io/molmodsim-md-theory-lesson-novice