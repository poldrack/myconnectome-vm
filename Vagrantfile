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
$HOME/miniconda/bin/pip install flask
$HOME/miniconda/bin/pip install flup

echo 'deb http://cran.rstudio.com/bin/linux/ubuntu precise/' >/tmp/myppa.list
sudo cp /tmp/myppa.list /etc/apt/sources.list.d/
rm /tmp/myppa.list

sudo apt-get update > /dev/null
sudo apt-get install -y --force-yes libicu48
sudo apt-get install -y --force-yes r-base git connectome-workbench
sudo apt-get install -y --force-yes r-base
sudo apt-get install -y --force-yes xserver-xorg-core

# NGINX install
sudo apt-get install -y nginx gunicorn
sudo apt-get install -y supervisor


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

if ! [ -L /var/www/myconnectome ]; then
  sudo ln -fs /home/vagrant/myconnectome /var/www
fi

# Web interface
if [ ! -d /var/www/templates ]
then
  sudo mkdir /var/www/templates
fi

# wcgi file
if ! [ -f /var/www/myconnectome.wcgi ]; then
echo """
#!/home/vagrant/miniconda/bin/python
import sys, os
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert (0,'/var/www')
os.chdir("/var/www")
from index import app as application
""" >/tmp/oljnjkhf
  sudo cp /tmp/oljnjkhf /var/www/myconnectome.wcgi

sudo chmod +x /var/www/myconnectome.wcgi

# Index script
if ! [ -f /home/vagrant/myconnectome/index.py ]; then
  echo """
from flask import Flask, render_template, Markup
import os

app = Flask(__name__)

@app.route('/')
def show_analyses():

    timeseries_files = {'myconnectome/timeseries/timeseries_analyses.html':'Timeseries analyses',
                 'myconnectome/timeseries/Make_Timeseries_Heatmaps.html':'Timeseries heatmaps',
                       'myconnectome/timeseries/Make_timeseries_plots.html':'Timeseries plots',
                       'myconnectome/timeseries/behav_heatmap.pdf':'Behavioral timeseries heatmap',
                       'myconnectome/timeseries/wincorr_heatmap.pdf':'Within-network connectivity timeseries heatmap',
                       'myconnectome/timeseries/wincorr_heatmap.pdf':'Within-network connectivity timeseries heatmap',
                       'myconnectome/timeseries/wgcna_heatmap.pdf':'Gene expression module timeseries heatmap'}

    rna_files =        {'myconnectome/rna-seq/RNAseq_data_preparation.html':'RNA-seq data preparation',
                 'myconnectome/rna-seq/Run_WGCNA.html':'RNA-seq WGCNA analysis',
                       'myconnectome/rna-seq/snyderome/Snyderome_data_preparation.html':'RNA-seq Snyderome analysis'}

    meta_files =       {'myconnectome/metabolomics/Metabolomics_clustering.html':'Metabolomics data preparation'}

    # Check if the file exists, render context based on existence            
    timeseries_context = create_context(timeseries_files)
    rna_context = create_context(rna_files)
    meta_context = create_context(meta_files)
    return render_template('index.html',timeseries_context=timeseries_context,rna_context=rna_context,meta_context=meta_context)

def create_context(link_dict):
    processing_markup = Markup(' (processing)')
    urls = []; descriptions = []; styles = []; titles = []
    for filename,description in link_dict.iteritems():
        if os.path.exists(filename):
            urls.append(filename)
            descriptions.append(description)
            styles.append('color:green;')
            titles.append(description)
        else:
            urls.append('#')
            descriptions.append('%s %s' %(description,processing_markup))
            styles.append('color:#ACBAC1;')
            titles.append('PROCESSING')
    return zip(urls,descriptions,styles,titles)

if __name__ == '__main__':
    app.debug = False
    app.run(host="0.0.0.0")
""" >/tmp/boocoo
  sudo cp /tmp/boocoo /var/www/index.py
fi

