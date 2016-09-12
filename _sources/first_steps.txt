======================================
First Steps
======================================

Once you've logged into titan, there are still a few things that need to be done before you can run simulations.

--------------------------------------
Basic Understanding
--------------------------------------

The UMass cluster consists of a head node named **titan.physics.umass.edu** along with a series of nodes named **titan31**, **titan32**, **titan33**,... . Following the instructions in :doc:`getting_started` gets you onto the head node. From there, you have access to and can submit jobs to any of the other nodes.

Batch job management is done with Torque (OpenPBS). Jobs are submitted by users with the qsub command and are scheduled to run using a fair-share algorithm which prioritizes jobs based on all users' recent cpu utilization. This means, for example, that a user who wants to run a few quick jobs will not have to wait for another user who has hundreds of 10-hour long jobs already in the system.

To learn more about the UMass Tier 3 cluster see `here <https://twiki.cern.ch/twiki/bin/view/Main/UMassCluster>`_. To learn more about Torque and job scheduling/management on the UMass cluster see `here <https://twiki.cern.ch/twiki/bin/view/Main/OpenPBS>`_.


--------------------------------------
First Time Setup
--------------------------------------

Once you've logged into titan, you should be in your home directory [#]_. Here you should find a **.zshenv** file (if there is none, then just create it). Open it with your favorite text editor and insert the following lines ::

	# cvmfs Tier 3 env setup
	export ATLAS_LOCAL_ROOT_BASE=/cvmfs/atlas.cern.ch/repo/ATLASLocalRootBase
	alias setupATLAS='source ${ATLAS_LOCAL_ROOT_BASE}/user/atlasLocalSetup.sh'
	export ALRB_localConfigDir=$HOME/localConfig

Once this is done, save and exit. In order for the changes to take effect in your current shell, type ::

	source ~/.zshenv

Before doing any analysis, you'll always want to cd to your folder on the /data2 disk. Once here, you should check to make sure that the node you're on (should be the titan node) is up to date and running SLC6. To do this, run the following commands ::

	cat /etc/*-release



.. [#] You can check this with the **pwd** command, which should return **/home/username**.