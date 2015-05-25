# MyConnectome-VM: A virtual machine to implement MyConnectome analyses.

The [MyConnectome project](http://www.myconnectome.org) is a project meant to investigate the relations between mind, brain, and body across an extended period of time in a single individual.  One of the major goals of the project is to serve as a testbed for reproducible analysis practices.  For this reason, we have released the data and as much code as possible for the processing and analyses.  

Much of the preprocessing of the neuroimaging and genomic data were performed using supercomputing resources at the [Texas Advanced Computing Center](http://www.tacc.utexas.edu), which cannot easily be reproduced on standard computers. For these, we have (or will) made the code and data available.  For all other processing and analyses, we have created the [myconnectome python package] (https://github.com/poldrack/myconnectome) that implements these analyses. Any intersted researcher can clone this repository and run the analyses on their own computer.  

Because the software requires a large number of dependencies, we have also created a virtual machine that one can easily set up on one's own machine to perform all of the statistical analyses reported in the MyConnectome paper (Poldrack et al., submitted).  

## Setting up the virtual machine

1. Install [VirtualBox] (https://www.virtualbox.org/wiki/Downloads)

2. Install [Vagrant] (http://www.vagrantup.com/downloads) - Vagrant is a provisioning system that sets up the virtual machine.

3. If you don't already have it, install [git] (https://git-scm.com/downloads)

4.  cd to the directory where you want to house the project, and then clone the myconnectome vagrant setup:
`git clone https://github.com/poldrack/myconnectome-vm.git`

5. cd into the vm directory: `cd myconnectome-vm`

6. set up the vagrant VM (which may take a little while):
`vagrant up`

7.  Step 6 will automatically start the analysis processes, which will take several hours to complete.  Using a web browser on your local machine, view the results at [http://192.168.0.20/](http://192.168.0.20/)

## Digging deeper

If you wish to log into the virtual machine and look more closely at the results, you can do so from within the directory containing the Vagrantfile, using this command:

`vagrant ssh`

