# Ollama & Open-WebUI on Ryzentosh
Instructions for how to setup Ollama and Open WebUI on Ryzentosh (AMD Hackintosh). It was tested on MacOS Sonoma 14.5.

## Prerequisites
- Have [Homebrew](https://brew.sh) already installed.

## Instructions

### Part 1: Install Ollama
1. Download & Install [Ollama](https://ollama.com/download);
2. Install at least one language model (example: run `ollama run llama3` in Terminal);

(Optional) Test it out! go to [localhost:11434](http://localhost:11434). You should see "Ollama is running".

### Part 2: Install & setup Docker
Docker is a bit tricky to install on Ryzentosh, because MacOS doesn't support AMD's virtualization technologies AMD-V (previously knowns as SVM). Do you know who _does_ support AMD-V in MacOS? VirtualBox! That's right. I couldn't beleive it myself, but somehow, VirtualBox supports AMD-V even in MacOs. Funny, eh!

So, what we'll want to do here, is installing Docker telling it to use Virtual Box for virtualization rather than MacOS hypervisor (which doesn't support AMD). 


### Part 3: Install Open-WebUI in Docker
