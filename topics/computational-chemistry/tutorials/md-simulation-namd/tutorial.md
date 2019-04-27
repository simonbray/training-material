---
layout: tutorial_hands_on

title: Running molecular dynamics simulations using NAMD
zenodo_link: ''
questions:
- How do I use the NAMD engine in Galaxy?
- What is the correct procedure for performing a simple molecular dynamics simulation of a protein?
objectives:
- Learn about the NAMD engine provided in BRIDGE
requirements:
  -
    type: "internal"
    topic_name: computational-chemistry
    tutorials:
      - setting-up-molecular-systems
time_estimation: 3H
key_points:
- Several MD engines are available in BRIDGE
- Workflows are available for common simulations tasks such as equilibration and production dynamics for various ensembles (NVT, NPT)
- You've run an equilibrium and production MD simulation using NAMD
contributors:
- chrisbarnettster
- tsenapathi
- simonbray

---


# Introduction
{:.no_toc}


> ### Agenda
>
> In this tutorial, we will cover:
>
> 1. TOC
> {:toc}
>
{: .agenda}


In this tutorial we will perform a simulation with the popular [NAMD](http://www.ks.uiuc.edu/Research/namd/) molecular dynamics software.

This tutorial is made up of two parts. In the first section, we will look at preparation of a system (solvation, charge neutralisation, energy minimisation) using CHARMM. In the second section, we will perform an equilibration and production simulation, using NAMD. If you already completed the [Setting up molecular systems](../setting-up-molecular-systems/tutorial.html) tutorial, which covers the use of the CHARMM graphical user interface (GUI), you have already prepared your system, so go straight to the [second section](#namd-md-workflow), using the files you prepared earlier.

The process can be accomplished by selecting each tool from the tools menu, or by importing the workflow. The workflow method is most efficient and the individual tools used in the workflows are discussed below. The entire workflow (preparation + simulation) is shown below for [CHARMM and NAMD](#figure-1).



![Snapshot of CHARMM and NAMD analysis workflow](images/NAMD_CHARMMGUI_workflow.png"A simple simulation workflow starting with CHARMM for setup and NAMD to continue the production simulation")



> ### {% icon details %} NVT, NPT and statistical mechanics theory
>
> [See Statistical Mechanics, McQuarrie for more in depth theory ISBN:9781891389153](https://books.google.co.za/books/about/Statistical_Mechanics.html?id=itcpPnDnJM0C&redir_esc=y)
>
{: .details}

# System preparation with CHARMM

If you already prepared your system using the CHARMM-GUI, and saved the output files, you can skip this section.

## Setup

Initially, we need to prepare a protein-ligand system in CHARMM. 

This tool will:
- solvate the protein-ligand complex, using the TIP3P water model
- neutralise the system, using 0.05M NaCl
- conduct a short energy minimisation 

> ### {% icon hands_on %} Hands-on:
>
> Run **setup** {% icon tool %} with the following parameters:
>    - {% icon param-file %} *"psf input"*: `???` (Input dataset)
>    - {% icon param-file %} *"crd input"*: `???` (Input dataset)
>    - *"Buffer"*: `???` ()
>    - *"Custom topology and parameter files for ligands"*: `???` ()
>
{: .hands_on}

### Energy minimisation

The setup provides us with ??? files, including one which defines the parameters of the unit cell. Now, we need to perform energy minimisation. This relaxes the system, removing any steric clashes or unusual geometry which could artificially raise the energy.

This tools will:
- Minimise energy using a steepest descent algorithm followed by Adopted Newton Raphson (using the defined number of steps)
- Set up periodic boundaries and generate Particle Mesh Ewald (PME)
- generate reference structures for restraints in NAMD (if selected)

> ### {% icon hands_on %} Hands-on:
>
> Run **minimizer** {% icon tool %} with the following parameters:
>    - {% icon param-file %} *"system_setup_crd"*: `???` (Input dataset)
>    - {% icon param-file %} *"system_setup_psf"*: `???` (Input dataset)
>    - {% icon param-file %} *"waterbox parameters input"*: `???` (Input dataset)
>    - *"Minimization steps"*: `???` ()
>    - *"Create reference structures for RMSD restraints for NAMD?"*: `???` ()
>    - *"Custom topology and parameter files for ligands"*: `???` ()
>
{: .hands_on}


# NAMD MD workflow

At this point we are ready to run the simulation workflow, which uses NAMD as a molecular dynamics engine. An NVT simulation is followed by an NPT simulation.

![Snapshot of NAMD analysis workflow](images/NAMD_workflow.png "A simple NAMD simulation workflow")



> ### {% icon hands_on %} Hands-on: Access the workflow
> Access the published workflows
> ![List of published workflows](images/published_workflows_update.png "Published workflows")
> Choose to import the NAMD MD workflow from published workflows
> ![Import workflow](images/import_workflow.png "Import workflow")
> Choose to run a workflow from your available workflows
> ![Your workflows](images/workflows_yours.png "Your workflows")
{: .hands_on}



### NVT

Classical NVT dynamics, maintaining constant **n**umber of particles, **v**olume and **t**emperature.

This tool runs classical molecular dynamics simulations in NAMD using an NVT ensemble. User can run the simulation in small time intervals. The coordinates, velocities and the extended system files can be use to restart the simulations. If required, harmonic restraints can be used to maintain the protein shape. These restraints, in particular RMSD harmonic restraints can be added with the NAMD collective variable module.

> ### {% icon hands_on %} Hands-on: NVT dynamics
>
> 1. **namd_nvt** {% icon tool %} with the following parameters:
>    - {% icon param-file %} *"xplor psf input"*: PSF file from CHARMM preparation (Input dataset)
>    - {% icon param-file %} *"pdb input"*: PDB file from CHARMM preparation (Input dataset)
>    - {% icon param-file %} *"PME grid specs"*: ??? (Input dataset)
>    - {% icon param-file %} *"waterbox prm input"*: ??? (Input dataset)
>    - *"Temperature (K)"*: `300`
>    - *"Are you restarting a simulation?"*: `No`
>    - *"Use Harmonic restraints simulation?"*: `No`
>    - *"Custom parameter file for ligands"*: `No`
>    - *"DCD Frequency (ps)"*: `1` (Frequency to record frames in the DCD trajectory)
>    - *"Simulation Time (ps)"*: `10`
>    - *"Number of processors"*: `4`
>
> ![Snapshot of NAMD NVT tool parameters](images/namd_nvt_tool_params.png "NAMD NVT parameters")
{: .hands_on}


### NPT
Classical NPT dynamics, maintaining constant **n**umber of particles, **p**ressure and **t**emperature.

This tool runs classical molecular dynamics simulations in NAMD using an NPT ensemble. User can run the simulation in small time intervals. The coordinates, velocities and the extended system files can be use to restart the simulations. Harmonic restraints can be used. NAMD collective variable module is used to give RMSD harmonic restraints.

> ### {% icon hands_on %} Hands-on: NPT dynamics
>
> 1. **namd_npt** {% icon tool %} with the following parameters:
>    - {% icon param-file %} *"xplor psf input"*: PSF file from CHARMM preparation (Input dataset)
>    - {% icon param-file %} *"pdb input"*: PDB file from CHARMM preparation (Input dataset)
>    - {% icon param-file %} *"PME grid specs"*: ??? (Input dataset)
>    - {% icon param-file %} *"waterbox prm input"*: ??? (Input dataset)
>    - *"Temperature (K)"*: `300`
>    - *"Pressure (bar)"*: `1.01325`
>    - *"Are you restarting a simulation?"*: `Yes`
>    - *"Coordinates from the previous simulation"*:  from the NVT simulation
>    - *"Velocities from the previous simulation"*:  from the NVT simulation
>    - *"Extended system of the previous simulation"*: from the NVT simulation
>    - *"Use Harmonic restraints simulation?"*: `No`
>    - *"Custom parameter file for ligands"*: `No`
>    - *"DCD Frequency (ps)"*: `1` (Frequency to record frames in the DCD trajectory)
>    - *"Simulation Time (ps)"*: `15`
>    - *"Number of processors"*: `4`
> 
> ![Snapshot of NAMD NPT tool parameters](images/namd_npt_tool_params.png "NAMD NPT parameters")
{: .hands_on}


# Conclusion
After completing the steps, or running the workflow, we have successfully produced a trajectory (the xtc file) which describes the atomic motion of the system. This can be viewed using molecular visualization software or analysed further; please visit the visualization and [analysis](../analysis-md-simulations/tutorial.html) tutorials for more information.

{:.no_toc}

