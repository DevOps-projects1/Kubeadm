# Kubeadm
Setup kubernetes cluster using kubeadm
# Installing Containerd, Kubectl, And Kubeadm Packages
 
  kubeadm – a CLI tool that will install and configure the various components of a cluster in a standard way.
  kubelet – a system service/program that runs on all nodes and handles node-level operations.
  kubectl – a CLI tool used for issuing commands to the cluster through its API Server.

# Step 1) take SSH of the server.
    $ sudo -i

# Step 2) Configure persistent loading of modules.
      tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

 # Step 3) Load at runtime.
      $ modprobe overlay
      $ modprobe br_netfilter
      
# Step 4) Update Iptables Settings.
Note: To ensure packets are properly processed by IP tables during filtering and port forwarding. Set the      
 net.bridge.bridge-nf-call-iptables to ‘1’ in your sysctl config file.
         
    $ tee /etc/sysctl.d/kubernetes.conf<<EOF
      net.bridge.bridge-nf-call-ip6tables = 1
      net.bridge.bridge-nf-call-iptables = 1
      net.ipv4.ip_forward = 1
      EOF

# Step 5) Applying Kernel Settings Without Reboot.
     $ sysctl --system

# Step 6) Adding Docker Repository
Adding Docker Repository GPG Key to Trusted Keys, so it allows the system to verify the integrity of the downloaded
Docker packages. 

    $ mkdir -m 0755 -p /etc/apt/keyrings
    $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

 # Step 7) Adding Docker Repository to Ubuntu Package Sources.
 Adding Docker Repository to Ubuntu Package Sources to enable the system to download and install Docker packages from         the specified repository.
 (Note: The below is a single command. Please copy and paste the command into a notepad first, then execute it.)
 
     echo \
     "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg]  
     https://download.docker.com/linux/ubuntu \
     $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null


# Step 8) Install containerd.
       $ apt-get update && apt-get install -y containerd.io

# Step 9) Configure containerd for Systemd Cgroup Management
 Configure containerd for Systemd Cgroup Management to enable the use of systemd for managing cgroups in containerd.

     $ mkdir -p /etc/containerd
     $ containerd config default>/etc/containerd/config.toml
     $ sed -e 's/SystemdCgroup = false/SystemdCgroup = true/g' -i /etc/containerd/config.toml

# Step 10) Reloading Daemon, Restarting, Enabling, and Checking containerd Service Status.

     $ systemctl daemon-reload
     $ systemctl restart containerd
     $ systemctl enable containerd 
     $ systemctl status containerd

# Step 11) Update the apt package index and install packages needed to use the Kubernetes https certificate configuration.

     $ apt-get update && apt-get install -y apt-transport-https ca-certificates curl

# Step 12)Download the GPG key,

Download the GPG key, as it is used to verify Kubernetes packages from the Kubernetes package repository, and store it in the kubernetes-apt-keyring.gpg file in the /etc/apt/keyrings/ directory.

     $curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.27/deb/Release.key | sudo gpg --dearmor -o     
      /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Step 13) Add the Kubernetes Repository to System’s Package Sources.
This command adds the Kubernetes repository to the system’s list of package sources by appending the repository information to the file.

   $ echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.27/deb/ /' |         sudo tee /etc/apt/sources.list.d/kubernetes.list


# Step 14)Install kubelet, kubeadm, kubectl packages

   $ apt-get update && apt-get install -y kubelet kubeadm kubectl

# Step 15)To hold the installed packages at their installed versions, use the following command. (Not Required).

    $ apt-mark hold kubelet kubeadm kubectl

# Step 16)Start the kubelet service is required on all the nodes.

   $ systemctl enable kubelet

### Create A Kubernetes Cluster
As we have successfully installed Kubeadm, next we will create a Kubernetes cluster using the following mentioned steps


## Step 1) We have to initialize kubeadm on the master node.
This command will check against the node that we have all the required dependencies. If it is passed, then it will install control plane components.
(Note: Run this command in Master Node only.)

    $ kubeadm init
    
 ![image](https://github.com/DevOps-projects1/Kubeadm/assets/104455878/8b888a85-4286-4a8f-8b79-7dcea7e9085b)

You will see a similar output:
![image](https://github.com/DevOps-projects1/Kubeadm/assets/104455878/a233038e-bfc6-4a18-bc64-310f5e54e117)


If cluster initialization has succeeded, then we will see a cluster join command. This command will be used by the worker nodes to join the Kubernetes cluster, so copy this command and save it for the future use.

## Step 2) To start using the cluster, we have to set the environment variable on the master node

     $ export KUBECONFIG=/etc/kubernetes/admin.conf
     $ echo 'export KUBECONFIG=/etc/kubernetes/admin.conf' >> .bashrc
     
 ![image](https://github.com/DevOps-projects1/Kubeadm/assets/104455878/3da0e5b4-7be3-4c05-8fa1-dc7b05c641e6)


## Join Worker Nodes to the Kubernetes Cluster
Now our Kubernetes master node is set up, we should join Worker nodes to our cluster. Perform the following same steps on all of the worker nodes:

## Step 1) SSH into the Worker node with the username and password.

     $ ssh <external ip of worker node>

## Step 2) Run the kubeadm join command that we have received and saved.
Note: This is above cluster command, you will get your command in your cluster so use that command not this command

     $ kubeadm join 10.1.0.4:6443 --token 9jp68n.1xw5sup0xpsf5mwk \
       --discovery-token-ca-cert-hash sha256:2e85f2d20cff1432051be4bd7800a57e9d6963bc664f1190e293152e99a6a12b

(Note: Don’t use this same command, use the command that you have received and saved while doing kubeadm init command.)

 ![image](https://github.com/DevOps-projects1/Kubeadm/assets/104455878/e1fa9c71-73af-4286-8a12-a2dd7d164328)

If you have forgotten to save the above received kubeadm join command, then you can create a new token and use it for joining worker nodes to the cluster.

     $ kubeadm token create --print-join-command
 
 ![image](https://github.com/DevOps-projects1/Kubeadm/assets/104455878/916e8d4b-6ddd-4468-a9d4-b20ef16e6945)


 ## Testing the Kubernetes Cluster
 
 After creating the cluster and joining worker nodes, we have to make sure that everything is working properly. To see and verify the cluster status, we can use kubectl command on the master node:
 
Using Kubectl get nodes command, we can see the status of our Nodes (master and worker) whether they are ready or not.

     $ kubectl get nodes

  ![image](https://github.com/DevOps-projects1/Kubeadm/assets/104455878/2babffe0-e6ed-4b0a-a82f-f716d848ffa8)


## We have to install CNI so that pods can communicate across nodes.
We have to install CNI so that pods can communicate across nod and also Cluster DNS to start functioning. Apply Weave CNI (Container Network Interface) on the master node.

     $ kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml

 ![image](https://github.com/DevOps-projects1/Kubeadm/assets/104455878/675213b4-2fe9-4264-9b4e-0cb67757fc0f)


Wait for a few minutes and verify the cluster status by executing kubectl command on the master node and see that nodes come to the ready state.

     $ kubectl get nodes

 ![image](https://github.com/DevOps-projects1/Kubeadm/assets/104455878/100c3afe-f73d-414b-a757-c1109ef08b73)


To verify the status of the system pods like coreDNS, weave-net, Kube-proxy, and all other master node system processes, use the following command:

    $ kubectl get pods -n kube-system 

 ![image](https://github.com/DevOps-projects1/Kubeadm/assets/104455878/2b68baaa-11d1-4138-9243-79288c73b582)

