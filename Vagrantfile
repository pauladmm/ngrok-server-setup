# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"
  config.vm.network "private_network", ip: "192.168.56.107"


  # Configuración de recursos
  config.vm.provider "virtualbox" do |vb|
    vb.memory = 2048   
    vb.cpus = 2         
  end

  config.vm.provision "shell", name: "server", inline: <<-SHELL
    apt update
    apt install -y nginx openssl ufw curl git

    ufw allow ssh
    ufw allow 'WWW Full'
    ufw delete allow 'WWW'
    ufw allow 'Nginx Full'
    ufw delete allow 'Nginx HTTP'
    ufw --force enable

    mkdir -p /var/www/myserver/html  
    chown -R www-data:www-data /var/www/myserver/html
    chmod -R 755 /var/www/myserver

    cp /vagrant/files/myserver /etc/nginx/sites-available/myserver
      
    cp /vagrant/files/error404.html /var/www/myserver/html
    cp /vagrant/files/index.html /var/www/myserver/html
    cp /vagrant/files/admin.html /var/www/myserver/html

    cp -r /vagrant/files/assets /var/www/myserver/html



    sudo openssl req -x509 -nodes -days 365 \
        -newkey rsa:2048 -keyout /etc/ssl/private/myserver.key \
        -out /etc/ssl/certs/myserver.crt \
       -subj "/C=ES/ST=Andalucia/L=GRX/O=IZV/OU=DAW/CN=myserver/emailAddress=adminr@myserver.com"
  
    ln -sf /etc/nginx/sites-available/myserver /etc/nginx/sites-enabled/myserver
    nginx -t
    systemctl restart nginx

    curl -sSL https://ngrok-agent.s3.amazonaws.com/ngrok.asc \
    | sudo tee /etc/apt/trusted.gpg.d/ngrok.asc >/dev/null \
    && echo "deb https://ngrok-agent.s3.amazonaws.com buster main" \
    | sudo tee /etc/apt/sources.list.d/ngrok.list \
    && sudo apt update \
    && sudo apt install ngrok

    ngrok config add-authtoken 2qFg0ntPWiXOG8iHCgClXLS0pmm_6xA3PSQcbN3t7vV1G6hiH
    nohup ngrok http --url=amazingly-proper-cod.ngrok-free.app 443 &

    sh -c "echo -n 'admin:' >> /etc/nginx/.htpasswd_admin"
    sh -c "openssl passwd -apr1 'asir' >> /etc/nginx/.htpasswd_admin"
    sh -c "echo -n 'sysadmin:' >> /etc/nginx/.htpasswd_status"
    sh -c "openssl passwd -apr1 'risa' >> /etc/nginx/.htpasswd_status"

    bash <(curl -sSL https://my-netdata.io/kickstart.sh) --dont-wait

  SHELL
end