---
title: "gpu"
date: 2018-10-13 16:06
---

 - [Install NVIDIA Driver and CUDA.md](https://gist.github.com/wangruohui/df039f0dc434d6486f5d4d098aa52d07)
sudo apt-get purge nvidia*

# Note this might remove your cuda installation as well
sudo apt-get autoremove 

# Recommended if .deb files from NVIDIA were installed
# Change 1404 to the exact system version or use tab autocompletion
# After executing this file, /etc/apt/sources.list.d should contain no files related to nvidia or cuda
sudo dpkg -P cuda-repo-ubuntu1404