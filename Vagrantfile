# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  vivaria_version = "main"
  base_url = "https://raw.githubusercontent.com/METR/vivaria/#{vivaria_version}"

  config.vm.box = "ubuntu/jammy64"
  config.vm.network "forwarded_port", guest: 4002, host: 4000
  
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "4096"
    vb.cpus = 2
    vb.name = "vivaria-dev"
    
    # Añadir un disco adicional para XFS
    unless File.exist?('./xfs_disk.vdi')
      vb.customize ['createhd', '--filename', './xfs_disk.vdi', '--variant', 'Fixed', '--size', 60 * 1024]
    end
    vb.customize ['storageattach', :id, '--storagectl', 'SCSI', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', './xfs_disk.vdi']
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
      software-properties-common \
      xfsprogs
    
    # Configurar el disco XFS
    if [ ! -e /dev/sdc1 ]; then
      echo "Partitioning the new disk..."
      parted /dev/sdc mklabel gpt
      parted /dev/sdc mkpart primary xfs 0% 100%
    fi
    
    # Formatear con XFS si es necesario
    if ! blkid /dev/sdc1 | grep xfs; then
      mkfs.xfs -f /dev/sdc1
    fi
    
    # Crear directorio para Docker
    mkdir -p /var/lib/docker
    
    # Montar el sistema de archivos XFS con pquota
    if ! grep -q "/dev/sdc1" /etc/fstab; then
      echo "/dev/sdc1 /var/lib/docker xfs defaults,pquota 0 0" >> /etc/fstab
      mount /var/lib/docker
    fi

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
    
    # Configurar Docker para usar overlay2
    cat > /etc/docker/daemon.json <<EOF
{
  "storage-driver": "overlay2"
}
EOF
    
    systemctl restart docker
  SHELL

  config.vm.provision "shell", privileged: false, inline: <<-SHELL
    ssh-keygen -t ed25519 -a 100 -f ~/.ssh/id_ed25519 -N ""

    mkdir ~/vivaria
    cd ~/vivaria
    
    curl -fsSL "#{base_url}/docker-compose.yml" -o docker-compose.yml
    curl -fsSL "#{base_url}/scripts/setup-docker-compose.sh" | bash -
    
    if [ -f /vagrant/api_keys.txt ]; then
      cat /vagrant/api_keys.txt >> ~/vivaria/.env.server
    fi

    grep -E "ACCESS_TOKEN|ID_TOKEN" ~/vivaria/.env.server > /vagrant/vivaria_tokens.txt
    
    sudo docker compose up --wait --detach --pull=always

    git clone https://github.com/METR/public-tasks.git --branch staging-2025-03-11-13-34-07 ~/public-tasks

    mkdir ~/agents
    git clone https://github.com/poking-agents/flock-public.git ~/agents/flock-public
    git clone https://github.com/poking-agents/modular-public.git ~/agents/modular-public
    git clone https://github.com/poking-agents/headless-human.git ~/agents/headless-human

    python3.12 -m venv ~/venv
    source ~/venv/bin/activate
    pip install "git+https://github.com/METR/vivaria.git@#{vivaria_version}#subdirectory=cli"
    curl -fsSL "#{base_url}/scripts/configure-cli-for-docker-compose.sh" | bash -
    
    viv register_ssh_public_key ~/.ssh/id_ed25519.pub
  SHELL
end
