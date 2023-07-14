---
layout: page
title: "Lesson Design"
permalink: /design/
---

## Outline

* Introduction
    * Overview of Information Flow in GROMACS
        * flow-chart similar to the one in the AMBER lesson
    * A quick look at important file formats used by GROMACS
        * PDB
        * GRO
        * Topology
        * MDP
        * XVG
    * Phases of MD-experiment (or should this go into extra-topics?)
        * Planning
        * System preparation
        * Relaxation
        * Equilibration
        * Production
        * post-processing
        * Analysis
* *BREAK*
* System: Protein in Water
    * using pdb2gmx
* *BREAK*
* System: Mixture of organic molecules
    * use Antechamber/acpype for topologies
    * 
* *BREAK*
* System: Bi-Phasic system
    * using `gmx solvate`
    * using `editconf` to shift and then combine files


