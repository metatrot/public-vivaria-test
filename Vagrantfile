# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
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

    curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
      | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

    echo "deb [arch=$(dpkg --print-architecture) \
      signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
      https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) stable" \
      | tee /etc/apt/sources.list.d/docker.list > /dev/null

    apt-get update
    apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
  SHELL

  config.vm.provision "shell", privileged: false, inline: <<-SHELL
    mkdir ~/vivaria
    cd ~/vivaria
    
    curl -fsSL "#{base_url}/docker-compose.yml" -o docker-compose.yml
    curl -fsSL "#{base_url}/scripts/setup-docker-compose.sh" | bash -
    
    if [ -f /vagrant/api_keys.txt ]; then
      echo "Appending API keys to .env.server file..."
      cat /vagrant/api_keys.txt >> ~/vivaria/.env.server
    else
      echo "WARNING: No api_keys.txt file found!"
      echo "The server requires valid API keys to run AI agents."
      echo "You can use api_keys.txt.example as a template but must replace the keys with valid ones."
      echo "Continuing setup without API keys. You can still use the headless-human agent."
    fi
    
    sudo docker compose up --wait --detach --pull=always

    git clone https://github.com/METR/public-tasks.git --branch staging-2025-03-11-13-34-07 ~/public-tasks

    mkdir ~/agents
    git clone https://github.com/poking-agents/flock-public.git ~/agents/flock-public
    git clone https://github.com/poking-agents/modular-public.git ~/agents/modular-public
    git clone https://github.com/poking-agents/headless-human.git ~/agents/headless-human

    mkdir ~/vivaria-venv
    python3.12 -m venv ~/vivaria-venv
    source ~/vivaria-venv/bin/activate
    pip install "git+https://github.com/METR/vivaria.git@#{vivaria_version}#subdirectory=cli"
    curl -fsSL "#{base_url}/scripts/configure-cli-for-docker-compose.sh" | bash -
    
    viv run crossword/5x5_verify --task-family-path ~/public-tasks/crossword/ --agent-path ~/agents/modular-public/
  SHELL
end
