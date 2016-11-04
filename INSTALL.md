## How to install Zammad on Debian 8

#### commands / code marked with an asterisk "*" should be executed as root, everything else - please use the zammad user

#### Install needed Packages
    * apt-get install curl git-core patch build-essential bison build-essential ruby nano
    zlib1g-dev libssl-dev libxml2-dev libxml2-dev sqlite3 libsqlite3-dev autotools-dev 
    libxslt1-dev libyaml-0-2 autoconf automake libreadline6-dev libyaml-dev libtool visudo 
    libgmp-dev libgdbm-dev libncurses5-dev pkg-config libffi-dev libmysqlclient-dev nginx
    

#### We have to tell Debian to get Mysql v5.6+ manually, because by default Debian still goes for v5.5
    
    * wget https://dev.mysql.com/downloads/repo/apt/
    * dpkg -i downloaded package 

Choose mysql-server 5.6 & Click "Ok"
   
    * apt-get update 
    * apt-get install mysql-server

#### Add the zammad user

    * useradd zammad -m -d /opt/zammad -s /bin/bash 
    * echo "export RAILS_ENV=production" >> /opt/zammad/.bashrc 
    
Add the zammad user to the root group
    
    * visudo
    
It should then look something similar to this
    
    # User privilege specification
    root    ALL=(ALL:ALL) ALL
    zammad  ALL=(ALL:ALL) ALL
 
Be sure to choose a save password

    * passwd zammad 


#### Create the MySQL User zammad
    
    * mysql 

this piece of code goes directly into the mysql console

    * CREATE USER 'zammad'@'localhost' IDENTIFIED BY 'CHOOSE A SAFE PASSWORD PLZ'; GRANT ALL PRIVILEGES ON zammad_prod.* TO 'zammad'@'localhost'; FLUSH PRIVILEGES; 

#### Downloading and unpacking Zammad

    *su zammad 
    cd ~ 
    wget https://ftp.zammad.com/zammad-latest.tar.gz 
    tar -xzf zammad-latest.tar.gz 
    exit 

#### Copy and change the Nginx Config

    * cp /opt/zammad/contrib/nginx-zammad.conf /etc/nginx/sites-available/zammad.conf 
    * vi /etc/nginx/sites-available/zammad.conf 

change "zammad.example.com" to your hostname.yourdomain.yourtld

    * ln -s /etc/nginx/sites-available/zammad.conf /etc/nginx/sites-enabled/zammad.conf 

#### Install Environnment

    su zammad 
    cd ~ 
    gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 
    curl -L https://get.rvm.io | bash -s stable 
    source /opt/zammad/.rvm/scripts/rvm 
    echo "source /opt/zammad/.rvm/scripts/rvm" >> /opt/zammad/.bashrc 
    echo "rvm --default use 2.1.5" >> /opt/zammad/.bashrc 
    rvm install 2.1.5 

Change the ruby version according to your installed one

    ruby --version 
    cd /opt/zammad 
    nano 

    gem install bundler 

Install Zammad

    bundle install --without test development postgres 
    cp config/database.yml.dist config/database.yml 

#### Configure the database properties 

Set the username and password you set earlier for the zammad database user (not the linux user!).
You can also specify the database address (if you have a server solely acting as database server).
But localhost/127.0.0.1 will just do fine if you've folowed this guide.

    nano config/database.yml 
    
Now the config should look something like this

    production:
    adapter: mysql2
    database: zammad_prod
    pool: 50
    timeout: 5000
    encoding: utf8
    username: zammad <-- zammad is the myqsl server 
    password: <-- hopefully you set a safe password
    host: 127.0.0.1 <-- If you're running the database locally, you're fine. If not please change to the desired IP.
   
Create the database & seed it 

    rake db:create 
    rake db:migrate 
    rake db:seed 
    rake assets:precompile 

#### Start Zammad

    rails s -p 3001 &>> log/zammad.log & 
    script/websocket-server.rb start &>> log/zammad.log & 
    script/scheduler.rb start &>> log/zammad.log & 
    exit

Restart nginx as root

     * systemctl restart nginx 

You can force nginx to start whenever the system is booting.

     * systemctl enable nginx.service

