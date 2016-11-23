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

To get started, we'll first install MadGraph and Pythia. Once this is done, we'll have to set up root and then install Delphes. In order to set up MadGraph and Pythia, log into titan, cd into the ``local`` folder, open a text editor, paste in the following script ::

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
	echo 'Finished MadGraph+Pythia setup...'

save it as ``MGsetup.sh``, make it an executable with ``chmod +x MGsetup.sh``, and run it with ``./MGsetup.sh``. This script will first download a fresh copy of MadGraph and unpack it. It then will download the FeynRules-generated BSM model we'll work with and place it in the ``models`` folder so we can use it. After this, it will download and install pythia in the main ``MG5_aMC_v2_4_3`` folder. 

In order to install Delphes, we need to first setup root. To do this, type ``setupATLAS`` and, once this is done running, type ``localSetupROOT``. Root should now be available to you and you can start it by typing ``root`` [#]_. To install Delphes, cd into the ``local`` folder, open a text editor, paste in the following script ::

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
	echo 'Finished Delphes setup...'

save it as ``DELPHESsetup.sh``, make it an executable with ``chmod +x DELPHESsetup.sh``, and run it with ``./DELPHESsetup.sh``. This script will install Delphes in the ``local`` folder and download the specially designed tcl script mentioned above to simulate a detector in a 100 TeV hadron collider and place it in the delphes cards folder. 

**Note:** in order to properly set all environment variables for the rest of the analysis, you should now log out and then back into titan before continuing.

-----------------------
Parallelized Simulation
-----------------------

In this section, we'll take a simple BSM process and run it on the titan cluster for a single point in the BSM parameter space. To begin, cd to the ``local`` folder, open a text editor, paste in the following script ::

	import model LNVatLHC_UFO
	generate p p > sp~ sp @1
	add process p p > sp sp~ j @2
	output testrun

and save it as ``mgscript.cmd``. To run it, cd into the ``MG5_aMC_v2_4_3`` folder and type the cmd ``./bin/mg5_aMC ../mgscript.cmd``. This will create MadGraph output in the ``testrun`` folder for pair production of two new charged scalars. In order to demonstrate how to run parallel simulations of this process on titan, cd back to the ``local`` folder, open a text editor, paste in the following script ::

	#/bin/sh

	MGbase=$HOME/local/MG5_aMC_v2_4_3
	Base=$HOME/MGjobs
	mkdir $Base

	for (( i=1; i<=5; i++ ));
	do

	jobBase=$Base/job$i
	cardBase=$jobBase/MG5_aMC_v2_4_3/testrun/Cards

	mkdir $jobBase
	cp -r $MGbase $jobBase

	cd $jobBase

	echo "" >> bout.log
	echo "" >> berr.log

	echo "#/bin/sh" >> MGscript.sh
	echo "#PBS -N LNVjob$i" >> MGscript.sh
	echo "echo 'seed: $i'" >> MGscript.sh
	echo "printenv" >> MGscript.sh
	echo "$jobBase/MG5_aMC_v2_4_3/testrun/bin/generate_events 0 run$i" >> MGscript.sh

	cp $jobBase/MG5_aMC_v2_4_3/testrun/Cards/pythia_card_default.dat $jobBase/MG5_aMC_v2_4_3/testrun/Cards/pythia_card.dat
	sed -i "33s/.*/      $i       = iseed   ! rnd seed (0=assigned automatically=default))/" $jobBase/MG5_aMC_v2_4_3/testrun/Cards/run_card.dat

	chmod +x MGscript.sh
	qsub -e berr.log -o bout.log MGscript.sh

	done

save it as ``MGbatch.sh``, make it an executable with ``chmod +x MGbatch.sh``, and run it with ``./MGbatch.sh``. This script will run 5 separate MadGraph instances simultaneously which each generate 10,000 parton level events and then shower, hadronize, and match the results with Pythia. 

Once all these jobs are done running, i.e., none of the jobs show up when you type ``qstat`` anymore, then we need to run Delphes on each of these files. This cannot be done in a parallel manner with the current state of the titan cluster but we can still run it locally. To do this, cd back to the ``local`` folder, open a text editor, paste in the following script ::

	#!/bin/bash

	DelphesBase=$HOME/local/delphes

	for (( i=1; i<=5; i++ ));
	do

	echo ''
	echo "Running Delphes on run$i..."

	EventBase=$HOME/MGjobs/job$i/MG5_aMC_v2_4_3/testrun/Events/run$i

	gunzip $EventBase/tag_1_pythia_events.hep.gz
	$DelphesBase/DelphesSTDHEP $DelphesBase/cards/PW_FCC_delphes_card.tcl $EventBase/delphes_output.root $EventBase/tag_1_pythia_events.hep
	$DelphesBase/root2lhco $EventBase/delphes_output.root $EventBase/delphes_output.lhco

	done

	echo ''
	echo 'Done...'

save it as ``runDELPHES.sh`` and make it an executable with ``chmod +x runDELPHES.sh``. Now, after setting up root again by typing ``setupATLAS`` and then ``localSetupROOT``, run this script with ``./runDELPHES.sh``. This will go into each ``Events`` folder, one by one, and run Delphes on the existing pythia.hep files to produce both root and lhco files using the specialized 100 TeV tcl Delphes card. If everything ran correctly, there should now be a delphes_output.root and delphes_root.lhco file in each of the Events/run folders.

--------------
Final Comments
--------------

This concludes the tutorial on how to perform distributed MadGraph + Pythia + Delphes simulations of a BSM FeynRules model on the titan cluster. In order to expand on this for your own research, you'll need your own FeynRules UFO model folder. In order to make changes to run_card.dat or param_card.dat in a parallel manner, add extra ``sed`` commands into the ``MGbatch.sh`` script. 

If you have any questions or problems with the above procedure, feel free to contact `me <mailto:pwinslow@physics.umass.edu>`_.



.. [#] To quit root, type .q.