# Kubeadm
Setup kubernetes cluster using kubeadm
## Installing Containerd, Kubectl, And Kubeadm Packages
 
  kubeadm – a CLI tool that will install and configure the various components of a cluster in a standard way.
  kubelet – a system service/program that runs on all nodes and handles node-level operations.
  kubectl – a CLI tool used for issuing commands to the cluster through its API Server.

# Step 1) take SSH of the server.
    $ sudo -i

## Step 2) Configure persistent loading of modules.
      $ tee /etc/modules-load.d/containerd.conf <<EOF
        overlay
        br_netfilter
        EOF

 ## Step 3) Load at runtime.
      $ modprobe overlay
      $ modprobe br_netfilter
## Step 4) Update Iptables Settings.
    Note: To ensure packets are properly processed by IP tables during filtering and port forwarding. Set the      
         net.bridge.bridge-nf-call-iptables to ‘1’ in your sysctl config file.
    $ tee /etc/sysctl.d/kubernetes.conf<<EOF
      net.bridge.bridge-nf-call-ip6tables = 1
      net.bridge.bridge-nf-call-iptables = 1
      net.ipv4.ip_forward = 1
      EOF

## Step 5) Applying Kernel Settings Without Reboot.
    $ sysctl --system

## Step 6) Adding Docker Repository
   Adding Docker Repository GPG Key to Trusted Keys, so it allows the system to verify the integrity of the downloaded    
    Docker packages.
   $ mkdir -m 0755 -p /etc/apt/keyrings
   $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

 ## Step 7) Adding Docker Repository to Ubuntu Package Sources.
   Adding Docker Repository to Ubuntu Package Sources to enable the system to download and install Docker packages from         the specified repository.
 (Note: The below is a single command. Please copy and paste the command into a notepad first, then execute it.)

 echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null


## Step 8) Install containerd.
