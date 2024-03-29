Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  config.vm.synced_folder "html/", "/var/www/html", \
    create: true, group: "www-data", owner: "www-data"
  config.vm.synced_folder "Apache logs/", "/var/log/apache2", \
    create: true
  config.vm.network :forwarded_port, guest: 80, host: 8080
  config.vm.network :forwarded_port, guest: 443, host: 8081
  config.vm.network :forwarded_port, guest: 9090, host: 9091
  config.vm.provider "virtualbox" do |virtualbox|
    virtualbox.memory = 3072
  end
  config.vm.provision "shell", inline: <<-SHELL

# https://www.debian.org/releases/stable/amd64/ch05s03.en.html#installer-args
export DEBIAN_FRONTEND=noninteractive

apt-get update

# Install Apache from ppa:ondrej/apache2 repository:
add-apt-repository -y ppa:ondrej/apache2
apt-get install -y apache2

# Enable .htaccess files:
bash -c 'echo -e "<Directory /var/www/html>\n\tAllowOverride All\n</Directory>" >> /etc/apache2/apache2.conf'
a2enmod rewrite

# Enable TLS
bash -c 'openssl req -x509 -out /etc/apache2/localhost.crt -keyout /etc/apache2/localhost.key -newkey rsa:2048 -nodes -sha256 -subj "/CN=localhost" -extensions EXT -config <( printf "[dn]\nCN=localhost\n[req]\ndistinguished_name = dn\n[EXT]\nsubjectAltName=DNS:localhost\nkeyUsage=digitalSignature\nextendedKeyUsage=serverAuth")'
bash -c 'echo -e "\n<VirtualHost *:443>\n    ServerName localhost\n    SSLEngine on\n    SSLCertificateFile /etc/apache2/localhost.crt\n    SSLCertificateKeyFile /etc/apache2/localhost.key\n</VirtualHost>" >> /etc/apache2/apache2.conf'
a2enmod ssl

# Enable Apache mod_headers to set HTTP headers in .htaccess
a2enmod headers

# Install MariaDB from its own repository (https://mariadb.com/kb/en/installing-mariadb-deb-files/):
apt-get install -y apt-transport-https curl
curl -LsS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash
apt-get install -y mariadb-server

# Create a DB and a DB user for WP using default WP installer placeholders for faster WP installation:
mariadb -u root -p --skip-password -e \
"CREATE DATABASE wordpress; \
CREATE USER 'username' IDENTIFIED BY 'password'; \
GRANT ALL ON wordpress.* TO 'username';"

# Install PHP 8 from ppa:ondrej/php repository:
add-apt-repository -y ppa:ondrej/php
apt-get install -y php8.0 php8.0-mysql php8.0-mbstring php8.0-zip php8.0-gd \
php8.0-curl php8.0-xml php8.0-imagick php8.0-mcrypt php8.0-ssh2 imagemagick php8.0-bcmath php8.0-soap

# Increase PHP 8 limits:
sed -i "s/memory_limit = 128M/memory_limit = 1G/" /etc/php/8.0/apache2/php.ini
sed -i "s/post_max_size = 8M/post_max_size = 8G/" /etc/php/8.0/apache2/php.ini
sed -i "s/upload_max_filesize = 2M/upload_max_filesize = 8G/" /etc/php/8.0/apache2/php.ini

# Restart Apache after enabling the Apache rewrite module, ssl module, and increasing the PHP limits:
systemctl restart apache2

# https://cockpit-project.org/
apt-get install -y cockpit

  SHELL
end
