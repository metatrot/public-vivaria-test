# Public Vivaria Test

Demo of running tasks from the [public-tasks](https://github.com/METR/public-tasks) repo on public agents from [poking-agents](https://github.com/poking-agents) in Vivaria. This demo uses Vagrant and has been tested to work on Linux and Windows using VirtualBox as the VM backend.


## Steps to run the demo

1. Install VirtualBox from [here](https://www.virtualbox.org/wiki/Downloads)
2. Install Vagrant from [here](https://developer.hashicorp.com/vagrant/install)
3. Clone this repo
4. In order to start agent runs, put your keys in `api_keys.txt` inside the repo directory
5. Run `vagrant up` in the repo directory
6. Get `ACCESS_TOKEN` and `ID_TOKEN` from `vivaria_tokens.txt` that will appear after the VM starts
7. Go to [https://localhost:4000](https://localhost:4000) and put in the above values when asked


### Running an AI agent

1. Run `vagrant ssh` from the repo directory.
2. Run `source ~/venv/bin/activate` inside the VM.
3. Start an agent running on a task, for example:

```bash
viv run crossword/5x5_verify --task-family-path ~/public-tasks/crossword/ --agent-path ~/agents/modular-public/ --max-tokens 1000
```

This will print out a link you can connect to in your browser.


### Manually running a task

1. Run `vagrant ssh` from the repo directory.
2. Run `source ~/venv/bin/activate` inside the VM.
3. Start a task using the `headless-human` agent, for example:

```bash
viv run debug_small_libs/markdown --task-family-path ~/public-tasks/debug_small_libs/ --agent-path ~/agents/headless-human/
```

(Note: unfortunately, a valid API key is still needed, even though it won't be used)

This will print out a run number. SSH into the run (while already inside of the VM):

```bash
viv ssh 3 --user agent  # replace 3 with your run number
```

If you ever want to access a run container as the root user, just omit the `--user agent` part.


## Breakdown of the Vagrantfile

The `Vagrantfile` has two blocks of code, one executed as root and the other executed as the user. You can ignore all of the first block of code. The second block of code closely corresponds to how you would set up Vivaria and run agents on your own host OS. See [here](https://vivaria.metr.org/tutorials/set-up-docker-compose/) for a more in-depth tutorial for setting up Vivaria locally.

In brief, the process is:

```bash
# Download the docker compose file and generate settings and secret tokens
vivaria_version="main"
base_url="https://raw.githubusercontent.com/METR/vivaria/$vivaria_version"
curl -fsSL "$base_url/docker-compose.yml" -o docker-compose.yml
curl -fsSL "$base_url/scripts/setup-docker-compose.sh" | bash -

# Set your API keys here to call LLMs
cat api_keys.txt >> ~/vivaria/.env.server

# Start Vivaria
sudo docker compose up --wait --detach --pull=always

# Get tasks
git clone https://github.com/METR/public-tasks

# Get an agent
git clone https://github.com/poking-agents/modular-public

# Create a virtual environment for the Vivaria CLI program
python3.12 -m venv ~/venv
source ~/venv/bin/activate
pip install "git+https://github.com/METR/vivaria.git@${vivaria_version}#subdirectory=cli"
curl -fsSL "${base_url}/scripts/configure-cli-for-docker-compose.sh" | bash -

# Set your SSH public key so that you can connect to run containers
viv register_ssh_public_key ~/.ssh/id_ed25519.pub
```

## Issues

The VM generates new secret tokens each time it's run, but your browser will cache the last entered ones. Make sure to clear the local storage.

If the queue gets stuck:
1. Connect to the VM using `vagrant ssh`
2. Kill the active runs using `viv kill`, after activating the `~/venv` as shown above
3. Restart the containers using `cd ~/vivaria && sudo docker compose restart`