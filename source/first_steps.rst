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

	ATLAS_LOCAL_ROOT_BASE=/cvmfs/atlas.cern.ch/repo/ATLASLocalRootBase
	source ${ATLAS_LOCAL_ROOT_BASE}/user/atlasLocalSetup.sh
	localSetupROOT

and save it as ``setupROOT``. Once you've closed the text editor, change the permissions on this file to make it an executable by typing ``chmod +x setupROOT``. While still in the **local** folder, open the text editor again and copy and paste the following script into it ::

	if [ -d "$HOME/local/anaconda" ]
	then
	printf "\nAnaconda Python already installed...\n"
	else
	# Download Anaconda and install
	printf "\nInstalling Anaconda Python Distribution...\n"
	cd $HOME/local
	wget https://3230d63b5fc54e62148e-c95ac804525aac4b6dba79b00b39d1d3.ssl.cf1.rackcdn.com/Anaconda-2.3.0-Linux-x86_64.sh
	bash Anaconda-2.3.0-Linux-x86_64.sh -b -p $HOME/local/anaconda
	# Clean up and append local anaconda distribution to python path
	rm Anaconda-2.3.0-Linux-x86_64.sh
	printf '\nexport PYTHONPATH="$HOME/local/anaconda/lib/python2.7/site-packages:{$PYTHONPATH}"\n' >> $HOME/.zshenv
	printf '\nexport PATH="$HOME/local/anaconda/bin:$PATH"\n' >> $HOME/.bashrc
	printf "\nDone installing Anaconda Python...\n"
	fi
	source $HOME/.bash_profile
	bash

and save it as ``setupPYTHON``. Using the same cmd as before, make this file an executable as well. 

The ``setupROOT`` script installs root. However, this install is not a proper install in the sense that it doesn't install it locally to a cluster node in any way. It just references an existing installation of root made available to you by CERN. The ``setupPYTHON`` script first checks to see if python is locally installed already and, if not, then it installs it. You only need to install python once but you will need to run ``setupROOT`` each time you want to use root. 

If you're just getting started for the first time, then just run ``setupPYTHON`` with the cmd ``./setupPYTHON`` from the ``local`` folder to install python. This process will likely take a minute. After this, python will be locally installed and you should be able to use it with **ONE IMPORTANT CAVEAT**: each time you sign into titan you need to type ``source ~/.zshenv`` before being able to use python again. Without this, you will likely see errors of the form ``ImportError: undefined symbol: PyUnicodeUCS2_AsUnicodeEscapeString``. These indicate that there are two python distributions. The new one you just installed and a much older version with no useful modules. Running ``source ~/.zshenv`` when you login just tells titan to use the new python.

The Anaconda distribution installs python and almost all scientific modules you could want with it. However, if you find you have need of python modules that aren't currently installed, just type ``conda install **module-name**`` at the cmd line. For example, the beautiful soup module is very helpful for scraping data directly off of webpages and can be installed with the cmd ``conda install beautiful-soup``. The Anaconda distribution will also keep track of all dependencies so you don't have to worry about version requirements or any other such issues.