# Index Template
if ! [ -f /home/vagrant/myconnectome/templates/index.html ]; then
  echo """
<!DOCTYPE html>
<html lang='en'>
  <head>
    <meta charset='utf-8'>
    <meta http-equiv='X-UA-Compatible' content='IE=edge'>
    <meta name='viewport' content='width=device-width, initial-scale=1.0'>
    <meta name='description' content='MyConnectome Analyses, Poldracklab'>
    <meta name='author' content='Poldracklab'>
    <link rel='shortcut icon' href='assets/img/favicon.png'>

    <title>My Connectome Analyses</title>

    <!-- Bootstrap -->
    <link href='https://maxcdn.bootstrapcdn.com/bootstrap/3.3.4/css/bootstrap.min.css' rel='stylesheet'>

    <style>
    @import "http://fonts.googleapis.com/css?family=Lato:400,300,700,900";body,div,dl,dt,dd,ul,ol,li,h1,h2,h3,h4,h5,h6,pre,code,form,fieldset,legend,input,textarea,p,blockquote,th,td{margin:0;padding:0}table{border-collapse:collapse;border-spacing:0}fieldset,img{border:0}address,caption,dfn,th,var{font-style:normal;font-weight:400}li{list-style:none}caption,th{text-align:left}h1,h2,h3,h4,h5,h6{font-size:100%;font-weight:400}body{background:url(https://raw.githubusercontent.com/vsoch/myconnectome-vm/update/flask/assets/img/bg.jpg);font-family:'Lato',sans-serif;font-weight:300;font-size:16px;color:#555;line-height:1.6em;-webkit-font-smoothing:antialiased;-webkit-overflow-scrolling:touch}h1,h2,h3,h4,h5,h6{font-family:'Lato',sans-serif;font-weight:300;color:#444}h1{font-size:40px}p{margin-bottom:20px;font-size:16px}a{color:#ACBAC1;word-wrap:break-word;-webkit-transition:color .1s ease-in,background .1s ease-in;-moz-transition:color .1s ease-in,background .1s ease-in;-ms-transition:color .1s ease-in,background .1s ease-in;-o-transition:color .1s ease-in,background .1s ease-in;transition:color .1s ease-in,background .1s ease-in}a:hover,a:focus{color:#4F92AF;text-decoration:none;outline:0}a:before,a:after{-webkit-transition:color .1s ease-in,background .1s ease-in;-moz-transition:color .1s ease-in,background .1s ease-in;-ms-transition:color .1s ease-in,background .1s ease-in;-o-transition:color .1s ease-in,background .1s ease-in;transition:color .1s ease-in,background .1s ease-in}.alignleft{text-align:left}.alignright{text-align:right}.aligncenter{text-align:center}.btn{display:inline-block;padding:10px 20px;margin-bottom:0;font-size:14px;font-weight:400;line-height:1.428571429;text-align:center;white-space:nowrap;vertical-align:middle;cursor:pointer;-webkit-user-select:none;-moz-user-select:none;-ms-user-select:none;-o-user-select:none;user-select:none;background-image:none;border:1px solid transparent;border-radius:0}#wrapper{text-align:center;padding:50px 0;min-height:650px;width:100%;-webkit-background-size:100%;-moz-background-size:100%;-o-background-size:100%;background-size:100%;-webkit-background-size:cover;-moz-background-size:cover;-o-background-size:cover;background-size:cover}#wrapper h1{margin-top:60px;margin-bottom:40px;color:#fff;font-size:45px;font-weight:900;letter-spacing:-1px}h2.subtitle{color:#fff;font-size:24px}.logo_box{width:349px;float:left;border-right:1px solid #303030;height:280px;position:relative}h1{padding:12px 70px 12px 20px;position:absolute;right:0;text-align:left;top:25%;float:left;color:#fff;letter-spacing:-1px;font-size:38px}h1 cufon{margin-bottom:-4px}.main_box{float:left;width:500px;height:95px;padding:25px}h2{font-family:Georgia;color:#ffe400;font-size:24px;margin-bottom:10px;margin-top:20px}h2 span{color:#fff;font-size:16px;line-height:26px;font-style:italic}ul.info{width:500px;padding:0;margin:10px 0 0;float:left}ul.info li{margin-bottom:20px;clear:both;float:left}ul.info li p{font-size:13px;line-height:20px;color:#fff;float:left;margin:0}.connect{width:145px;padding-left:20px;float:left;padding-top:20px}.connect img{margin-right:5px}
    </style>
    
    <!-- HTML5 shim and Respond.js IE8 support of HTML5 elements and media queries -->
    <!--[if lt IE 9]>
      <script src='https://oss.maxcdn.com/libs/html5shiv/3.7.0/html5shiv.js'></script>
      <script src='https://oss.maxcdn.com/libs/respond.js/1.3.0/respond.min.js'></script>
    <![endif]-->
  </head>
  <body>
<div id='wrapper'>
  <div class='container'>
    <div class='logo_box'><h1>MyConnectome<br/>Analyses</h1></div>          
      <div class='main_box'>
	<h2>Timeseries analyses</h2>
	<ul>
            {% for timeseries_url, timeseries_description, timeseries_style, timeseries_title in timeseries_context %}
    	    <li><a href='{{ timeseries_url }}' style='{{ timeseries_style }}' title='{{ timeseries_title }}'>{{ timeseries_description }}</a></li>
            {% endfor %}
	    <li><a href='myconnectome/timeseries/'>Listing of all files</a></li>
        </ul>
	<h2>RNA-seq analyses</h2>
        <ul>
            {% for rna_url, rna_description, rna_style, rna_title in rna_context %}
    	    <li><a href='{{ rna_url }}' style='{{ rna_style }}' title='{{ rna_title }}'>{{ rna_description }}</a></li>
            {% endfor %}
	    <li><a href='myconnectome/rna-seq/'>Listing of all files</a><li>
        </ul>
	<h2>Metabolomic analyses</h2>
        <ul>
            {% for meta_url, meta_description, meta_style, meta_title in meta_context %}
    	    <li><a href='{{ meta_url }}' style='{{ meta_style }}' title='{{ meta_title }}'>{{ meta_description }}</a></li>
            {% endfor %}
	<li><a href='myconnectome/metabolomics/'>Listing of all files</a></li>		
	</ul>
    </div>
  </div>
</div>
    <script src='https://code.jquery.com/jquery-2.1.4.min.js'></script>
    <script src='https://maxcdn.bootstrapcdn.com/bootstrap/3.3.4/js/bootstrap.min.js'></script>
  </body>
</html>
""" >/tmp/asfdlkjsd
  sudo cp /tmp/asfdlkjsd /home/vagrant/myconnectome/templates/index.html 
