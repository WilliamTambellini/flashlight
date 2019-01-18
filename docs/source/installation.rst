Building and Installing
=======================
Currently, flashlight must be built and installed from source.

Building/installing flashlight creates ``libflashlight``, which contains the entire flashlight library. Headers are contained in ``flashlight/``, which is placed in the specified ``include`` directory after install.

First, clone flashlight from `its repository on Github <https://github.com/facebookresearch/flashlight>`_:

::

   git clone https://github.com/facebookresearch/flashlight.git


Build Requirements
~~~~~~~~~~~~~~~~~~

- A C++ compiler with good C++ 11 support (e.g. g++ >= 4.8)
- `cmake <https://cmake.org/>`_ -- version 3.5.1 or later, and ``make``

Dependencies
------------

flashlight can be built with either a CUDA, CPU (in development), or OpenCL (coming soon) backend. Requirements vary depending on which backend is selected.

- For all backends, `ArrayFire <https://github.com/arrayfire/arrayfire/wiki>`_ >= 3.6.1 is required. flashlight has been tested with `ArrayFire 3.6.1 <https://github.com/arrayfire/arrayfire/releases/tag/v3.6.1>`_.
- The following dependencies are `downloaded, built, and installed automatically` with flashlight:

  - `Cereal <https://github.com/USCiLab/cereal>`_ is required for serialization -- the `develop` branch must be used.
  - If building tests, `Google Test <https://github.com/google/googletest>`_ >= 1.8.0 is required.


Distributed Training Dependencies
---------------------------------
Building with distributed training is optional. See ``Build Options`` below.

- Regardless of backend, running with distributed training requires an MPI installation. `OpenMPI <https://www.open-mpi.org/>`_ is recommended if supported on your OS.
- If building the CUDA backend, `NVIDIA's NCCL library <https://developer.nvidia.com/nccl>`_ is required. Flashlight has been tested with NCCL 2.2.13.
- If building with the CPU backend, `Facebook's Gloo library <https://github.com/facebookincubator/gloo>`_ is required, and must be built with MPI.

CUDA Backend Dependencies
-------------------------

- CUDA >= 9.2 is required. flashlight has been tested with `CUDA 9.2 <https://developer.nvidia.com/cuda-92-download-archive>`_.
- CUDNN >= 7.1.2 is required. flashlight has been tested with `CUDNN 7.1.2 <https://developer.nvidia.com/rdp/cudnn-archive>`_.

CPU Backend Dependencies
------------------------

The CPU backend is currently under active development. Required dependencies include:

- Intel's `Math Kernel Library <https://software.intel.com/en-us/mkl>`_ >= 2017. flashlight has been tested with MKL 2017.
- Intel's `MKL-DNN <https://github.com/intel/mkl-dnn/>`_ framework, which is best linked directly to MKL for better performance.

In order for the flashlight build to properly find MKL, in most cases, the environment variable ``MKLROOT`` must be set to a directory where the MKL installation is located. This should be done in the same shell session where ``cmake`` is to be run when building flashlight. On most Linux-based systems, the correct command will be:

.. code-block:: shell

   export MKLROOT=/opt/intel/mkl

OpenCL Backend Dependencies
---------------------------

The OpenCL backend is not currently supported.

Build Instructions
~~~~~~~~~~~~~~~~~~
Build Options
-------------
+-------------------------+-------------------+---------------+
| Options                 | Configurations    | Default Value |
+=========================+===================+===============+
| FLASHLIGHT_BACKEND      | CUDA, CPU, OPENCL | CUDA          |
+-------------------------+-------------------+---------------+
| FL_BUILD_DISTRIBUTED    | ON, OFF           | ON            |
+-------------------------+-------------------+---------------+
| FL_BUILD_TESTS          | ON, OFF           | ON            |
+-------------------------+-------------------+---------------+
| FL_BUILD_EXAMPLES       | ON, OFF           | ON            |
+-------------------------+-------------------+---------------+
| CMAKE_BUILD_TYPE        | CMake build types | Debug         |
+-------------------------+-------------------+---------------+
| ArrayFire_DIR           | Path to ArrayFire |               |
+-------------------------+-------------------+---------------+


