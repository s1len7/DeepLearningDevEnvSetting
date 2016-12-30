# Setting Caffe&Tensorflow Dev. Environment on Ubuntu 14.04.05
---

#### the purpose of this guide is to set dev. environments of Tensorflow ans Caffe for deep Learning with GPU.

---
	Machine Intellegence Lab. Korea Univ.

	Original creator : Jiseok Yun

	edit : Junsik Choi (10/28/16) 

---

# 0. System configuration


### 0.1 how to check hardware info?

`$ sudo lshw`

	- CPU : Intel i7-6700k
	- GPU : Gigabyte GTX 970 Windforce
	- Moderboard : H110M-A/M.2 (ASUSTeK COMPUTER INC.)
	- Firmware : BIOS ver 1801 (American Megatrends Inc.)
	- RAM : 16GiB (Samsung DDR4 DIMM 8GB PC4-17000 x 2)
	- OS : Ubuntu 14.04.05


### 0.2 Check BIOS Setting and disable 'SECURE BOOT' option

With secure boot option, You can not install some nvidia graphic drivers.
 
---

# 1.File preparation

### 1.1 Download following files and change its name and save it to /home/username/temp

A.	Cuda 7.5 --> cuda.run ( if your GPU is nvidia GTX 10xx, Titan X or higer, use cuda 8.0 or higher version)
([https://developer.nvidia.com/cuda-toolkit-archive](https://developer.nvidia.com/cuda-toolkit-archive "Download site"))
	
B.	Extracted Caffe --> caffe (directory)

`$ sudo apt-get install git`

`$ cd /home/usrname/temp`

`$ git clone https://github.com/BVLC/caffe.git`
	
D.	Extracted cuDNN v5.0 for CUDA 7.5 --> cudnn (directory)
([https://developer.nvidia.com/cudnn](https://developer.nvidia.com/cudnn))
	
E.	(OPTIONAL) Extracted OpenBLAS --> openblas (directory)
([https://www.openblas.net](https://www.openblas.net))
	
F.	(OPTIONAL) Anaconda 4.2.0 for python2.7 64bit installer --> anaconda.sh
([https://www.contunuum.io/downloads#linux](https://www.continuum.io/downloads#linux))

G. Nvidia Graphic Driver --> driver.run
[www.nvidia.co.kr/Drivers](www.nvidia.co.kr/Drivers) -> check your GPU driver name and Download driver
# 2.Update and Upgrade and install prerequisite

`$sudo apt-get update && sudo apt-get upgrade -y && sudo apt-get install build-essential -y && sudo apt-get install gfortran vim -y`

`$sudo apt-get install dkms fakeroot linux-headers-generic`
# 3.Disable nouveau and reboot into text mode

### 3.1 disable graphic mode

`$sudo vim /etc/modprobe.d/blacklist-nouveau.conf && sudo update-initramfs -u`

	blacklist nouveau
	blacklist lbm-nouveau
	options nouveau modeset=0
	alias nouveau off
	alias lbm-nouveau off

### 3.2 Enter text mode

`$sudo vim /etc/default/grub && sudo update-grub && sudo reboot`

	GRUB_CMDLINE_LINUX= “text”

# 4.Install NVIDIA CUDA toolkit

### 4.1 run CUDA installation file

`$cd ~/temp`

`$sudo sh cuda.run`

### 4.2 CUDA INSTALLATION 

#### **IMPORTANT : Do not install OpenGL & NVIDIA Driver of CUDA
	Do you accept the previously read EULA? (accept/decline/quit): accept
	Install NVIDIA Accelerated Graphics Driver for Linux-x86_64 346.46? ((y)es/(n)o/(q)uit): N
	Do you want to install the OpenGL libraries? ((y)es/(n)o/(q)uit) [ default is yes ]: n 
	Install the CUDA 7.5 Toolkit? ((y)es/(n)o/(q)uit): y
	Enter Toolkit Location [ default is /usr/local/cuda-7.5 ]: [Enter]
	Do you want to install a symbolic link at /usr/local/cuda? ((y)es/(n)o/(q)uit): y
	Install the CUDA 7.5 Samples? ((y)es/(n)o/(q)uit): y
	Enter CUDA Samples Location [ default is /root ]: [Enter]

### 4.3 Kill Nouveau and install nvidia driver

`$sudo vim /etc/default/grub`

	GRUB_CMDLINE_LINUX="rdblacklist=nouveau nouveau.modeset=0"

`$sudo update-grub`

`$cd ~/temp`

`$sudo sh driver.run`

#### do not make new kernal while installing driver. ignore warning for 32bits

### 4.4 reboot with GPU.

#### if you use VGA connector with your monitor, change to DVI. This will connect your monitor to GPU directly.

`$sudo reboot`

### 4.5 Check installations

`$nvidia-smi`
--> this should show you information of driver you just installed.

`$nvcc -V`
--> this should show you Cuda compiler driver informations.

#### if there is error with nvcc -V command,

`sudo gedit ~/.profile`

	export PATH=/usr/local/cuda-7.5/bin:$PATH
	export LD_LIBRARY_PATH=/usr/local/cuda-7.5/lib64:$LD_LIBRARY_PATH

`source ~/.profile`


## 5. Install BLAS

(ATLAS BLAS; default BLAS for caffe)

`$sudo apt-get install libatlas-base-dev`

(OpenBLAS) (optional)

`$cd ~/temp/openblas && make FC=gfortran -j8 && sudo make PREFIX=~/opt/OpenBLAS install -j8`

## 6. Install cuDNN

`$cd ~/temp/cudnn && sudo cp lib64/* /usr/local/cuda/lib64/ && sudo cp include/* /usr/local/cuda/include/`

## 7. Install anaconda (OPTIONAL; for me, PATH CONFIG. of Anaconda Is TOO DIFFICULT..)

`$cd ~/temp && bash anaconda.sh -b && sudo vim ~/.bashrc && source ~/.bashrc`

#### PATH setting

	export PATH=/usr/local/cuda/bin:$PATH
	export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
	export LD_LIBRARY_PATH=/opt/OpenBLAS/lib/:$LD_LIBRARY_PATH
	export PATH=${HOME}/anaconda2/bin:$PATH
	export PYTHONPATH=${HOME}/caffe/python:$PYTHONPATH

## 8. Install Caffe 

#### 8.1 Install prerequisite packages 
	$sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libhdf5-serial-dev protobuf-compiler libgflags-dev libgoogle-glog-dev liblmdb-dev -y && sudo apt-get install --no-install-recommends libboost-all-dev -y

#### 8.2 Install python requirements 

(If you using ANACONDA)

	$sudo mv ~/temp/caffe ~/caffe && cd ~/caffe && cat python/requirements.txt | xargs -L 1 conda install && sudo cp Makefile.config.example Makefile.config && sudo vim Makefile.config

(PYTHON)
        
        $sudo apt-get install python-pip

        $sudo mv ~/temp/caffe ~/caffe && cd ~/caffe && cat python/requirements.txt | xargs -L 1 sudo pip install && sudo cp Makefile.config.example Makefile.config && sudo vim Makefile.config

#### 8.3 Edit makefile(Makefile.config) (refer to attached Makefile.txt file)


## 9. Install Caffe and run test

#### 9.1 compile and test

`make all -j8`

`make test -j8`

`make runtest -j8`

#### error 127 while runtest
1) `$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda/lib64`

2)try:
32-bit: sudo ldconfig /usr/local/cuda/lib
64-bit: sudo ldconfig /usr/local/cuda/lib64

3) see https://groups.google.com/forum/#!topic/caffe-users/dcZrE3-60mc

4)  fatal error: hdf5.h: No such file or directory compilation terminated.
-> see https://github.com/BVLC/caffe/issues/2690
## 10. Install pycaffe

`make pycaffe -j8`

#### python/caffe/_caffe.cpp:10:31: fatal error: numpy/arrayobject.h: No such file or directory
==> EDIT Makefile.config
PYTHON_INCLUDE := /usr/include/python2.7 \
        /usr/local/lib/python2.7/dist-packages/numpy/core/include

## 11. Install tensorflow

	export TF_BINARY_URL=https://storage.googleapis.com/tensorflow/linux/gpu/tensorflow-0.10.0-cp27-none-linux_x86_64.whl && sudo pip install --ignore-installed --upgrade $TF_BINARY_URL

##(Optional)
## 12. Theano 0.8.2

`conda install -c trung theano=0.8.2`

## 13. FSL 5.0.9
	wget -O- http://neuro.debian.net/lists/trusty.jp.full | sudo tee /etc/apt/sources.list.d/neurodebian.sources.list

`sudo apt-key adv --recv-keys --keyserver hkp://pgp.mit.edu:80 0xA5D32F012649A5A9`

`sudo apt-get update`

`sudo apt-get install fsl-complete`


# Issues and trobleshooting

## 1. Python3 issues

	A. Only beta version (3.0.0-b2) of protobuf will support python3.
	B. Unsuported (by Ubuntu) version of Boosted must be used.

## 2. CUDA 7.5 Issues
	A. Caffe support upto Matlab 2015a, and 2015a does not support CUDA 7.5 --> MatConvNet

## 3. ***** not found

	do (ldconfig) --> https://github.com/BVLC/caffe/issues/1463#issuecomment-109074773
