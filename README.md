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

# Important Note:

While YouTube videos can be a valuable resource, it's crucial to reference the official Kubernetes documentation (https://kubernetes.io/docs/home/) for the most up-to-date and reliable information.
Creating a Ubuntu Template:

1. Base Ubuntu Image: Begin by downloading a base Ubuntu image suitable for your Kubernetes cluster. Refer to the official Ubuntu documentation (https://cloud-images.ubuntu.com/) for available options.

2. Customization (Optional): You can customize the downloaded image by installing essential packages (e.g., Docker, kubectl) and configuring basic settings within the virtual machine using tools like apt.

3. Template Creation: Once prepared, convert the customized Ubuntu image into a Proxmox VE template for easy deployment of worker nodes in your Kubernetes cluster. Utilize the Proxmox VE web interface or command-line tools for template creation.

# Building a Kubernetes Cluster:

# YouTube Video References:

. Building a Ubuntu Template: https://www.youtube.com/watch?v=MJgIm03Jxdo by Learn Linux TV (https://www.youtube.com/@LearnLinuxTV)
. Building a Kubernetes Cluster: https://m.youtube.com/watch?v=Ro2qeYeisZQ (This video requires enabling desktop mode first)
