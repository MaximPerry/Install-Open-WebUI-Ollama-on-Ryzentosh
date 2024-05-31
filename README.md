# Install Open-WebUI (Ollama) on Ryzentosh
Instructions on how to setup Ollama and Open WebUI on Ryzentosh (AMD Hackintosh). It was tested on MacOS Sonoma 14.5.

![Static Badge](https://img.shields.io/badge/Sonoma-14.5-blue) ![Static Badge](https://img.shields.io/badge/OpenCore-1.0.0-blue) ![GitHub License](https://img.shields.io/github/license/MaximPerry/Install-Open-WebUI-Ollama-on-Ryzentosh)


<img width="1920" alt="Screenshot 2024-05-30 at 11 13 46 PM" src="https://github.com/MaximPerry/Ollama-Open-WebUI-on-Ryzentosh/assets/28932114/28f7f8ea-0203-4c61-8e66-ec157c22e538">

## Prerequisites
- Have [Homebrew](https://brew.sh) already installed;
- Have AMD-V / VSM enabled in your BIOS.

</br>

## Instructions

</br>

### Part 1: Install Ollama
1. Download & Install [Ollama](https://ollama.com/download);
2. Install at least one language model (example: run `ollama run llama3` in Terminal);

(Optional) Test it out! go to [localhost:11434](http://localhost:11434). You should see "Ollama is running".

</br>

### Part 2: Install & setup Docker
Docker is a bit tricky to install on Ryzentosh, because MacOS doesn't support AMD's virtualization technologies AMD-V (previously SVM). Do you know who _does_ support AMD-V in MacOS? **VirtualBox**! That's right. I couldn't beleive it myself, but somehow, VirtualBox supports AMD-V even on MacOS. Funny, eh!

So, what we'll want to do here, is install Docker telling it to use VirtualBox for virtualization rather than MacOS's hypervisor (which doesn't support AMD). We can do that using a middleman app called **Minikube**.

****Note:** Commands bellow are run in Terminal.

---

1. Download and install **VirtualBox**, specifically version [6.1.40](https://download.virtualbox.org/virtualbox/6.1.40/VirtualBox-6.1.40-154048-OSX.dmg);

    Older versions don't work on Sonoma, and newer versions have a bug that will stop us in a later step. So this version is the sweet spot.
    ****Note:** After installation, you should be prompted to allow a system extension. Enable it, and reboot MacOS.

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
    Quit and re-open Terminal for changes to take effect.

5. (Optional) You can install Docker-Compose normally:
    ```
    brew install docker-compose
    ```
</br>

### Part 3: Install Open-WebUI in Docker
If you try to install **Open-WebUI** at this point using its [documentation](https://docs.openwebui.com), you'll see that it works, but it won't find your installed language model(s). This is because **Docker** (and therefor Open-WebUI) is unusually being run in a virtual machine (VM), and so it's trying to find Ollama installed on the VM too. We need to setup Open-WebUI so that it looks for Ollama on Host instead of on its virtual disk.

To do this, we need to slightly customize Open-WebUI's default setup command.

<details>
  
  <summary>Explanation (optional)</summary>
  
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

</br>

## Post installation (optional)

### Access Open-WebUI using localhost:3000

If you want to be able to access Open-WebUI using localhost again as you would with a normal installation of Docker:
1. Launch Virtual Box;
2. Right-click on "minikube" > Settings;
3. In the "Network" tab, click on Advanced > Port Forwarding
4. Add a new entry with the following settings:

| Name | Protocol | Host IP | Host Port | Guest IP | Guest Port |
| --- | --- | --- | --- | --- | --- |
| Open-WebUI | TCP | 127.0.0.1 | 3000 |  | 3000 |

<details>
  
  <summary>Explanation (optional)</summary>
  
Open-WebUI is runing on the VM's localhost:3000, also known as 127.0.0.1:3000. So we tell VirtualBox to expose the VM's port 3000 as our own (Host). 
- **Name**: This can be the name of your choise. I chose "Open-WebUI", but you can rename it.
- **Protocol**: There are two types of protocols: UDP, and TCP. We have to match the protocol that the application we're port forwarding is using. In our case, Open-WebUI uses TCP.
- **Host IP**: This can be misleading, I admit... We'd be tempted to insert the IP of our hackingtosh, but that wouldn't work. This entry is not asking for the Host's IP; it's asking: "As Host, what IP do you want to uset o access Open-WebUI?". We want to use localhost / 127.0.0.1.
- **Host Port**: This is the port that will be used to access Open-WebUI. We put `3000` here so that it matches Open-WebUI's documentation, but you can use the port of your choice. If you put "1234" for example, you'll have to use `localhost:1234` or `127.0.0.1:1234` to access the app instead of the default `localhost:3000` / `127.0.0.1:3000`.
- **Guest IP**: From the VM's perspective, which IP would we use to access Open-WebUI? By default, it's localhost, so we can leave it blank. 
- **Guest Port**: This has to match the port where Open-WebUI is actually runing, for the VM's perspective. In our original docker install command, we told it to use port 3000.
  
</details>

You should now be able to access Open-WebUI using [localhost:3000](http://localhost:3000)!
