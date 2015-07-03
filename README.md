# MyConnectome-VM: A virtual machine to implement MyConnectome analyses.

The [MyConnectome project](http://www.myconnectome.org) is a project meant to investigate the relations between mind, brain, and body across an extended period of time in a single individual.  One of the major goals of the project is to serve as a testbed for reproducible analysis practices.  For this reason, we have released the data and as much code as possible for the processing and analyses.  

Much of the preprocessing of the neuroimaging and genomic data were performed using supercomputing resources at the [Texas Advanced Computing Center](http://www.tacc.utexas.edu), which cannot easily be reproduced on standard computers. For these, we have (or will) made the code and data available.  For all other processing and analyses, we have created the [myconnectome python package] (https://github.com/poldrack/myconnectome) that implements these analyses. Any intersted researcher can clone this repository and run the analyses on their own computer.  

Because the software requires a large number of dependencies, we have also created a virtual machine that one can easily set up on one's own machine to perform all of the statistical analyses reported in the MyConnectome paper (Poldrack et al., submitted).  
More info on how to obtain these keys is available [here](http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSGettingStartedGuide/AWSCredentials.html).


## Setting up the virtual machine

1. Install [VirtualBox] (https://www.virtualbox.org/wiki/Downloads)

2. Install [Vagrant] (http://www.vagrantup.com/downloads) - Vagrant is a provisioning system that sets up the virtual machine.

3. If you don't already have it, install [git] (https://git-scm.com/downloads)

4.  cd to the directory where you want to house the project, and then clone the myconnectome vagrant setup:
`git clone https://github.com/poldrack/myconnectome-vm.git`

5. cd into the vm directory: `cd myconnectome-vm`

6. set up the vagrant VM (which may take a little while):
`vagrant up`

7.  Step 6 will automatically start the analysis processes, which will take several hours to complete.  Using a web browser on your local machine, view the results at [http://192.128.0.20:5000](http://192.128.0.20:5000)

## Digging deeper

If you wish to log into the virtual machine and look more closely at the results, you can do so from within the directory containing the Vagrantfile, using this command:

`vagrant ssh`

The code for the analyses along with all of the results are located in the myconnectome directory.

## Copying the data

If you wish to copy the data and results from the virtual machine to your host machine, you can do so by logging into the VM (using "vagrant ssh") and executing the following command:

`cp -r myconnectome /vagrant/`

This will place a copy of the results in the directory where the Vagrantfile is located on your host machine.

## If things go wrong

If the VM crashes for some reason (which can occur if there is a network hiccup when it is trying to download data), the best thing to do is to destroy the VM using the command:

`vagrant destroy`

And then restart it as outlined above.  

## Things we wish we would have done

We learned a lot in the course of doing this project, and there are a number of things that we would probably do differently, but at this point just don't have the cycles to do it.  Some of these include:

- Use a better file distribution system.  Currently the get_data function uses fixed lists of files to download from S3, because we didn't want to require each user to enter an AWS access key.  This is a rather brittle system.
- Use Jupyter to expose working R notebooks
- Use conda to manage all of the installation dependencies [http://continuum.io/blog/conda-data-science](http://continuum.io/blog/conda-data-science) rather than the current mixture of conda/pip/R