Building on Linux or MacOS
--------------------------
flashlight has been thoroughly tested on Ubuntu 16.04 and above, CentOS 7.5, and macOS 10.14, but has good compatability with older operating systems.

Building on Linux and MacOS is simple:

.. code-block:: shell

  # in the flashlight project directory:
  mkdir -p build
  cd build
  # Note: If you have done a local custom build/install of AF, you can specify the install path with ArrayFire_DIR : 
  # example: if you built and installed AF in /tmp/AF add -DArrayFire_DIR=/tmp/AF/share/ArrayFire/cmake/
  cmake .. -DCMAKE_BUILD_TYPE=Release -DFLASHLIGHT_BACKEND=[backend] # valid backend
  make -j4  # (or any number of threads)
  make test

To change the location of the install, simply `set CMake's <https://cmake.org/cmake/help/v3.5/variable/CMAKE_INSTALL_PREFIX.html>`_ ``CMAKE_INSTALL_PREFIX`` before running ``cmake``, then:

.. code-block:: shell

 make install

To build a shared object, simply `set CMake's <https://cmake.org/cmake/help/v3.5/variable/BUILD_SHARED_LIBS.html>`_ ``BUILD_SHARED_LIBS`` when running ``cmake``.

Building on Windows
-------------------
Building flashlight on Windows is not supported at this time (coming soon).

Building/Running flashlight with Docker
---------------------------------------
flashlight and its dependencies can also be built with the provided Dockerfile.

To build flashlight with Docker:

- Install `Docker <https://docs.docker.com/engine/installation/>`_
- For CUDA backend install `nvidia-docker <https://github.com/NVIDIA/nvidia-docker/>`_
- Run the given Dockerfile in a new container:

.. code-block:: shell
 # for CUDA backend
 sudo docker run --runtime=nvidia --rm -itd --ipc=host --name flashlight flml/flashlight:cuda-latest
 # for CPU backend
 sudo docker run --rm -itd --name flashlight flml/flashlight:cpu-latest
 # go to terminal in the container
 sudo docker exec -it flashlight bash

- to run tests inside a container

.. code-block:: shell

 cd /root/flashlight/build && make test

- Build Docker image from source:

.. code-block:: shell

 git clone --recursive https://github.com/facebookresearch/flashlight.git
 cd flashlight
 # for CUDA backend
 sudo docker build -f ./Dockerfile-CUDA -t flashlight .
 # for CPU backend
 sudo docker build -f ./Dockerfile-CPU -t flashlight .

Building Your Project with flashlight
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Once flashlight is built and installed, including it in another project is simple, using CMake. Suppose we have a project in ``project.cpp`` that uses flashlight:

::

   #include <iostream>

   #include <arrayfire.h>
   #include "flashlight/flashlight.h"

   /**
    * ###### #         ##    ####  #    # #      #  ####  #    # #####
    * #      #        #  #  #      #    # #      # #    # #    #   #
    * #####  #       #    #  ####  ###### #      # #      ######   #
    * #      #       ######      # #    # #      # #  ### #    #   #
    * #      #       #    # #    # #    # #      # #    # #    #   #
    * #      ####### #    #  ####  #    # ###### #  ####  #    #   #
    */
   int main() {
     fl::Variable v(af::constant(1, 1), true);
     auto result = v + 10;
     std::cout << "Hello World!" << std::endl;
     af::print("Array value is ", result.array()); // 11.000
     return 0;
   }

We can link flashlight with the following CMake configuration:

.. code-block:: shell

  # CMake 3.5.1+ is recommended
  cmake_minimum_required(VERSION 3.5.1)
  # C++ 11 is required
  set(CMAKE_CXX_STANDARD 11)
  set(CMAKE_CXX_STANDARD_REQUIRED ON)

  add_executable(myProject project.cpp)

  find_package(ArrayFire REQUIRED)
  # ...

  find_package(flashlight REQUIRED)
  # ...

  target_link_libraries(
    myProject
    PRIVATE
     # the correct ArrayFire backend is transitively included by flashlight
    flashlight::flashlight
  )

The above will automatically link all flashlight backend-specific dependencies and will add the correct directories to the target's (``myProject``'s) include directories.
