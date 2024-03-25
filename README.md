# Kubernetes-Cloudflared
## Setting Up a Home Kubernetes Lab with Nginx and Cloudflare Tunnel

This guide outlines the steps to build a home Kubernetes lab environment on a Dell server with 16 GB of RAM. You'll deploy an Nginx web server container and configure it to be accessible externally through a Cloudflare Tunnel.

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
## Prerequisites:

Ubuntu Virtual Machines: It's assumed you have created Ubuntu virtual machines for your Kubernetes cluster nodes. You can refer to the following YouTube videos

## YouTube Video and article References:

* Building a Ubuntu Template: https://www.youtube.com/watch?v=MJgIm03Jxdo by Learn Linux TV (https://www.youtube.com/@LearnLinuxTV)

* Building a Kubernetes Cluster: https://www.youtube.com/watch?v=U1VzcjCB_sY 

* https://www.learnlinux.tv/how-to-build-an-awesome-kubernetes-cluster-using-proxmox-virtual-environment/

## Building a Kubernetes Cluster:

## Building a Kubernetes Cluster on the Control Plane Node
This section details the steps to install and configure Kubernetes on the control plane node of your Proxmox VE lab. Be sure to replace v1.28.1 with the desired Kubernetes version if needed.

## Important Note:

While these steps provide a general guide, refer to the official Kubernetes documentation (https://kubernetes.io/docs/home/) for the most up-to-date and reliable information.

## 1. System Updates and Verification:

* `sudo apt update && sudo apt dist-upgrade` (1): Updates the package lists and upgrades existing packages.
* `cd /etc/netplan/` (3): Changes directory to the network configuration directory.
* `ls` (4): Lists files in the current directory (useful for verification).
* `cat 50-cloud-init.yaml` (5): Displays the contents of the cloud-init network configuration file (optional).
* `cat /etc/hostname` (6, 7): Shows the hostname of the machine (useful for verification).
* `sudo apt update` (37): Updates the package lists again (might be needed after adding new repositories).

Explanation:

This section ensures your system is up-to-date and verifies the hostname configuration.

## 2. Qemu Guest Agent Installation and Verification:

* `sudo apt install qemu-guest-agent` (8): Installs the Qemu Guest Agent for improved communication between the virtual machine and Proxmox VE.
* `dpkg -l | grep qemu-guest-agent` (9): Checks the installation status of the Qemu Guest Agent.
* `sudo systemctl start qemu-guest-agent` (10): Starts the Qemu Guest Agent service.
* `sudo systemctl enable qemu-guest-agent` (11): Enables the Qemu Guest Agent service to start automatically on boot.
* `sudo systemctl status qemu-guest-agent` (12): Checks the status of the Qemu Guest Agent service.
  
Explanation:

This section installs and configures the Qemu Guest Agent to enhance communication and monitoring within the Proxmox VE environment.

## 3. Containerd Installation and Configuration:

* `sudo apt install containerd` (13): Installs the containerd runtime, a core component for managing container images.
* `systemctl status containerd` (14): Checks the status of the containerd service.
* `sudo mkdir /etc/containerd` (15): Creates a directory for containerd configuration files (if it doesn't exist).
* `containerd config default | sudo tee /etc/containerd/config.toml` (16): Generates a default containerd configuration file.
* `ls -l /etc/containerd/` (17): Lists files in the containerd configuration directory (useful for verification).
* `sudo nano /etc/containerd/config.toml` (18): Opens the containerd configuration file for editing (optional, advanced configuration).
  Within that file, find the following line of text:
  
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
Underneath that, find the SystemdCgroup option and change it to true, which should look like this:

SystemdCgroup = true

Explanation:

This section installs and configures containerd, which is essential for running containerized applications like Kubernetes.

## 4. System Configuration for Kubernetes:

* `sudo nano /etc/sysctl.conf` (22): Opens the system control file for editing (optional, advanced network configuration for Kubernetes).
  Within that file, look for the following line:

#net.ipv4.ip_forward=1
Uncomment that line by removing the # symbol in front of it, which should make it look like this:

net.ipv4.ip_forward=1
* `cat /etc/sysctl.conf` (23): Displays the contents of the system control file (optional, verification).
* `sudo nano /etc/modules-load.d/` (24): Changes directory to the kernel modules directory.
* `sudo nano /etc/modules-load.d/k8s.conf` (25): Creates a new file named k8s.conf to load required kernel modules for Kubernetes (optional, might be pre-configured).
* `sudo reboot` (26): Reboots the system to apply any configuration changes.

Explanation:

This section might involve optional configuration steps for the kernel and system controls to optimize the environment for Kubernetes.


## 5. Adding Kubernetes Repository and Installing Packages:

* `curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg` (45): Downloads the GPG key to verify the authenticity of Kubernetes packages.
* `echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list` (46): Adds the official Kubernetes repository to your system's package sources list.
* `sudo apt update` (47, 51): Updates the package lists to include Kubernetes packages after adding the new repository.
* `sudo apt install -y kubeadm=1.28.1-1.1 kubelet=1.28.1-1.1 kubectl=1.28.1-1.1` (52): Installs the required Kubernetes packages: kubeadm (for initializing the Kubernetes control plane), kubelet (for running containers on the node), and kubectl (for managing the Kubernetes cluster).

## 6. Initialize our Kubernetes cluster
So long as you have everything complete so far, you can initialize the Kubernetes cluster now. Be sure to customize the first IP address shown here (not the second) and also change the name to match the name of your controller.

`sudo kubeadm init --control-plane-endpoint=10.0.0.145 --node-name k8s-ctrlr --pod-network-cidr=10.244.0.0/16`
After the initialization finishes, you should see at least four commands printed within the output.

* `sudo`: Initiates the command with elevated privileges for administrative tasks.
* `kubeadm init`: The core command used to initialize a Kubernetes cluster on the current node.
* `--control-plane-endpoint`=10.0.0.145:
  * Sets the API server endpoint, which is the publicly reachable address worker nodes will use to communicate with the control plane.
  * In this example, it's set to 10.0.0.145. This could be the master node's public IP address, a hostname resolving to the IP, or a load balancer address for high availability setups.
* `--node-name=k8s-ctrlr`:
Assigns a unique hostname (k8s-ctrlr) to this node within the cluster. This hostname helps identify the control plane node among other cluster members.
* `--pod-network-cidr=10.244.0.0/16`:
  * Defines the Pod Network CIDR (Cluster IP Range) used for allocating IP addresses to pods within the cluster.
  * Here, the CIDR block is set to 10.244.0.0/16, indicating that pods will be assigned IP addresses from this range.

## Worker Nodes configuration for Kubernetes Cluster
Once you've initialized the Kubernetes control plane on your master node, you can add worker nodes to scale your cluster. Repeat these steps on all nodes. Here's a breakdown of the steps to prepare and join a worker node running on Proxmox VE:

## 1. System Updates and Verification:

* `sudo apt install qemu-guest-agent` (6): Installs the Qemu Guest Agent for improved communication with Proxmox VE. You might have already installed this on the control plane, but it's a good practice to ensure it's also present on worker nodes.
* `sudo apt install containerd` (17): Installs containerd, a core component for managing container images on the worker node.

## 2. Qemu Guest Agent Service Management:

* `sudo systemctl start qemu-guest-agent` (14): Starts the Qemu Guest Agent service on the worker node.
* `sudo systemctl enable qemu-guest-agent` (15): Enables the Qemu Guest Agent service to start automatically on boot.
* `sudo systemctl status qemu-guest-agent` (16): Checks the status of the Qemu Guest Agent service to ensure it's running properly.
## 4. Containerd Configuration:

* `sudo mkdir /etc/containerd` (19): Creates a directory for containerd configuration files (if it doesn't exist).
* `containerd config default | sudo tee /etc/containerd/config.toml` (20): Generates a default containerd configuration file.
* Optional: You might need to edit the configuration file (`sudo nano /etc/containerd/config.toml` - command 22) for specific configurations, but the default settings usually work well.
## 5. System Configuration for Kubernetes:

* Optional: You might need to adjust some system configurations for optimal Kubernetes performance. These commands are for verification and potential adjustments:
* sudo nano /etc/sysctl.conf (26): Opens the system control file for editing (advanced configuration).
  Within that file, look for the following line:

#net.ipv4.ip_forward=1
Uncomment that line by removing the # symbol in front of it, which should make it look like this:

`net.ipv4.ip_forward=1`

* `cat /etc/sysctl.conf | grep "ipv4"` (27): Shows lines related to IPv4 configuration in the system control file (optional, verification).
* `sudo nano /etc/modules-load.d/k8s.conf` (28): Opens the kernel modules configuration file for editing (advanced configuration, might be pre-configured).
## 6. Worker Node Rebooting:

`sudo reboot` (29): Depending on your system configuration, a reboot might be necessary for the changes to take effect. Uncomment this command if needed.
## 7. Adding Kubernetes Repository and Installing Packages:

* These commands add the official Kubernetes repository and download the GPG key to verify the authenticity of Kubernetes packages. The steps might differ slightly depending on your chosen Kubernetes provider (here, Google Kubernetes Engine - GKE). Refer to the official documentation for your specific provider for exact commands.
* `curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg` (45): Downloads the GPG key to verify the authenticity of Kubernetes packages.
* `echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list` (46): Adds the official Kubernetes repository to your system's package sources list.
* `sudo apt update` (47, 51): Updates the package lists to include Kubernetes packages after adding the new repository.
* `sudo apt install -y kubeadm=1.28.1-1.1 kubelet=1.28.1-1.1 kubectl=1.28.1-1.1` (52): Installs the required Kubernetes packages: kubeadm (for initializing the Kubernetes control plane), kubelet (for running containers on the node), and kubectl (for managing the Kubernetes cluster).


## Adding Worker Nodes to Your Kubernetes Cluster
Once you've initialized the Kubernetes control plane on your master node, you can add worker nodes to scale your cluster. Here's a breakdown of the steps to prepare and join a worker node running on Proxmox VE:

## 1. Generate a Join Token on the Control Plane (on Kubernetes controller):

Use the following command on the control plane node to generate a bootstrap token for worker nodes to join the cluster:

`kubeadm token create --print-join-command`

This command will generate a random token and display the full kubeadm join command that needs to be run on each worker node.

## 2. Joining the Cluster on Worker Nodes:

Copy the command generated in step 1 and run it on each nodes. The command will look like below:

`kubeadm join --control-plane --token <your_worker_token> <control-plane-ip>:6443 --discovery-token-ca-cert-hash sha256:<long_hash_value>`

## Exposing Your Nginx Deployment with Cloudflare Tunnel on a Kubernetes Cluster
This guide details how to install Cloudflare Tunnel on your Kubernetes cluster and configure it to expose your Nginx deployment publicly through Cloudflare's network.

## Prerequisites:

A functional Kubernetes cluster with Nginx deployed as an Ingress controller.
A Cloudflare account with a registered domain name.
kubectl configured to interact with your Kubernetes cluster.

## Steps:

## 1. Create a new tunnel on Cloudflare Dashboard:

1. Create a tunnel using this guide: https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/create-remote-tunnel/
2. Instead of running the command to install a connector you will select docker as the environment and copy just the token rather than the whole command.

## 2. Deploy Nginx with Cloudflare Tunnel (file attached in this repository):

Deploy the provided nginx-cloudflared.yaml file using kubectl:
`kubectl apply -f nginx-cloudflared.yaml`

I copied yaml file to the controller node and ran the command in controller. 

## Explanation:

This YAML file defines two resources for running an Nginx container with a Cloudflare tunnel in a Kubernetes cluster:

### 1. Deployment:
      This deployment creates pods with two containers:

      * Nginx: This container runs the Nginx web server, serving content on port 80.
      * Cloudflare tunnel injector: This container injects the Cloudflare tunnel functionality using the cloudflared command and your tunnel token (replace "tunnel-token" with     
        your actual token).
### 2. Service:
      This service exposes the Nginx container running inside the pods through a NodePort. The NodePort allows external traffic to reach the Nginx container from outside the   
      Kubernetes cluster using a randomly assigned port on a worker node.

### How it Works:

1. The deployment creates pods with the Nginx and Cloudflare tunnel containers.
2. The Cloudflare tunnel container establishes a tunnel to your Cloudflare account.
3. The Service exposes the Nginx container on a NodePort.
4. You can access your Nginx application from outside the cluster using the following format: <Node IP Address>:<NodePort>. Replace <Node IP Address> with the IP address of one of your worker nodes and <NodePort> with the actual port number assigned by Kubernetes (obtained using kubectl get service nginx-with-tunnel-service).

## 3. Add a route on the Cloudflare Tunnel Dashboard:
Login to Cloudflare Zero Trust dashboard > Network > Tunnels > Click on the 3 dot tab of the tunnel entry > select 'configure'> Navigate to 'Public Hostname' > Add 'Public Hostaname'

<img width="1703" alt="image" src="https://github.com/arunvl88/Kubernetes-Cloudflared/assets/7003647/71e2dd16-4722-4043-bf9d-092906b17c33">


`<Node IP Address>:<NodePort>`

* Replace <Node IP Address> with the IP address of one of your Kubernetes worker nodes (obtained from your cloud provider console or using kubectl get nodes).
* Replace <NodePort> with the actual NodePort value you obtained from the output. You can get a NodePort by running the following command:
  `kubectl get service nginx-with-tunnel-service`
  


