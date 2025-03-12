# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  # Define external variables
  vivaria_version = "main"
  base_url = "https://raw.githubusercontent.com/METR/vivaria/#{vivaria_version}"

  config.vm.box = "ubuntu/jammy64"
  config.vm.network "forwarded_port", guest: 4000, host: 4000
  
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "4096"
    vb.cpus = 2
    vb.name = "vivaria-dev"
  end
  
  config.vm.provision "shell", inline: <<-SHELL
    export DEBIAN_FRONTEND=noninteractive
    
    apt-get update
    apt-get upgrade -y
    
    apt-get install -y \
      apt-transport-https \
      ca-certificates \
      curl \
      gnupg \
      lsb-release \
      software-properties-common

    add-apt-repository -y ppa:deadsnakes/ppa
    apt-get update
    apt-get install -y python3.12 python3.12-venv python3.12-dev python3-pip

    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
    apt-get update
    apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
  SHELL

  config.vm.provision "shell", privileged: false, inline: <<-SHELL
    mkdir /home/vagrant/vivaria
    cd /home/vagrant/vivaria
    
    curl -fsSL "#{base_url}/docker-compose.yml" -o docker-compose.yml
    curl -fsSL "#{base_url}/scripts/setup-docker-compose.sh" | bash -
    
    sudo docker compose up --wait --detach --pull=always

    mkdir /home/vagrant/vivaria-venv
    python3.12 -m venv /home/vagrant/vivaria-venv
    source /home/vagrant/vivaria-venv/bin/activate
    pip install "git+https://github.com/METR/vivaria.git@#{vivaria_version}#subdirectory=cli"
    curl -fsSL "#{base_url}/scripts/configure-cli-for-docker-compose.sh" | bash -
  SHELL
end
