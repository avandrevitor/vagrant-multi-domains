# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "puppetlabs/debian-8.2-64-puppet"
  
  config.vm.define "server" do |server|
	server.vm.hostname = "fortis.local"
	server.vm.network "forwarded_port", guest: 8123, host: 8123
	server.vm.network "forwarded_port", guest: 8124, host: 8124
	server.vm.network "forwarded_port", guest: 8125, host: 8125
	server.vm.synced_folder "./app", "/var/www/"
	server.vm.synced_folder "./files", "/etc/apache2/sites-available/"
	
	server.vm.provider :virtualbox do |v|
		v.name = "fortis.local"
		v.customize [
			"modifyvm", :id,
			"--name", "fortis.local",
			"--memory", 512,
			"--cpus", 1,
		]
	end
	
	server.vm.provision "shell", inline: <<-SHELL
		# APT UPDATE
		apt-get update --fix-missing -y
		apt-get autoclean
		
		# APACHE 2
		apt-get install -y apache2 apache2-utils apache2-mpm-prefork curl build-essential vim git subversion 
		a2enmod rewrite headers
		sed -i "s/LockFile ${APACHE_LOCK_DIR}/accept.lock/Mutex file:${APACHE_LOCK_DIR} default/g" /etc/apache2/apache2.conf
		echo "ServerName localhost" >> /etc/apache2/httpd.conf
		/etc/init.d/apache2 restart
		
		# PHP
		apt-get install -y php5 php5-cli php5-curl php5-gd php5-mysql php5-recode php5-gmp php5-xmlrpc php5-xsl php5-intl php5-mcrypt php5-imagick php5-json libapache2-mod-php5 php5-xdebug
		sed -i "s/short_open_tag = On/short_open_tag = Off/g" /etc/php5/apache2/php.ini
		sed -i "s/;date.timezone =/date.timezone = America\/Sao_Paulo/g" /etc/php5/apache2/php.ini
		sed -i "s/memory_limit = 128M/memory_limit = 256M/g" /etc/php5/apache2/php.ini
		sed -i "s/_errors = Off/_errors = On/g" /etc/php5/apache2/php.ini
		sed -i "s/error_reporting = -1/error_reporting = 30711/g" /etc/php5/apache2/php.ini #E_ALL & ~E_NOTICE & ~E_STRICT

		/etc/init.d/apache2 restart
		
		# COMPOSER
		curl -sS https://getcomposer.org/installer | php
		mv composer.phar /usr/local/bin/composer
		
		echo "
			NameVirtualHost *:8123
			NameVirtualHost *:8124
			NameVirtualHost *:8125
			 
			Listen 8123
			Listen 8124
			Listen 8125
		" >> /etc/apache2/ports.conf

		a2ensite br_fortis
		a2ensite en_fortis
		a2ensite fr_fortis
		
		echo "127.0.1.1		br.fortis.local" >> /etc/hosts
		echo "127.0.1.1		en.fortis.local" >> /etc/hosts
		echo "127.0.1.1		fr.fortis.local" >> /etc/hosts
		
		/etc/init.d/apache2 restart
		
		# CUSTOMIZE VIM
		#wget https://gist.githubusercontent.com/avandrevitor/86319d80065f35616ff914580ea59327/raw/53220a893521dd8b46955270f00adbf5d81b2bb5/.vimrc && mv ~/.vimrc /home/vagrant/
				
	SHELL
  end  
end
