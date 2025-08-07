# This begins the trial run of k3s on my 16gb storage Arch Linux chromebook.

# Date: August 6th, 2025

# The goal is to get a single container running using k3s.

# I'm using ChatGPT to guide me through most of this process, I'll be using YouTube as well, and put resources used in this file.

# Starting from the relatively fresh install of Arch Linux on this computer, all that's really functioning on the computer is alacritty, x3, and firefox.

# Downloading k3s:
curl -sfL https://get.k3s.io | sh -

# Chat reccomends checking if k3s is running. The command did not install fully?

# Running
sudo systemctl status k3s
# returns that the sercice is loaded but that one or more of the processes seems to have failed.

# Attempting to restart k3s with command
sudo systemctl start k3s
# after a few minutes of waiting, nothing has happened. ^C and I'm going to sleep.

---

