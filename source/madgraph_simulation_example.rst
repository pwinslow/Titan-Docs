================================
Parallelized MadGraph Simulation
================================

MadGraph is a framework that aims at providing all the elements necessary for SM and BSM phenomenology, such as the computations of cross sections, the generation of hard events and their matching with event generators, and the use of a variety of tools relevant to event manipulation and analysis. Collider processes can be simulated to LO accuracy for any user-defined Lagrangian. These user-defined models are usually generated through the FeynRules package, which you can read about and download `here <https://feynrules.irmp.ucl.ac.be/>`_. For more information on MadGraph, see the `launchpad <https://launchpad.net/mg5amcnlo>`_ site. 

-------------
Preliminaries
-------------

In this tutorial, we'll be using MadGraph to generate collision events for a specific process and then Pythia and Delphes to shower, hadronize, and simulate the response of an actual detector. We'll use the titan cluster to repeat this process a number of times in order to sample the model parameter space for a BSM model. The example BSM model we'll use consists of a single Majorana fermion and a single scalar with U(1)\ :sub:`EM`\  charge +1. The Majorana nature of the fermion induces lepton number violation (LNV) in certain processes. At high energy, this LNV manifests as same-sign lepton production. It is this process which we'll be interested in simulating and we'll specifically be interested in simulating these processes using the estimated specs for a proposed detector at a 100 TeV future circular hadron collider. In order to simulate the response of a detector at a 100 TeV collider, we'll use a tcl script which tells Delphes the specs of the detector it should simulate. The specific tcl script we'll use borrows heavily from a script created by Heather Gray and Filip Moortgat for FCC detector studies.

-------------
Installations
-------------

To get started, we need to setup root. To do this, log into titan and type ``setupATLAS`` and, once this is done running, type ``localSetupROOT``. Now that root is available, cd into the ``local`` folder, open a text editor, paste in the following script ::

	#!/bin/bash

	echo ''
	echo 'Downloading MadGraph...'
	echo ''
	wget -P $HOME/local https://launchpad.net/mg5amcnlo/2.0/2.4.x/+download/MG5_aMC_v2.4.3.tar.gz
	cd $HOME/local
	tar -xzf MG5_aMC_v2.4.3.tar.gz
	sed -i -e 's/# auto_update = 7/auto_update = 0/g' MG5_aMC_v2_4_3/input/mg5_configuration.txt
	sed -i -e 's/# automatic_html_opening = True/automatic_html_opening = False/g' MG5_aMC_v2_4_3/input/mg5_configuration.txt
	rm MG5_aMC_v2.4.3.tar.gz

	echo ''
	echo 'Downloading LNV model...'
	echo ''
	git clone https://github.com/pwinslow/Titan-Docs.git
	mv Titan-Docs/LNVatLHC_UFO MG5_aMC_v2_4_3/models
	rm -rf Titan-Docs

	echo ''
	echo 'Downloading Pythia...'
	echo ''
	wget -O $HOME/local/MG5_aMC_v2_4_3/pythia-pgs.tar.gz http://madgraph.phys.ucl.ac.be/Downloads/pythia-pgs_V2.4.5.tar.gz
	cd $HOME/local/MG5_aMC_v2_4_3/
	tar xzf pythia-pgs.tar.gz
	rm pythia-pgs.tar.gz
	cd pythia-pgs/src
	echo ''
	echo 'Building Pythia. This may take a little while...'
	echo ''
	make all

	echo ''
	echo 'Downloading Delphes...'
	echo ''
	cd $HOME/local
	git clone https://github.com/delphes/delphes.git
	echo ''
	echo 'Building Delphes. This may take a little while...'
	echo ''
	cd delphes
	make
	echo ''
	echo 'Finished installing Delphes...'
	
	echo 'Downloading 100 TeV delphes detector simulation card...'
	echo ''
	wget -P $HOME/local/delphes/cards https://raw.githubusercontent.com/pwinslow/Machine-Learning-Projects/master/Lepton-Number-Violation-at-100-TeV/Data_Production/PW_FCC_delphes_card.tcl
	
	echo ''
	echo 'Finished MadGraph+Pythia+Delphes setup...'

save it as ``MGsetup.sh``, make it an executable with ``chmod +x MGsetup.sh``, and run it with ``./MGsetup.sh``. This script will first download a fresh copy of MadGraph and unpack it. It then will download the FeynRules-generated BSM model we'll work with and place it in the ``models`` folder so we can use it. After this, it will download and install pythia in the main ``MG5_aMC_v2_4_3`` folder and then install delphes in the ``local`` folder. Finally, it will download the specially designed tcl script mentioned above to simulate a detector in a 100 TeV hadron collider and place it in the delphes cards folder. 

To test our installation, open the MadGraph interface in the ``MG5_aMC_v2_4_3`` folder and type ``./bin/mg5_aMC``. Once in, type in the following cmds one-by-one ::

	import model LNVatLHC_UFO
	generate p p > sp f0 f0 @1
	add process p p > sp f0 f0 j @2
	output testrun
	exit

Once you've exited the MadGraph interface, cd into the ``testrun`` folder and create a Pythia card with ``cp Cards/pythia_card_default.dat Cards/pythia_card.dat``. We can now perform a simulation of the hard scattering, showering, and hadronization of our process by modifying the collider energies of both beams to be 50000.0 GeV in the ``Cards/run_card.dat`` file and then running the cmd ``./bin/generate_events 0 testrun_1``. This should generate parton-level events and shower and hadronize the results with Pythia. To simulate the effects of a 100 TeV detector, we'll run Delphes on the ``tag_1_pythia_events.hep.gz`` file in the ``Events/testrun_1/`` folder with the following cmds, which should be run one-by-one from the ``testrun`` folder ::

	gunzip Events/testrun_1/tag_1_pythia_events.hep.gz
	../../delphes/DelphesSTDHEP ../../delphes/cards/PW_FCC_delphes_card.tcl Events/testrun_1/delphes_output.root Events/testrun_1/tag_1_pythia_events.hep
	../../delphes/root2lhco Events/testrun_1/delphes_output.root Events/testrun_1/delphes_output.lhco

The result of these cmds will be the creation of ``tag_1_delphes_events.root`` and ``tag_1_delphes_events.lhco`` files in the ``Events/testrun_1`` folder, which contain the final form of all the simulated events.

----------------------------------------------
Setting up for Partial Parallelized Simulation
----------------------------------------------

In this section, we'll take the process laid out in the last section and run it on the titan cluster for a single parameter point.



.. [#] MadGraph may ask if you want to install a newer version. Type ``n`` to keep working with the downloaded version, the newest MadGraph hasn't been tested for this tutorial.