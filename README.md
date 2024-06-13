# Kubeadm
Setup kubernetes cluster using kubeadm
## Installing Containerd, Kubectl, And Kubeadm Packages
 
  kubeadm – a CLI tool that will install and configure the various components of a cluster in a standard way.
  kubelet – a system service/program that runs on all nodes and handles node-level operations.
  kubectl – a CLI tool used for issuing commands to the cluster through its API Server.

# Step 1) 
    We have to do SSH to our virtual machines with the username and password. If you are a Linux or Mac user then use ssh           command   and if you are a Windows user then you can use Putty.

  $ sudo -i

##Step 2) Configure persistent loading of modules.
      $ tee /etc/modules-load.d/containerd.conf <<EOF
        overlay
        br_netfilter
        EOF
