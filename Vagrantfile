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
$HOME/miniconda/bin/pip install gunicorn
$HOME/miniconda/bin/pip install Flask-AutoIndex

echo 'deb http://cran.rstudio.com/bin/linux/ubuntu precise/' >/tmp/myppa.list
sudo cp /tmp/myppa.list /etc/apt/sources.list.d/
rm /tmp/myppa.list

sudo apt-get update > /dev/null
sudo apt-get install -y --force-yes libicu48
sudo apt-get install -y --force-yes r-base git connectome-workbench
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
fi


if [ ! -d $HOME/myconnectome ]
then
  git clone https://github.com/poldrack/myconnectome.git $HOME/myconnectome
  echo "export MYCONNECTOME_DIR=$HOME/myconnectome" >> .bashrc
  echo "export WORKBENCH_BIN_DIR=/usr/bin" >> .bashrc
  cd $HOME/myconnectome
fi

if ! [ -L /var/www/myconnectome ]; then
  sudo ln -fs /home/vagrant/myconnectome /var/www/results
fi

# Web interface
if [ ! -d /var/www/templates ]
then
  sudo mkdir /var/www/templates
fi

# Index script
if ! [ -f /var/www/index.py ]; then
  echo """
from flask import Flask, render_template, redirect
from flask.ext.autoindex import AutoIndex
from subprocess import Popen
import os

app = Flask(__name__)
index = AutoIndex(app, browse_root='/var/www/results',add_url_rules=False)

@app.route('/results')
@app.route('/results/<path:path>')
def autoindex(path='.'):
    return index.render_autoindex(path)


@app.route('/start')
def start_analyses():

    if not os.path.exists('/home/vagrant/myconnectome/.started'):
        p = Popen(['python', '/home/vagrant/myconnectome/myconnectome/scripts/run_everything.py'])
        filey = open('/home/vagrant/myconnectome/.started','wb').close()

    return redirect('/')

@app.route('/')
def show_analyses():

    timeseries_files = {'/var/www/results/timeseries/timeseries_analyses.html':'Timeseries analyses',
                       '/var/www/results/timeseries/Make_Timeseries_Heatmaps.html':'Timeseries heatmaps',
                       '/var/www/results/timeseries/Make_timeseries_plots.html':'Timeseries plots',
                       '/var/www/results/timeseries/behav_heatmap.pdf':'Behavioral timeseries heatmap',
                       '/var/www/results/timeseries/wincorr_heatmap.pdf':'Within-network connectivity timeseries heatmap',
                       '/var/www/results/timeseries/wincorr_heatmap.pdf':'Within-network connectivity timeseries heatmap',
                       '/var/www/results/timeseries/wgcna_heatmap.pdf':'Gene expression module timeseries heatmap',
                       '/var/www/results/timeseries':'Listing of all files'}

    rna_files =        {'/var/www/results/rna-seq/RNAseq_data_preparation.html':'RNA-seq data preparation',
                       '/var/www/results/rna-seq/Run_WGCNA.html':'RNA-seq WGCNA analysis',
                       '/var/www/results/rna-seq/snyderome/Snyderome_data_preparation.html':'RNA-seq Snyderome analysis',
                       '/var/www/results/rna-seq':'Listing of all files'}

    meta_files =       {'/var/www/results/metabolomics/Metabolomics_clustering.html':'Metabolomics data preparation',
                        '/var/www/results/metabolomics':'Listing of all files'}

    # How many green links should we have?
    number_analyses = len(meta_files) + len(rna_files) + len(timeseries_files)

    # If analyses not started, show button
    start_button = True
    if os.path.exists('/home/vagrant/myconnectome/.started'):
        start_button = False

    # Check if the file exists, render context based on existence            
    counter = 0
    timeseries_context,counter = create_context(timeseries_files,counter)
    rna_context,counter = create_context(rna_files,counter)
    meta_context,counter = create_context(meta_files,counter)

    # The counter determines if we've finished running analyses
    analysis_status = 'Analysis is Running'
    if counter == number_analyses:
        analysis_status = 'Analysis Complete'

    return render_template('index.html',timeseries_context=timeseries_context,
                                        rna_context=rna_context,
                                        meta_context=meta_context,
                                        start_button=start_button,
                                        analysis_status=analysis_status)

def create_context(link_dict,counter):
    urls = []; descriptions = []; styles = []; titles = []
    for filename,description in link_dict.iteritems():
        if os.path.exists(filename):
            counter+=1
            urls.append(filename.replace('/var/www',''))
            descriptions.append(description)
            styles.append('color:rgb(25, 234, 25)')
            titles.append(description)
        else:
            urls.append('#')
            descriptions.append('%s %s' %(description,'(processing)'))
            styles.append('color:#ACBAC1;')
            titles.append('PROCESSING')
    return zip(urls,descriptions,styles,titles),counter

if __name__ == '__main__':
    app.debug = False
    app.run(host='0.0.0.0')
""" >/tmp/boocoo
  sudo cp /tmp/boocoo /var/www/index.py
