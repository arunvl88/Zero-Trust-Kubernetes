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

