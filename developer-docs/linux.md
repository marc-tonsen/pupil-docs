+++
date = "2017-01-19T16:40:20+07:00"
title = "linux"
section_weight = 4
page_weight = 1.1
+++

## Linux Dependencies

These installation instructions are tested using **Ubuntu 16.04 or higher** running on many machines. Do not run Pupil on a VM unless you know what you are doing.

### Install Linux Dependencies

Let's get started! Its time for `apt`!  Just copy paste into the terminal and listen to your machine purr.

```
sudo apt install -y pkg-config git cmake build-essential nasm wget python3-setuptools libusb-1.0-0-dev  python3-dev python3-pip python3-numpy python3-scipy libglew-dev libglfw3-dev libtbb-dev
```

> ffmpeg >= 3.2

```
sudo add-apt-repository ppa:jonathonf/ffmpeg-3
sudo apt-get update
sudo apt install libavformat-dev libavcodec-dev libavdevice-dev libavutil-dev libswscale-dev libavresample-dev ffmpeg libav-tools x264 x265 libportaudio2 portaudio19-dev
```

> OpenCV

We require a version of OpenCV that is >= 3.0. In **Ubuntu 18.04** it can simply be installed with `sudo apt install python3-opencv libopencv-dev`

In older versions of Ubuntu it needs to be compiled from source.

The requisites for opencv to build python3 cv2.so library are:

1. python3 interpreter found
1. libpython***.so shared lib found (make sure to install python3-dev)
1. numpy for python3 installed.

```bash
git clone https://github.com/opencv/opencv
cd opencv
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=RELEASE -DWITH_TBB=ON -DWITH_CUDA=OFF -DBUILD_opencv_python2=OFF -DBUILD_opencv_python3=ON ..
make -j2
sudo make install
sudo ldconfig
```

<aside class="notice">
OpenCV is not able to build Python 2 and Python 3 modules at the same time.
OpenCV will build the Python 2 module by default if requirements for both
versions are met. Setting the Python 2 Numpy include directory to an empty
string effectively disables the Python 2 module build.
</aside>

<aside class="faq">
Do *not* install `opencv-python` via pip if you see `ImportError: No module named 'cv2'`.
The error appears if the above requisites were not met.
Delete the build folder, recheck the requisites and try again.
</aside>

<aside class="faq">
The `ImportError: */detector_2d.*.so: undefined symbol: *ellipse*InputOutputArray*RotatedRect*Scalar*`
appears if opencv has been installed previously via `apt-get`.

1. Remove all opencv installations that were installed via `apt-get`.
1. Delete the `*.so` files as well as the `build` dirctory within the `pupil_detectors` directory.
1. Start Pupil Capture. This should trigger a recompilation of the detector modules.
</aside>

> Turbojpeg

```
wget -O libjpeg-turbo.tar.gz https://sourceforge.net/projects/libjpeg-turbo/files/1.5.1/libjpeg-turbo-1.5.1.tar.gz/download
tar xvzf libjpeg-turbo.tar.gz
cd libjpeg-turbo-1.5.1
./configure --enable-static=no --prefix=/usr/local
sudo make install
sudo ldconfig
```

> custom version of libusb
Required for 17.10 and with 200hz cameras only. Otherwise IGNORE!)

1. Build or download fixed binary from release: https://github.com/pupil-labs/libusb/releases/tag/v1.0.21-rc6-fixes
1. Replace system libusb-1.0.so.0 with this binary.

```
sudo cp '~/path to your fixed binary/libusb-1.0.so.0' '/lib/x86_64-linux-gnu/libusb-1.0.so.0'
```

> libuvc

```
git clone https://github.com/pupil-labs/libuvc
cd libuvc
mkdir build
cd build
cmake ..
make && sudo make install
```

> udev rules for running libuvc as normal user

```
echo 'SUBSYSTEM=="usb",  ENV{DEVTYPE}=="usb_device", GROUP="plugdev", MODE="0664"' | sudo tee /etc/udev/rules.d/10-libuvc.rules > /dev/null
sudo udevadm trigger
```

