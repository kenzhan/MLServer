# Private ML Server setup and configuration 
# (Ubuntu 20.4 + CUDA 10.1 + TensorFlow 2.3 + JupyterNotebook)

These are steps to setup private ML server. The goal is to setup a dedicated private machine learning server with deep learning capability and Jupyter Notebook server as web interface for personal ML projects. The configuration is for single user on local network scenario. As I don't want to spend too much time on server setup, all versions I picked are latest supported version without custom build.

- Ubuntu 20.4 LTS
- CUDA 10.1
- cnDDN 7.6.5
- TensorFlow 2.3
- JupyterNotebook

If you want the latest CUDA 11, cuDNN 8.0 with local build from TensorFlow source you can follow this [page](https://gist.github.com/kmhofmann/cee7c0053da8cc09d62d74a6a4c1c5e4).

## Hardware config


### Setup RAID (optional)

* Enable RAID

Some RAID configed under BIOS with so called [Fake RAID](https://help.ubuntu.com/community/FakeRaidHowto). This is to use software RAID for RAID-0 type instead.


[How To Create RAID Arrays with mdadm on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-create-raid-arrays-with-mdadm-on-ubuntu-16-04)

```

lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT

sudo apt-get install mdadm

sudo mdadm --create --verbose /dev/md0 --level 0 --raid-devices=2 /dev/sdb /dev/sdc
```

## Software config

### Ubuntu LTS

Download latest Ubuntu 20.04 LTS iso from https://releases.ubuntu.com/20.04/. I took desktop version at this moument as it comes with graphics interface.

Use [rufus](https://rufus.ie/) from  to burn the iso file to an USB drive for installation.

Plugin the USB drive and set boot from the USB in BIOS to get Linux installed.

### Server Setup

Scenarios are different for how the server will be used. My scenario is a single JupyterNotebook user on this server. To enable multiple scenario, you might have to setup a Ubuntu Server and [JupyterNotebook Hub](https://jupyter.org/hub) which are out of scope of this project.


```
$ sudo apt update
```

* Turn on SSL for PuTTy remote login

```
$ sudo apt install openssh-server 
```

* Install xrdp for remote desktop from windows

```
$ sudo apt install xrdp 
$ sudo systemctl status xrdp
$ sudo adduser xrdp ssl-cert
$ sudo reboot
```

to have the option to indicate if a new session should be created use following settings.
```
$ sudo nano /etc/xrdp/xrdp.ini
 
[Xorg]
name=Xorg
lib=libxup.so
username=ask
password=ask
ip=127.0.0.1
port=ask-1
code=20

$ sudo service xrdp restart
```


* Update python-dev and python virtual environment

```
$ sudo apt install python3-venv, python3-dev, python3-pip
$ sudo pip3 install -U virtualenv  # system-wide install
```

* Static IP address

A static local ip address is more convinent for home server. 

### CUDA

CUDA 10.1 is can be installed using following command. 

```
$ sudo apt install nvidia-cuda-toolkit
```

* Specify nVidia driver

In Software & Update --> Additional Drivers --> NVIDIA Corporation
Pick Using NVIDA driver metapackage from nvidia-driver-450 (proprietary)
Apply Changes


* Add following to ~/.bashrc

```
export LD_LIBRARY_PATH=/usr/lib/cuda/lib64:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=/usr/lib/cuda/include:$LD_LIBRARY_PATH
export PATH=$PATH:/home/[user]/.local/bin
```

* Check CUDA version after install it should shows 10.1 

```
$ nvcc -V

nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2019 NVIDIA Corporation
Built on Sun_Jul_28_19:07:16_PDT_2019
Cuda compilation tools, release 10.1, V10.1.243
```

```
$ nvidia-smi

+-----------------------------------------------------------------------------+
| NVIDIA-SMI 450.80.02    Driver Version: 450.80.02    CUDA Version: 11.0     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Tesla T4            On   | 00000000:00:1E.0 Off |                    0 |
| N/A   42C    P0    28W /  70W |      0MiB / 15079MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+

```
[CUDA Compatibility](https://docs.nvidia.com/deploy/cuda-compatibility/index.html)

### cuDNN

Download [cuDNN **v7.6.5** (November 5th, 2019), for CUDA 10.1](https://developer.nvidia.com/rdp/cudnn-archive)

Select cuDNN Library for Linux

```
$ tar -xzvf cudnn-10.1-linux-x64-v7.6.5.32.tgz

$ sudo cp cuda/include/cudnn*.h /usr/local/cuda/include
$ sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64

$ sudo chmod a+r /usr/local/cuda/include/cudnn*.h /usr/local/cuda/lib64/libcudnn*
```

[Installation Guid](https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html)

### Python setup

* Setup virtual environemnt for ML service and Jupyter Notebook


### install sklearn

```
sudo apt-get install python3-sklearn python3-sklearn-lib
```

### TensorFlow

TensorFlow no longer separates cpu and gpu package post v1.15.

```
pip install tensorflow
```

[TensorFlow official](https://www.tensorflow.org/install/gpu#linux_setup) has more details if needed.


### JupyterNotebook Setup

First install jupyter Notebook

```
pip install jupyternotebook
```

Setup Jupter Notebook by instruction provided by [Setup Jupter Notebook server as a daemon service](https://towshif.github.io/site/tutorials/Python/setup-Jupyter/) and [Running a notebook server](https://jupyter-notebook.readthedocs.io/en/stable/public_server.html#) 

Change ip address binding to allow remote access (Jupyter turn the remote access off by default) 

```
$ nano .jupyter/jupyter_notebook_config.py
```

```
## Allow requests where the Host header doesn't point to a local server
#
#  By default, requests get a 403 forbidden response if the 'Host' header shows
#  that the browser thinks it's on a non-local domain. Setting this option to
#  True disables this check.
#
#  This protects against 'DNS rebinding' attacks, where a remote web server
#  serves you a page and then changes its DNS to send later requests to a local
#  IP, bypassing same-origin checks.
#
#  Local IP addresses (such as 127.0.0.1 and ::1) are allowed as local, along
#  with hostnames configured in local_hostnames.
#  Default: False
c.NotebookApp.allow_remote_access = True

...

## The IP address the notebook server will listen on.
#  Default: 'localhost'
c.NotebookApp.ip = '*'
```

Jupyter Notebook server should accessible via http://[static ip address]:8888/tree


#### Validate python environment on Jupyter Notebook

```
!python --version
Python 3.8.5

!which python
/home/[user]/venv/bin/python


!pip --version

# Testing NVIDIA driver
!nvcc -V
!nvidia-smi

# Testing pip installation

!pip install pandas
```

#### Validate TenserFlow on Jupyter Notebook

```
import tensorflow as tf
tf.config.list_physical_devices("GPU")

output:

[PhysicalDevice(name='/physical_device:GPU:0', device_type='GPU')]
```


#### Add additional drive to JupyterNotebook

Under JupyterNotebook workspace folder create symbolic links for the additional folders from different mounted drives

```
ln -s "/mnt/slow/SlowData" "sdata"
ln -s "/mnt/fast/FastData" "fdata"
```





## Reference links (Thanks all peoples sharing their experience for the setup)

- [Slav Ivanov: The $1700 great Deep Learning box: Assembly, setup and benchmarks](https://blog.slavv.com/the-1700-great-deep-learning-box-assembly-setup-and-benchmarks-148c5ebe6415)
- [towshit: Setup Jupter Notebook server as a daemon service](https://towshif.github.io/site/tutorials/Python/setup-Jupyter/)
- [Stephen Gregory: Installing CUDA 10.1 on Ubuntu 20.04(https://medium.com/@stephengregory_69986/installing-cuda-10-1-on-ubuntu-20-04-e562a5e724a0)
- [whophil: jupyter.service](https://gist.github.com/whophil/5a2eab328d2f8c16bb31c9ceaf23164f)




### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/kenzhan/MLServer/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
