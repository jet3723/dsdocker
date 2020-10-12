# dsdocker
A Docker container for data science exploration. 

## Overview
This project provides a Docker image containing Anaconda and Python3. The image allows for the exploration of data science topics like Machine Learning, Computer Vision Learning, and AI within an isolated environment, thereby not polluting the host computer. 

In this case, the host computer is a MacBook Pro. There isn't anything special about the Docker image, but it does have the expectation of being able to display to an X11 server on the host computer. There are instructions for setting up an X11 server on a MacBook. If you are using another computer (Linux, Windows), simply setup an X11 server and the Docker image will connect to that server.

An overview of the project can be found at [Coderprocess](https://blog.coderprocess.com).

This document describes how to build the Docker image, start the Docker container, initialize the container, test the container, and restart the container. The hard work is performed in building and initializing the container. After that, startup/restart of the container is relatively easy.

**Pre-requisites**

1. Docker is installed on the host computer.

**Install Mac X11 Server**

Follow these instructions to install and run an X11 server on a MacBook.

    1. Install and run socat
       brew install socat
       socat TCP-LISTEN:6000,reuseaddr,fork UNIX-CLIENT:\"$DISPLAY\"

    2. Install XQuartz X11 Server
       https://www.xquartz.org
       Under Preferences->Security check the option "Allow connections from network clients"

    3. After the initial installation of socat and X11, make sure to start socat first, then start the X11 Server.
       

### Build The Docker Image
Because the Dockerfile has already been created, building the image is relatively easy. Execute the following command:

    docker build -t dsdocker .


### Start the Container
Start the container using the following command line. The DISPLAY IP address is the address of the MacBook en0 interface.

    1. Get the IP Address of the MacBook
       ifconfig en0   use the inet address

    2. docker run -i -t -p 8888:88888 -e DISPLAY=<ipaddr>:0 -v /tmp/.X11-unix:/tmp/.X11-unix dsdocker /bin/bash

    Example:
    ifconfig en0
    en0: flags=8863<UP,BROADCAST,SMART,RUNNING,SIMPLEX,MULTICAST> mtu 1500
	 options=400<CHANNEL_IO>
	 ether 3c:22:fb:78:6a:df 
	 inet6 fe80::449:88f8:c9ff:7226%en0 prefixlen 64 secured scopeid 0x6 
	 inet 192.168.7.81 netmask 0xffffff00 broadcast 192.168.7.255
	 nd6 options=201<PERFORMNUD,DAD>
	 media: autoselect
	 status: active


    docker run -i -t -p 8888:8888 -e DISPLAY=192.168.7.81:0 -v /tmp/.X11-unix:/tmp/.X11-unix dsdocker /bin/bash

### Container Initialization
Container initialization is only necessary after the initial startup of the container. There are instructions further down for steps required when restarting an existing container.

    1. Initialize Conda Shell
       Execute the following commands
          - bash
          - conda init bash
          - vi /home/anaconda/.bashrc
            Add the line:
               export XDG_RUNTIME_DIR=/home/anaconda/.xdg
          - mkdir .xdg
          - exit
          - bash
    2. Install a Python3.6 environment
       Execute the following commands
         - conda create --name py36 python=3.6
         - conda activate py36
    3. Install Qt and PyQt
       Execute the following commands
         - conda install qt pyqt -y
    4. Install OpenCV
       Execute the following command
         - pip install opencv-python opencv-contrib-python
    5. Install Keras and Tensorflow
       Execute the following command
         - conda install keras -y
    6. Install Matplotlib
       Execute the following command
         - conda install matplotlib -y
    7. Install Jupyter
       Execute the following commands
         - conda install jupyter -y
         - mkdir /opt/notebooks
    8. Run Jupyter
       Execute the following command
         - jupyter notebook --notebook-dir=/opt/notebooks --ip='*' --port=8888 --no-browser
       The log output gives instructions for how to connect a web browser
       to Jupyter.

### Test The Container
To test the installation, upload an image file through the web browser. Then create a python program to display the image. There are two tools that can be used, either a Python3 Notebook, Terminal, Container Shell. Both will be described.

#### Python3 Notebook
Using the Browser open a new Python3 Notebook. Enter the following program and run it.

    import matplotlib.pyplot as plt
    import matplotlib.image as mpimg

    fn = mpimg.imread('FilePath')
    plt.imshow(fn)
    plt.show()
The image should be displayed inline within the Notebook.

#### Terminal
Using the Browser open a new Terminal. The terminal needs a little bit of initialization before running a python program.

    - Initialize the terminal by executing the following commands
      - bash
      - conda activate py36
    - Run a Python Shell
      - python
    - Enter the following program
        import matplotlib.pyplot as plt
        import matplotlib.image as mpimg

        fn = mpimg.imread('FilePath')
        plt.imshow(fn)
        plt.show()

The image should be displayed in an X11 window on the MacBook.

#### Container Shell
A container shell can also be used in development. If the container is already running, a new shell be be created using the command:

    docker exec -i -t dsdocker /bin/bash

After the shell is running, initialize it and run the program:

    - Initialize the terminal by executing the following commands
      - bash
      - conda activate py36
    - Run a Python Shell
      - python
    - Enter the following program
        import matplotlib.pyplot as plt
        import matplotlib.image as mpimg

        fn = mpimg.imread('FilePath')
        plt.imshow(fn)
        plt.show()

### Restart an Existing Container
To restart an existing container execute the following command:

    docker start dsdocker 
    docker exec -i -t dsdocker /bin/bash
    conda activate py36
    jupyter notebook --notebook-dir=/opt/notebooks --ip='*' --port=8888 --no-browser

The container is now up and running again.