fi

# Index Template
if ! [ -f /var/www/templates/index.html ]; then
  echo """
<!DOCTYPE html>
<html lang='en'>
  <head>
    <meta charset='utf-8'>
    <meta http-equiv='X-UA-Compatible' content='IE=edge'>
    <meta name='viewport' content='width=device-width, initial-scale=1.0'>
    <meta name='description' content='MyConnectome Analyses, Poldracklab'>
    <meta name='author' content='Poldracklab'>

    <title>My Connectome Analyses</title>

    <!-- Bootstrap -->
    <link href='https://maxcdn.bootstrapcdn.com/bootstrap/3.3.4/css/bootstrap.min.css' rel='stylesheet'>

    <style>
    @import "http://fonts.googleapis.com/css?family=Lato:400,300,700,900";body,div,dl,dt,dd,ul,ol,li,h1,h2,h3,h4,h5,h6,pre,code,form,fieldset,legend,input,textarea,p,blockquote,th,td{margin:0;padding:0}table{border-collapse:collapse;border-spacing:0}fieldset,img{border:0}address,caption,dfn,th,var{font-style:normal;font-weight:400}li{list-style:none}caption,th{text-align:left}h1,h2,h3,h4,h5,h6{font-size:100%;font-weight:400}body{background:url(https://raw.githubusercontent.com/vsoch/myconnectome-vm/update/flask/assets/img/bg.jpg);font-family:'Lato',sans-serif;font-weight:300;font-size:16px;color:#555;line-height:1.6em;-webkit-font-smoothing:antialiased;-webkit-overflow-scrolling:touch}h1,h2,h3,h4,h5,h6{font-family:'Lato',sans-serif;font-weight:300;color:#444}h1{font-size:40px}p{margin-bottom:20px;font-size:16px}a{color:#ACBAC1;word-wrap:break-word;-webkit-transition:color .1s ease-in,background .1s ease-in;-moz-transition:color .1s ease-in,background .1s ease-in;-ms-transition:color .1s ease-in,background .1s ease-in;-o-transition:color .1s ease-in,background .1s ease-in;transition:color .1s ease-in,background .1s ease-in}a:hover,a:focus{color:#4F92AF;text-decoration:none;outline:0}a:before,a:after{-webkit-transition:color .1s ease-in,background .1s ease-in;-moz-transition:color .1s ease-in,background .1s ease-in;-ms-transition:color .1s ease-in,background .1s ease-in;-o-transition:color .1s ease-in,background .1s ease-in;transition:color .1s ease-in,background .1s ease-in}.alignleft{text-align:left}.alignright{text-align:right}.aligncenter{text-align:center}.btn{display:inline-block;padding:10px 20px;margin-bottom:0;font-size:14px;font-weight:400;line-height:1.428571429;text-align:center;white-space:nowrap;vertical-align:middle;cursor:pointer;-webkit-user-select:none;-moz-user-select:none;-ms-user-select:none;-o-user-select:none;user-select:none;background-image:none;border:1px solid transparent;border-radius:0}#wrapper{text-align:center;padding:50px 0;min-height:650px;width:100%;-webkit-background-size:100%;-moz-background-size:100%;-o-background-size:100%;background-size:100%;-webkit-background-size:cover;-moz-background-size:cover;-o-background-size:cover;background-size:cover}#wrapper h1{margin-top:60px;margin-bottom:40px;color:#fff;font-size:45px;font-weight:900;letter-spacing:-1px}h2.subtitle{color:#fff;font-size:24px}.logo_box{width:349px;float:left;border-right:1px solid #303030;height:600px;position:relative}h1{padding:12px 70px 12px 20px;position:absolute;right:0;text-align:left;top:25%;float:left;color:#fff;letter-spacing:-1px;font-size:38px}h1 cufon{margin-bottom:-4px}.main_box{float:left;width:500px;height:600px;padding:25px}h2{font-family:Georgia;color:#ffe400;font-size:24px;margin-bottom:10px;margin-top:20px}h2 span{color:#fff;font-size:16px;line-height:26px;font-style:italic}ul.info{width:500px;padding:0;margin:10px 0 0;float:left}ul.info li{margin-bottom:20px;clear:both;float:left}ul.info li p{font-size:13px;line-height:20px;color:#fff;float:left;margin:0}.connect{width:145px;padding-left:20px;float:left;padding-top:20px}.connect img{margin-right:5px}
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
    <div class='logo_box'><h1>MyConnectome<br/>Analyses</h1>
    <!-- Analysis Start Button-->
    {% if start_button %}
        <a class='btn btn-default' href='/start' style='left: 40px;bottom: 140px; position:absolute' role='button'>Start Analysis</a>
    {% else %}
        <a class='btn btn-default' href='#' style='left: 40px;bottom: 140px; position:absolute'; role='button' disabled>{{ analysis_status }}</a>
    {% endif %}

    </div>          
      <div class='main_box'>
	<h2>Timeseries analyses</h2>
	<ul>
            {% for timeseries_url, timeseries_description, timeseries_style, timeseries_title in timeseries_context %}
    	    <li><a href='{{ timeseries_url }}' style='{{ timeseries_style }}' title='{{ timeseries_title }}'>{{ timeseries_description }}</a></li>
            {% endfor %}
        </ul>
	<h2>RNA-seq analyses</h2>
        <ul>
            {% for rna_url, rna_description, rna_style, rna_title in rna_context %}
    	    <li><a href='{{ rna_url }}' style='{{ rna_style }}' title='{{ rna_title }}'>{{ rna_description }}</a></li>
            {% endfor %}
        </ul>
	<h2>Metabolomic analyses</h2>
        <ul>
            {% for meta_url, meta_description, meta_style, meta_title in meta_context %}
    	    <li><a href='{{ meta_url }}' style='{{ meta_style }}' title='{{ meta_title }}'>{{ meta_description }}</a></li>
            {% endfor %}		
	</ul>
    </div>
  </div>
</div>
    <script src='https://code.jquery.com/jquery-2.1.4.min.js'></script>
    <script src='https://maxcdn.bootstrapcdn.com/bootstrap/3.3.4/js/bootstrap.min.js'></script>
  </body>
</html>
""" >/tmp/asfdlkjsd
  sudo cp /tmp/asfdlkjsd /var/www/templates/index.html 
fi

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

cd /home/vagrant/myconnectome
$HOME/miniconda/bin/python /home/vagrant/myconnectome/setup.py install

sudo ln -s /etc/nginx/sites-available/flask_project /etc/nginx/sites-enabled/flask_project
cd /var/www
sudo /etc/init.d/nginx restart
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start flask_project
echo "Open browser to 192.168.0.20:5000"

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
