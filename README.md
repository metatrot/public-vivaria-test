# public-vivaria-test

Demo of running tasks from the [public-tasks](https://github.com/METR/public-tasks) repo on public agents from [poking-agents](https://github.com/poking-agents) in Vivaria. This demo uses Vagrant and is tested on Linux and Windows using VirtualBox as the VM backend.

## Steps to run the demo

1. Install VirtualBox from [here](https://www.virtualbox.org/wiki/Downloads)
2. Install Vagrant from [here](https://developer.hashicorp.com/vagrant/install)
3. Clone this repo
4. If you want to run LLM agents, put your keys in `api_keys.txt` inside the repo directory
5. Run `vagrant up` in the repo directory
6. Get `ACCESS_TOKEN` and `ID_TOKEN` from the generated `vivaria_tokens.txt` file that will appear after the VM starts
7. Go to [https://localhost:4000]() and put in the above values when asked

viv run crossword/5x5_verify --task-family-path ~/public-tasks/crossword/ --agent-path ~/agents/modular-public/