> Install packages with `pip`

```
sudo pip3 install numexpr
sudo pip3 install cython
sudo pip3 install psutil
sudo pip3 install pyzmq
sudo pip3 install msgpack
sudo pip3 install pyopengl
sudo pip3 install pyaudio
sudo pip3 install git+https://github.com/zeromq/pyre
sudo pip3 install git+https://github.com/pupil-labs/PyAV
sudo pip3 install git+https://github.com/pupil-labs/pyuvc
sudo pip3 install git+https://github.com/pupil-labs/pyndsi
sudo pip3 install git+https://github.com/pupil-labs/pyglui
```

> 3D eye model dependencies
First install 

```bash
sudo apt-get install libboost-dev
sudo apt-get install libboost-python-dev
sudo apt-get install libgoogle-glog-dev libatlas-base-dev libeigen3-dev
```

> Ceres

> **Ubuntu 18.04**

```bash
sudo apt install libceres-dev
```

Next we need to install the Ceres library. In **Ubuntu 18.04** Ceres is available as a package in the repositories.
In older versions it has to be compiled from source. Choose the correct command on the right depending on your version 
of Ubuntu!

> **Ubuntu <= 17.10**

```bash
# sudo apt-get install software-properties-common if add-apt-repository is not found
sudo add-apt-repository ppa:bzindovic/suitesparse-bugfix-1319687
sudo apt-get update
sudo apt-get install libsuitesparse-dev
# install ceres-solver
git clone https://ceres-solver.googlesource.com/ceres-solver
cd ceres-solver
mkdir build && cd build
cmake .. -DBUILD_SHARED_LIBS=ON
make -j3
make test
sudo make install
sudo sh -c 'echo "/usr/local/lib" > /etc/ld.so.conf.d/ceres.conf'
sudo ldconfig
```

> (Optional) Install PyTorch + CUDA and cuDNN. 

> **Version 1: No GPU acceleration**: Install PyTorch via pip

```bash
pip3 install http://download.pytorch.org/whl/cpu/torch-0.4.0-cp36-cp36m-linux_x86_64.whl 
pip3 install torchvision
```

Some bleeding edge features require the deep learning library PyTorch. 
For GPU acceleration CUDA and cuDNN are also required. Without GPU acceleration some of the features will probably not 
run in real-time.


> **Version 2: No GPU acceleration**: Install PyTorch via pip

```bash
pip3 install http://download.pytorch.org/whl/cu90/torch-0.4.0-cp36-cp36m-linux_x86_64.whl 
pip3 install torchvision
```

```bash
sudo dpkg -i cuda-repo-ubuntu1604-9-0-local_9.0.176-1_amd64.deb
sudo apt-key add /var/cuda-repo-9-0-local/7fa2af80.pub
sudo apt-get update
sudo apt-get install -y cuda libcupti-dev
``` 

Download CUDA 9.0 from https://developer.nvidia.com/cuda-90-download-archive?target_os=Linux
Choose the choose the **deb (local)** file for your architecture. We have tested the setup with the 16.04 version.
Install it via

```bash
echo "
# Setting up CUDA
export CUDA_ROOT=/usr/local/cuda
export LD_LIBRARY_PATH=\${LD_LIBRARY_PATH}:\${CUDA_ROOT}/lib64:\${CUDA_ROOT}/extras/CUPTI/lib64
export PATH=\${CUDA_ROOT}/bin:\${PATH}
" | sudo tee /etc/profile.d/cuda_env.sh 
```

Add CUDA files to the appropriate path variables

```bash
tar -xvf cudnn-9.0-linux-x64-v7.tgz
cd cuda
sudo cp -P include/cudnn.h /usr/include
sudo cp -P lib64/libcudnn* /usr/lib/x86_64-linux-gnu/
sudo chmod a+r /usr/lib/x86_64-linux-gnu/libcudnn*
```

Download cuDNN 7.1 from https://developer.nvidia.com/cudnn
The download requires you to register as a developer at Nvidia. This registration is free. Download the version for 
CUDA 9.0, specifically download the **Library for Linux**.