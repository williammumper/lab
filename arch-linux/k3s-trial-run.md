## This begins the trial run of k3s on my 16gb storage Arch Linux chromebook.

## Date: August 6th, 2025

## The goal is to get a single container running using k3s.

## I'm using ChatGPT to guide me through most of this process, I'll be using YouTube as well, and put resources used in this file.

## Starting from the relatively fresh install of Arch Linux on this computer, all that's really functioning on the computer is alacritty, x3, and firefox.

## Downloading k3s:
curl -sfL https://get.k3s.io | sh -

## Chat reccomends checking if k3s is running. The command did not install fully?

## Running

sudo systemctl status k3s

## returns that the sercice is loaded but that one or more of the processes seems to have failed.

## Attempting to restart k3s with command

sudo systemctl start k3s

## after a few minutes of waiting, nothing has happened. ^C and I'm going to sleep.

---

# Aug 7th 2025

## After restarting the system and checking status on k3s, it is running. Chat reccomends I check the nodes with

sudo k3s kubectl get nodes

## This returns error unable to read /etc/rancher/k3s/k3s.yaml . Checking that the file exists

sudo ls /etc/rancher/k3s/k3s.yaml

## Returns that the file exists. Then, copying it to the user to run with kubectl

mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER ~/.kube/config

## This is done to enable kubectl to talk to k3s without needing sudo every time... kubectl looks for ~/.kube/config in order to talk to the cluster. Copying the k3s.yaml file to the config skips needing to connect the two each time. Then, using chown, we allow the user to access the config.

## This still errored, need to start server using --write-kubeconfig-mode. Chat reccomends running:

curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--write-kubeconfig-mode 644" sh -

## After running this command,

kubectl get nodes 

## functioned as expected, showing control-plane/master node. The curl script here was essential as it reinstalled k3s and initialized the readable version of kubeconfig..?

# k3s is now running a single node! 

---

## Chat also explains: the k3s.yaml file was created on first install, but only readable by root. So copying to ~/.kube/config didn't work because it was still not readable by the non-root user. After re-installing with --write-kubeconfig-mode 644, the non-root user is able to see the nodes properly.

# Lesson: install k3s with proper permissions to ensure it is readable by root user.