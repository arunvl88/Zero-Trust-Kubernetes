# Kubernetes-Cloudflared
Home Kubernetes lab exposed to public via Cloudflare Tunnel

<img width="1402" alt="Pasted Graphic" src="https://github.com/arunvl88/Kubernetes-Cloudflared/assets/7003647/37a2889e-f49a-46ee-b429-99395f083412">


| Hardware | Component | Explanation |
|---|---|---|
| Proxmox | Virtualization platform | Runs multiple operating systems, including Ubuntu, as virtual machines. |
| Kubernetes controller (on Ubuntu) | Master node | Manages the Kubernetes cluster and schedules container deployments. |
| K8 node-1 (on Ubuntu) | Worker node | Runs containerized applications instructed by the Kubernetes controller. |
| K8 node-2 (on Ubuntu) | Worker node | Another worker node to distribute the workload of containerized applications. |
| Pods (on K8 node-1 & K8 node-2) | Containers grouped together |  Logical units for deploying and managing containerized applications. |

Example: A pod might contain a web server container and a database container that work together to run a web application. In our case pod contains two containers nginx web server and cloudflared tunnel to expose the nginx web server publicly.

## Installing Proxmox VE on Dell Precision 3660 Server

Refer to the official Proxmox documentation (https://www.proxmox.com/en/proxmox-virtual-environment/get-started) for detailed and up-to-date installation instructions. The hardware server I'm using comes with 16Gig RAM and 1TB hard disk which is more than enough for home Kubernetes lab.

## Creating an Ubuntu Template and Building a Kubernetes Cluster (Referencing YouTube Videos)
This section outlines the process for creating a Ubuntu template and building a Kubernetes cluster on your Proxmox VE lab, referencing helpful YouTube videos. However, we'll delve deeper into specific commands and address any outdated information found in those videos.

## Important Note:

While YouTube videos can be a valuable resource, it's crucial to reference the official Kubernetes documentation (https://kubernetes.io/docs/home/) for the most up-to-date and reliable information.
Creating a Ubuntu Template:

1. Base Ubuntu Image: Begin by downloading a base Ubuntu image suitable for your Kubernetes cluster. Refer to the official Ubuntu documentation (https://cloud-images.ubuntu.com/) for available options.

2. Customization (Optional): You can customize the downloaded image by installing essential packages (e.g., Docker, kubectl) and configuring basic settings within the virtual machine using tools like apt.

3. Template Creation: Once prepared, convert the customized Ubuntu image into a Proxmox VE template for easy deployment of worker nodes in your Kubernetes cluster. Utilize the Proxmox VE web interface or command-line tools for template creation.

## Building a Kubernetes Cluster:

## YouTube Video References:

. Building a Ubuntu Template: https://www.youtube.com/watch?v=MJgIm03Jxdo by Learn Linux TV (https://www.youtube.com/@LearnLinuxTV)
. Building a Kubernetes Cluster: https://m.youtube.com/watch?v=Ro2qeYeisZQ (This video requires enabling desktop mode first)

## Building a Kubernetes Cluster on the Control Plane Node
This section details the steps to install and configure Kubernetes on the control plane node of your Proxmox VE lab. Be sure to replace v1.28.1 with the desired Kubernetes version if needed.

## Important Note:

While these steps provide a general guide, refer to the official Kubernetes documentation (https://kubernetes.io/docs/home/) for the most up-to-date and reliable information.

## 1. System Updates and Verification (Commands 1-7, 37):

* sudo apt update && sudo apt dist-upgrade (1): Updates the package lists and upgrades existing packages.
* cd /etc/netplan/ (3): Changes directory to the network configuration directory.
* ls (4): Lists files in the current directory (useful for verification).
* cat 50-cloud-init.yaml (5): Displays the contents of the cloud-init network configuration file (optional).
* cat /etc/hostname (6, 7): Shows the hostname of the machine (useful for verification).
* sudo apt update (37): Updates the package lists again (might be needed after adding new repositories).
Explanation:

This section ensures your system is up-to-date and verifies the hostname configuration.

## 2. Qemu Guest Agent Installation and Verification (Commands 8-12):

* sudo apt install qemu-guest-agent (8): Installs the Qemu Guest Agent for improved communication between the virtual machine and Proxmox VE.
* dpkg -l | grep qemu-guest-agent (9): Checks the installation status of the Qemu Guest Agent.
* sudo systemctl start qemu-guest-agent (10): Starts the Qemu Guest Agent service.
* sudo systemctl enable qemu-guest-agent (11): Enables the Qemu Guest Agent service to start automatically on boot.
* sudo systemctl status qemu-guest-agent (12): Checks the status of the Qemu Guest Agent service.
Explanation:

This section installs and configures the Qemu Guest Agent to enhance communication and monitoring within the Proxmox VE environment.

## 3. Containerd Installation and Configuration (Commands 13-18):

* sudo apt install containerd (13): Installs the containerd runtime, a core component for managing container images.
* systemctl status containerd (14): Checks the status of the containerd service.
* sudo mkdir /etc/containerd (15): Creates a directory for containerd configuration files (if it doesn't exist).
* containerd config default | sudo tee /etc/containerd/config.toml (16): Generates a default containerd configuration file.
* ls -l /etc/containerd/ (17): Lists files in the containerd configuration directory (useful for verification).
* sudo nano /etc/containerd/config.toml (18): Opens the containerd configuration file for editing (optional, advanced configuration).
Explanation:

This section installs and configures containerd, which is essential for running containerized applications like Kubernetes.

## 4. System Configuration for Kubernetes (Commands 22-26):

sudo nano /etc/sysctl.conf (22): Opens the system control file for editing (optional, advanced network configuration for Kubernetes).
cat /etc/sysctl.conf (23): Displays the contents of the system control file (optional, verification).
sudo nano /etc/modules-load.d/ (24): Changes directory to the kernel modules directory.
sudo nano /etc/modules-load.d/k8s.conf (25): Creates a new file named k8s.conf to load required kernel modules for Kubernetes (optional, might be pre-configured).
sudo reboot (26): Reboots the system to apply any configuration changes.
Explanation:

This section might involve optional configuration steps for the kernel and system controls to optimize the environment for Kubernetes.


## 5. Adding Kubernetes Repository and Installing Packages (Commands 38, 45-47, 51-52):

cat /etc/os-release (38): Displays the operating system release information (useful for verification).
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg (45): Downloads the GPG key to verify the authenticity of Kubernetes packages.
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list (46): Adds the official Kubernetes repository to your system's package sources list.
sudo apt update (47, 51): Updates the package lists to include Kubernetes packages after adding the new repository.
sudo apt install -y kubeadm=1.28.1-1.1 kubelet=1.28.1-1.1 kubectl=1.28.1-1.1 (52): Installs the required Kubernetes packages: kubeadm (for initializing the Kubernetes control plane), kubelet (for running containers on the node), and kubectl (for managing the Kubernetes cluster).



