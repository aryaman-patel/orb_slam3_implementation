# Setting up ORB SLAM3

# Installation
Following are the instruction to install ORB-SLAM3, I have referenced [this repo](https://github.com/egdw/ORB_SLAM3_Ubuntu20.04.git) for the installation in 
Ubuntu 20.04 along with further changes added by me. 

## Modification

### Error1:

```
In file included from /usr/local/include/pangolin/utils/signal_slot.h:3,
                 from /usr/local/include/pangolin/windowing/window.h:35,
                 from /usr/local/include/pangolin/display/display.h:34,
                 from /usr/local/include/pangolin/pangolin.h:38,
                 from /home/a616708946/slambook/ch5/code/disparity.cpp:8:
/usr/local/include/sigslot/signal.hpp:109:79: error: ‘decay_t’ is not a member of ‘std’; did you mean ‘decay’?
  109 | constexpr bool is_weak_ptr_compatible_v = detail::is_weak_ptr_compatible<std::decay_t<P>>::value;
      |                                                                               ^~~~~~~
      |                                                                               decay
/usr/local/include/sigslot/signal.hpp:109:79: error: ‘decay_t’ is not a member of ‘std’; did you mean ‘decay’?
  109 | constexpr bool is_weak_ptr_compatible_v = detail::is_weak_ptr_compatible<std::decay_t<P>>::value;
      |                                                                               ^~~~~~~
      |                                                                               decay
/usr/local/include/sigslot/signal.hpp:109:87: error: template argument 1 is invalid
  109 | constexpr bool is_weak_ptr_compatible_v = detail::is_weak_ptr_compatible<std::decay_t<P>>::value;
```

update Cmakelists.txt from -std=c++11 to -std=c++14

```
CHECK_CXX_COMPILER_FLAG("-std=c++14" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX11)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
   add_definitions(-DCOMPILEDWITHC11)
   message(STATUS "Using flag -std=c++14.")
elseif(COMPILER_SUPPORTS_CXX0X)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
   add_definitions(-DCOMPILEDWITHC0X)
   message(STATUS "Using flag -std=c++0x.")
else()
   message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()
```

## error2

This is an error due to the Eigen version.

```
   ../core/base_edge.h: 33:10: fatal error: Eigen/Core: No such file or directory 
#include <Eigen/Core>

```

Replace all of ``#include <Eigen/(any packages)>`` to ``#include <eigen3/Eigen/(any packages)>``.

For example:

```
#include <Eigen/Core>
to
#include <eigen3/Eigen/Core>

```
These changes have to made in all the files using the Eigen dependency in the  `` ORB_SLAM3/include/`` folder.


# 1. Installation of ORB-SLAM 3 on a fresh installed Ubuntu 20.04
Install all liberay dependencies.

```

sudo add-apt-repository "deb http://security.ubuntu.com/ubuntu xenial-security main"
sudo apt update

sudo apt-get install build-essential
sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev

sudo apt-get install python-dev python-numpy libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libdc1394-22-dev libjasper-dev

sudo apt-get install libglew-dev libboost-all-dev libssl-dev

sudo apt install libeigen3-dev
```

# Install OpenCV 3.2.0
The ORB-SLAM 3 was test by
```
cd ~
mkdir Dev && cd Dev
git clone https://github.com/opencv/opencv.git
cd opencv
git checkout 3.2.0
```

Put the following at the top of header file 

```
gedit ./modules/videoio/src/cap_ffmpeg_impl.hpp
#define AV_CODEC_FLAG_GLOBAL_HEADER (1 << 22)
#define CODEC_FLAG_GLOBAL_HEADER AV_CODEC_FLAG_GLOBAL_HEADER
#define AVFMT_RAWPICTURE 0x0020
```

and save and close the file

```
mkdir build
cd build
cmake -D CMAKE_BUILD_TYPE=Release -D WITH_CUDA=OFF -D CMAKE_INSTALL_PREFIX=/usr/local ..
make -j 3
sudo make install

```

# Install Pangolin
Now, we install the Pangolin. I used the commit version 86eb4975fc4fc8b5d92148c2e370045ae9bf9f5d


```
cd ~/Dev
git clone https://github.com/stevenlovegrove/Pangolin.git
cd Pangolin 
mkdir build 
cd build 
cmake .. -D CMAKE_BUILD_TYPE=Release 
make -j 3 
sudo make install

```

# ORB-SLAM 3
Now, we install ORB-SLAM3. I used the commit version ef9784101fbd28506b52f233315541ef8ba7af57 tag: v0.3-beta

```
cd ~/Dev
git clone https://github.com/UZ-SLAMLab/ORB_SLAM3.git 
cd ORB_SLAM3

```

We need to change the header file gedit ``./include/LoopClosing.h`` at line 51
from
```
Eigen::aligned_allocator<std::pair<const KeyFrame*, g2o::Sim3> > > KeyFrameAndPose;
```
to
```
Eigen::aligned_allocator<std::pair<KeyFrame *const, g2o::Sim3> > > KeyFrameAndPose; 
```
in order to make this comiple.
Now, we can comiple ORB-SLAM3 and it dependencies as DBoW2 and g2o.

Now Simply just run (if you encounter compiler, try to run the this shell script 2 or 3 more time. It works for me.)

```
./build.sh

```
# Processing the images- 

[Kevin Robb](https://github.com/kevin-robb/orb_slam_implementation.git)'s provies a handy code ``get_images.py`` to convert the bag files to get images in every frame of the dataset.
The images had the following file structure:
```
/Dataset 
    KITTI/ 
        00/
            image_0/     image_1/       times.txt       calib.txt
    Nuance/
        00/
            image_0/     image_1/       times.txt
```


# Camera Calibration parameters-
Included the camera calibration parameters in the 
```
/Example/Stereo/Nuance.yaml 
``` 
file. 

# Command to run the datasets:

Run the following in the ``~/Dev/ORB_SLAM3/`` file.

1. EuRoC
```
./Examples/Stereo/stereo_euroc ./Vocabulary/ORBvoc.txt ./Examples/Stereo/EuRoC.yaml ~/Datasets/EuRoc/MH01 ./Examples/Stereo/EuRoC_TimeStamps/MH01.txt
```

2. KITTI Dataset
```
./Examples/Stereo/stereo_kitti ./Vocabulary/ORBvoc.txt ./Examples/Stereo/KITTI00-02.yaml ~/Datasets/Kitti/00
```

3. Nuance 
```
./Examples/Stereo/stereo_kitti ./Vocabulary/ORBvoc.txt ./Examples/Stereo/Nuance.yaml ~/Datasets/Nuance/00
```