fi

# NGINX Configuration
if ! [ -f /etc/nginx/sites-available/flask_project ]; then
  echo """
server {
    location / {
        proxy_pass http://0.0.0.0:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    location /templates {
        alias  /home/vagrant/myconnectome/templates/;
    }
}
""" >/tmp/aijfaef
  sudo cp /tmp/aijfaef /etc/nginx/sites-available/flask_project
fi

# Supervisor Configuration
if ! [ -f /etc/supervisor/conf.d/flask_project.conf ]; then
  echo """
[program:flask_project]
command = gunicorn index:app -b 0.0.0.0:5000
directory = /home/vagrant/myconnectome
user = vagrant
""" >/tmp/abcde
  sudo cp /tmp/abcde /etc/supervisor/conf.d/flask_project.conf
fi

sudo /etc/init.d/nginx start
sudo rm /etc/nginx/sites-enabled/default
sudo ln -s /etc/nginx/sites-available/flask_project /etc/nginx/sites-enabled/flask_project
sudo /etc/init.d/nginx restart
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start flask_project

SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.ssh.forward_x11 = true

  config.vm.define :engine do |engine_config|
    engine_config.vm.box = "gridneuro"
    #engine_config.vm.box_url = "http://files.vagrantup.com/precise64.box"
    engine_config.vm.box_url = "https://dl.dropboxusercontent.com/u/363467/precise64_neuro.box"

    engine_config.vm.network :private_network, ip: "192.168.0.20"
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
