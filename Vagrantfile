# this was adapted from nipype Vagrantfile

VAGRANTFILE_API_VERSION = "2"

$script = <<SCRIPT

if [ ! -d $HOME/miniconda ]
then
 # install anaconda
 wget http://repo.continuum.io/miniconda/Miniconda-latest-Linux-x86_64.sh -O miniconda.sh
 chmod +x miniconda.sh
 ./miniconda.sh -b
 echo "export PATH=$HOME/miniconda/bin:\\$PATH" >> .bashrc
fi

# install nipype dependencies
$HOME/miniconda/bin/conda update --yes conda
$HOME/miniconda/bin/pip install setuptools
$HOME/miniconda/bin/conda install --yes pip numpy scipy nose traits networkx
$HOME/miniconda/bin/conda install --yes dateutil ipython-notebook matplotlib
$HOME/miniconda/bin/conda install --yes statsmodels boto  pandas scikit-learn
$HOME/miniconda/bin/pip install nibabel
$HOME/miniconda/bin/pip install gtf_to_genes
$HOME/miniconda/bin/pip install suds
$HOME/miniconda/bin/pip install mygene

echo 'deb http://cran.rstudio.com/bin/linux/ubuntu precise/' >/tmp/myppa.list
sudo cp /tmp/myppa.list /etc/apt/sources.list.d/
rm /tmp/myppa.list

sudo apt-get update > /dev/null
sudo apt-get install -y --force-yes libicu48
sudo apt-get install -y --force-yes r-base git connectome-workbench
sudo apt-get install -y --force-yes r-base
sudo apt-get install -y --force-yes xserver-xorg-core

if [ ! -d $HOME/R_libs ]
then
mkdir $HOME/R_libs
echo "export R_LIBS_USER=$HOME/R_libs" >> .bashrc
fi


if [ ! -d $HOME/myconnectome ]
then
  git clone https://github.com/poldrack/myconnectome.git $HOME/myconnectome
  echo "export MYCONNECTOME_DIR=$HOME/myconnectome" >> .bashrc
  echo "export WORKBENCH_BIN_DIR=/usr/bin" >> .bashrc
  cd $HOME/myconnectome
fi

sudo apt-get install -y apache2
sudo a2enmod rewrite

if ! [ -L /var/www ]; then
  sudo rm -rf /var/www
  sudo ln -fs /vagrant /var/www
  sudo echo 'ServerRoot "/home/vagrant/myconnectome/"' |sudo tee --append /etc/apache2/apache2.conf
  echo """
<VIRTUALHOST *:80>
Alias /myconnectome/ "/home/vagrant/myconnectome/"
<Directory /home/vagrant/myconnectome/>
    Options Indexes FollowSymLinks Includes ExecCGI
    AllowOverride All
    Allow from all
</Directory>
</VIRTUALHOST>""" > /tmp/asdflj
  sudo cp /tmp/asdflj /etc/apache2/sites-enabled/001-myconnectome
  sudo service apache2 restart
fi

if ! [ -f /home/vagrant/myconnectome/index.html ]; then
  echo """
<h1>Myconnectome analyses</h1>

<h2>Timeseries analyses</h2>
<a href='timeseries/timeseries_analyses.html'>Timeseries analyses</a>
<p>
<a href='timeseries/Make_Timeseries_Heatmaps.html'>Timeseries heatmaps</a>
<p>
<a href='timeseries/Make_timeseries_plots.html'>Timeseries plots</a>
<p>
<a href='timeseries/behav_heatmap.pdf'>Behavioral timeseries heatmap</a>
<p>
<a href='timeseries/wincorr_heatmap.pdf'>Within-network connectivity timeseries heatmap</a>
<p>
<a href='timeseries/wgcna_heatmap.pdf'>Gene expression module timeseries heatmap</a>
<p>
<a href='timeseries/'>Listing of all files</a>

<h2>RNA-seq analyses</h2>
<a href='rna-seq/RNAseq_data_preparation.html'>RNA-seq data preparation</a>
<p>
<a href='rna-seq/Run_WGCNA.html'>RNA-seq WGCNA analysis</a>
<p>
<a href='rna-seq/snyderome/Snyderome_data_preparation.html'>RNA-seq Snyderome analysis</a>
<p>
<a href='rna-seq/'>Listing of all files</a>


<h2>Metabolomic analyses</h2>
<a href='metabolomics/Metabolomics_clustering.html'>Metabolomics data preparation</a>
<p>
<a href='metabolomics/'>Listing of all files</a>


""" >/tmp/asfdlkjsd
  cp /tmp/asfdlkjsd /home/vagrant/myconnectome/index.html 
fi

SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.ssh.forward_x11 = true

  config.vm.define :engine do |engine_config|
    engine_config.vm.box = "gridneuro"
    #engine_config.vm.box_url = "http://files.vagrantup.com/precise64.box"
    engine_config.vm.box_url = "https://dl.dropboxusercontent.com/u/363467/precise64_neuro.box"

    engine_config.vm.network :private_network, ip: "192.168.100.20"
    engine_config.vm.hostname = 'myconnectome'
    engine_config.vm.synced_folder "myconnectome_data", "/home/vagrant/myconnectome", create: true

    engine_config.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
      vb.customize ["modifyvm", :id, "--ioapic", "on"]
      vb.customize ["modifyvm", :id, "--memory", "4096"]
      vb.customize ["modifyvm", :id, "--cpus", "4"]
    end

    engine_config.vm.provision "shell", :privileged => false, inline: $script
  end
end
