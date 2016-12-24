======================================
First Steps
======================================

Both python and root must be installed and made accessible on all nodes of the cluster before you can run parallelized simulations. This section begins with some basic understanding of the titan cluster and then guides you through the necessary installation processes.

--------------------------------------
Basic Understanding
--------------------------------------

The UMass cluster consists of a head node named **titan.physics.umass.edu** along with a series of nodes named **titan31**, **titan32**, **titan33**,... . Following the instructions in :doc:`getting_started` gets you onto the head node. From there, you have access to and can submit jobs to any of the other nodes.

Batch job management is done with Torque (OpenPBS). Jobs are submitted by users with the qsub command and are scheduled to run using a fair-share algorithm which prioritizes jobs based on all users' recent cpu utilization. This means, for example, that a user who wants to run a few quick jobs will not have to wait for another user who has hundreds of 10-hour long jobs already in the system.

To learn more about the UMass Tier 3 cluster see `here <https://twiki.cern.ch/twiki/bin/view/Main/UMassCluster>`_. To learn more about Torque and job scheduling/management on the UMass cluster see `here <https://twiki.cern.ch/twiki/bin/view/Main/OpenPBS>`_.


--------------------------------------
Software Setup
--------------------------------------

Once you've logged into titan, you should be in your home directory (You can check this with the ``pwd`` command, which should return ``/home/your-username``). Here you should create a folder called ``local`` with the cmd ``mkdir ~/local``. Once you've done this, cd into ``local``, open a file with your favorite text editor, and copy and paste the following script into it ::

	#!/bin/bash

	# Install Python
	if [ -d "$HOME/local/anaconda" ]
	then
	echo ''
	echo 'Anaconda Python already installed...'
	echo ''
	else
	echo ''
	echo 'Installing Anaconda Python Distribution...'
	echo ''
	cd $HOME/local
	wget https://3230d63b5fc54e62148e-c95ac804525aac4b6dba79b00b39d1d3.ssl.cf1.rackcdn.com/Anaconda-2.3.0-Linux-x86_64.sh
	bash Anaconda-2.3.0-Linux-x86_64.sh -b -p $HOME/local/anaconda
	rm Anaconda-2.3.0-Linux-x86_64.sh
	echo 'export PYTHONPATH="$HOME/local/anaconda/lib/python2.7/site-packages:{$PYTHONPATH}"' >> $HOME/.zshenv
	echo '' >> $HOME/.bashrc
	echo 'export PATH="$HOME/local/anaconda/bin:$PATH"' >> $HOME/.bashrc
	echo '' >> $HOME/.bash_profile
	echo 'source ~/.zshenv' >> $HOME/.bash_profile
	echo ''
	echo 'Done installing Anaconda Python...'
	fi

	# Install ROOT environment variables
	echo ''
	echo 'Installing ROOT environment variables...'
	echo '' >> $HOME/.zshenv
	echo 'export ROOTSYS=/home/willocq/root/root_v5.34.21/root' >> $HOME/.zshenv
	echo 'export PATH="$ROOTSYS/bin:$PATH"' >> $HOME/.zshenv
	echo 'export LD_LIBRARY_PATH="$ROOTSYS/lib:/home/willocq/bin/hepmc/x86_64-slc6-gcc45-opt/lib:$LD_LIBRARY_PATH"' >> $HOME/.zshenv
	echo 'Done installing ROOT environment variables...'

and save it as ``setup.sh``. Once you've closed the text editor, change the permissions on this file to make it an executable by typing ``chmod +x setup.sh``. The ``setup.sh`` script first checks to see if python is locally installed already and, if not, then it installs it. Once that's done, it then sets a number of environment variables necessary for you to access root [#]_. If you're just getting started for the first time, then just run ``setup.sh`` with the cmd ``./setup.sh`` from the ``local`` folder to install python and set up your environment for access to root [#]_. This process will likely take a minute. After this, both python and root will be locally installed and accessible but, before you can use them, you need to first log out and then log back into titan. This action sources all the new environment settings which lets titan know where all relevant libraries are installed.

The Anaconda Python distribution installs almost all scientific modules you could want when using python. However, if you find you have need of python modules that aren't currently installed, just type ``conda install **module-name**`` at the cmd line. For example, the beautiful soup module is very helpful for scraping data directly off of webpages and can be installed with the cmd ``conda install beautiful-soup``. The Anaconda distribution will also keep track of all dependencies so you don't have to worry about version requirements or any other such issues.

.. [#] Thanks very much to Haolin Li for pointing out the root installation on titan, allowing us to bypass using the installation from CERN.
.. [#] You may see a warning about setting your PYTHONPATH environment variables during this process. This warning can be safely ignored as the script should have set all these variables for you.