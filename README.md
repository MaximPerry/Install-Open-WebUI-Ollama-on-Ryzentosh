
<img width="1920" alt="Screenshot 2024-05-30 at 11 13 46 PM" src="https://github.com/MaximPerry/Ollama-Open-WebUI-on-Ryzentosh/assets/28932114/28f7f8ea-0203-4c61-8e66-ec157c22e538">


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
Docker is a bit tricky to install on Ryzentosh, because MacOS doesn't support AMD's virtualization technologies AMD-V (previously SVM). Do you know who _does_ support AMD-V in MacOS? **VirtualBox**! That's right. I couldn't beleive it myself, but somehow, VirtualBox supports AMD-V even on MacOS. Funny, eh!

So, what we'll want to do here, is install Docker telling it to use VirtualBox for virtualization rather than MacOS's hypervisor (which doesn't support AMD). 

**Note:** Commands bellow are ran in Terminal.

---

1. Download and install **VirtualBox**, specifically version [6.1.40](https://download.virtualbox.org/virtualbox/6.1.40/VirtualBox-6.1.40-154048-OSX.dmg);

    Older versions don't work on Sonoma, and newer versions have a bug that will stop us in a later step. So this version is the sweet spot.

2. Install **Minikube** and **Docker**:
    ```
    brew install minikube docker
    ```

3. Setup **Minikube** to use the **VirtualBox** driver:
    ```
    minikube start --driver=virtualbox --keep-context
    ```

4. Tell docker to use Minikube's Docker (this way, `docker` commands will run as `Minikube docker` commands)...

    Option 1 - None-permanantly (in current terminal only):
    ```
    eval $(minikube docker-env)
    ```

    Option 2 - Permanently:
    ```
    open ~/.zshrc
    ```
    Add `eval $(minikube docker-env)` at the end of the file. Save & close.

5. Close and re-open Terminal for changes to take effect.

    (Optional) You can install Docker-Compose normally:
    ```
    brew install docker-compose
    ```

### Part 3: Install Open-WebUI in Docker
If you try to install **Open-WebUI** at this point using its [documentation](https://docs.openwebui.com), you'll see that it works, but it won't find your installed language model(s). This is because **Docker** (and therefor Open-WebUI) is unusually being run in a virtual machine (VM), and so it's trying to find Ollama installed on the VM too. We need to setup Open-WebUI so that it looks for Ollama on Host instead of on its virtual disk.

To do this, we need to slightly customize Open-WebUI's default setup command.

<details>
  
  <summary>Explanation</summary>
  
  Default command (don't run this): 
  
  ```
  docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
  ```
  \
  We'll want to add two variables:
  
  `-e OLLAMA_BASE_URL=http://10.0.2.2:11434`:\
  In a VM, `10.0.2.2` is actually the Host's IP. So we're now telling Docker to look outside of the VM for Ollama and the language models.
  
  `-e WEBUI_AUTH=False`:\
  Optional - this is more of a personal preference. By default, Open-WebUI enforces the creation and use of accounts, forcing you to login every time. As I'm the only one who's going to run Ollama on my RyzenTosh, I want to disable that - you probably want that too. 

</details>

Here's the command that we want to run:

```
docker run -d -p 3000:8080 -e WEBUI_AUTH=False -e OLLAMA_BASE_URL=http://10.0.2.2:11434 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```
    
And voilà! Everything should be installed and setup. 

---

You can now access Open-WebUI using Minikube's IP adress, followed by port 3000. You can find it by runing `minikube ip` in Terminal.
In my case, it's `http://192.168.59.106:3000`.



