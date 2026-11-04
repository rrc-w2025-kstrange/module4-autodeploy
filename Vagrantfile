# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"  # Ubuntu 22.04 LTS
  config.vm.hostname = "kyra"

  # Network configuration
  config.vm.network "private_network", ip: "192.168.56.58"
  config.vm.network "forwarded_port", guest: 3000, host: 3000  # Student Portfolio
  config.vm.network "forwarded_port", guest: 5090, host: 5090  # Banking App
  config.vm.network "public_network"

  # VirtualBox provider settings
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = 2
  end

  # Provisioning script
  config.vm.provision "shell", inline: <<-SHELL
    # Update system and install prerequisites
    sudo apt-get update -y
    sudo apt-get install -y ca-certificates curl gnupg lsb-release git

    # Install Docker 
    sudo mkdir -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update -y
    sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

    # Start and enable Docker
    sudo systemctl enable --now docker

    # Add vagrant user to docker group
    sudo usermod -aG docker vagrant

    # Clone repositories
    git clone https://github.com/rrc-w2025-kstrange/Studentportfoliotemplate.git /home/vagrant/Studentportfoliotemplate
    git clone https://github.com/rrc-w2025-kstrange/BankingApp.git /home/vagrant/BankingApp

    # Create docker-compose.yml with auto-restart
    cat <<EOL > /home/vagrant/docker-compose.yml
version: '3.8'
services:
  studentportfolio:
    build: ./Studentportfoliotemplate
    ports:
      - "3000:80"
    hostname: studentportfolio
    container_name: studentportfolio
    restart: unless-stopped
    networks:
      - appnet

  bankingapp:
    build: ./BankingApp
    ports:
      - "5090:5090"
    hostname: bankingapp
    container_name: bankingapp
    restart: unless-stopped
    networks:
      - appnet

networks:
  appnet:
    driver: bridge
EOL

    # Fix permissions
    sudo chown -R vagrant:vagrant /home/vagrant

    # Build and start containers (with proper group permissions)
    sudo -u vagrant bash -c 'cd /home/vagrant && docker compose up -d --build'

    # Verify containers are running
    sudo -u vagrant docker compose ps
  SHELL
end