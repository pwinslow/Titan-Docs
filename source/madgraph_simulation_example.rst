================================
Parallelized MadGraph Simulation
================================

MadGraph is a framework that aims at providing all the elements necessary for SM and BSM phenomenology, such as the computations of cross sections, the generation of hard events and their matching with event generators, and the use of a variety of tools relevant to event manipulation and analysis. Processes can be simulated to LO accuracy for any user-defined Lagrangian. These user-defined models are usually generated through the FeynRules package, which you can read about and download `here <https://feynrules.irmp.ucl.ac.be/>`_. For more information on MadGraph, see the `launchpad <https://launchpad.net/mg5amcnlo>`_ site. In this tutorial, we'll be using MadGraph to generate collision events for a specific and then we'll install and use Pythia and Delphes to shower, hadronize, and simulate the response of a detector like CMS or ATLAS. 

-------------
Preliminaries
-------------

In this tutorial, we'll walk through a Monte Carlo simulation of the parameter space for a BSM model of lepton number violation at 100 TeV using the titan cluster.