# this was adapted from nipype Vagrantfile

VAGRANTFILE_API_VERSION = "2"

$script = <<SCRIPT


# # Install neurodebian repo
bash <(wget -q -O- http://neuro.debian.net/_files/neurodebian-travis.sh)


if [ ! -d $HOME/miniconda ]
then
 # install anaconda
 wget http://repo.continuum.io/miniconda/Miniconda-latest-Linux-x86_64.sh -O miniconda.sh
 chmod +x miniconda.sh
 ./miniconda.sh -b
 echo "export PATH=$HOME/miniconda/bin:\\$PATH" >> .bashrc
 echo "export PATH=$HOME/miniconda/bin:\\$PATH" >> .env
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
$HOME/miniconda/bin/pip install flask
$HOME/miniconda/bin/pip install flup
$HOME/miniconda/bin/pip install gunicorn
$HOME/miniconda/bin/pip install nilearn
$HOME/miniconda/bin/pip install Flask-AutoIndex

echo 'deb http://cran.rstudio.com/bin/linux/ubuntu precise/' >/tmp/myppa.list
sudo cp /tmp/myppa.list /etc/apt/sources.list.d/
rm /tmp/myppa.list

sudo apt-get update > /dev/null
sudo apt-get install -y --force-yes libicu48
sudo apt-get install -y --force-yes r-base 
sudo apt-get install -y --force-yes git 
sudo apt-get install -y --force-yes connectome-workbench
sudo apt-get install -y --force-yes r-base
sudo apt-get install -y --force-yes xserver-xorg-core

# NGINX install
sudo apt-get install -y --force-yes nginx
sudo apt-get install -y --force-yes supervisor


if [ ! -d $HOME/R_libs ]
then
mkdir $HOME/R_libs
sudo mkdir /var/www
sudo mkdir /var/www/results
echo "export R_LIBS_USER=$HOME/R_libs" >> .bashrc
echo "export R_LIBS_USER=$HOME/R_libs" >> .env
fi

if [ ! -d $HOME/myconnectome ]
then
  git clone https://github.com/poldrack/myconnectome.git $HOME/myconnectome
  echo "export MYCONNECTOME_DIR=$HOME/myconnectome" >> .bashrc
  echo "export WORKBENCH_BIN_DIR=/usr/bin" >> .bashrc
  echo "export MYCONNECTOME_DIR=$HOME/myconnectome" >> .env
  echo "export WORKBENCH_BIN_DIR=/usr/bin" >> .env
  cd $HOME/myconnectome
fi

if ! [ -L /var/www/myconnectome ]; then
  sudo ln -fs /home/vagrant/myconnectome /var/www/results
fi

# Clone the data explorer
if [ ! -d $HOME/myconnectome-explore ]; then
  git clone https://github.com/vsoch/myconnectome-explore.git $HOME/myconnectome-explore
fi

# Move the static and templates directories
if [ ! -d /var/www/templates ]; then
  sudo mv $HOME/myconnectome-explore/templates /var/www/templates
fi

if [ ! -d /var/www/static ]; then
  sudo mv $HOME/myconnectome-explore/static /var/www/static
fi

if [ ! -f /var/www/index.py ]; then
  sudo mv $HOME/myconnectome-explore/index.py /var/www/index.py
fi

if [ ! -f /var/www/banner.py ]; then
  sudo mv $HOME/myconnectome-explore/banner.py /var/www/banner.py
fi


# NGINX SETUP
sudo /etc/init.d/nginx start
sudo rm /etc/nginx/sites-enabled/default

# NGINX Configuration
if ! [ -f /etc/nginx/sites-available/flask_project ]; then
  echo """
server {
        location / {
                proxy_pass http://0.0.0.0:5000;
        }
}
""" >/tmp/aijfaef
  sudo cp /tmp/aijfaef /etc/nginx/sites-available/flask_project
fi

# Supervisor Configuration
if ! [ -f /etc/supervisor/conf.d/flask_project.conf ]; then
  echo """[program:flask_project]
command = /home/vagrant/miniconda/bin/gunicorn index:app -b 0.0.0.0:5000
directory = /var/www
user = vagrant
""" >/tmp/abcde
  sudo cp /tmp/abcde /etc/supervisor/conf.d/flask_project.conf
fi

# Install my connectome and start analyses
if ! [ -f $HOME/myconnectome/.started ]; then
  cd /home/vagrant/myconnectome
  $HOME/miniconda/bin/python /home/vagrant/myconnectome/setup.py install
fi

# Start the flask application via supervisor
sudo ln -s /etc/nginx/sites-available/flask_project /etc/nginx/sites-enabled/flask_project
cd /var/www
sudo /etc/init.d/nginx restart
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start flask_project
echo ""
echo "Open your browser to 192.128.0.20:5000 to view analysis"

# need to set the date properly in order for AWS up/downloads to work
sudo ntpdate pool.ntp.org

# Start the analysis for the user
if ! [ -f $HOME/myconnectome/.started ]; then
  touch /home/vagrant/myconnectome/.started
  source /home/vagrant/.env
  $HOME/miniconda/bin/python /home/vagrant/myconnectome/myconnectome/scripts/run_everything.py > /home/vagrant/myconnectome/myconnectome_job.out 2> /home/vagrant/myconnectome/myconnectome_job.err &
  # Get the process ID
  MYCONNECTOME_ID=`pgrep python`
  sudo echo "$MYCONNECTOME_ID" >> /home/vagrant/myconnectome/.started
fi
  

SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.ssh.forward_x11 = true

  config.vm.define :engine do |engine_config|
    engine_config.vm.box = "precise64"
    engine_config.vm.box_url = "http://files.vagrantup.com/precise64.box"
    #engine_config.vm.box_url = "https://s3.amazonaws.com/openfmri/virtual-machines/precise64_neuro.box"

    engine_config.vm.network :private_network, ip: "192.128.0.20"
    engine_config.vm.hostname = 'myconnectome-analysis'
    #engine_config.vm.synced_folder "/tmp/myconnectome", "/home/vagrant/myconnectome", create: true

    engine_config.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
      vb.customize ["modifyvm", :id, "--ioapic", "on"]
      vb.customize ["modifyvm", :id, "--memory", "4096"]
      vb.customize ["modifyvm", :id, "--cpus", "4"]
    end

    engine_config.vm.provision "shell", :privileged => false, inline: $script
  end
end
