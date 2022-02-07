# Setting Up a Docker Runtime for (Local) Nvidia GPU
This guide is for setting up the Nvidia Docker Container Runtime on Ubuntu 20.04. The official documentation for all Linux distros can be found [here](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html).

## Pre-Requisites

Ensure that the latest nvidia drivers are installed on your system. In a nutshell, the following steps must be considered:
1. GNU/Linux x86_64 with kernel version > 3.10
1. Docker >= 19.03 (recommended, but some distributions may include older versions of Docker. The minimum supported version is 1.12)
1. NVIDIA GPU with Architecture >= Kepler (or compute capability 3.0)
1. NVIDIA Linux drivers >= 418.81.07 (Note that older driver releases or branches are unsupported.)

All of the above steps can be found in more detail [here](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#platform-requirements).

By default, the Docker daemon runs as the `root` user and only users with `sudo` access can use the daemon. You can circumvent this by adding users to a special group called `docker`. If you'd like to avoid having to run all commands as `root`, the following steps can accomplish that:

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
```

Log out and back for the changes to take effect or since this is a Linux distro, just run the following command for the changes to take effect:

```bash
newgrp docker
```

Verify that you can run `docker` commands as root with something like the following (**NOTE** Docker must already be installed for this work; skip to the next step if not already installed):

```bash
docker run hello-world
```

For more post-installation info, see the following [Docker documentation](https://docs.docker.com/engine/install/linux-postinstall/).

## Installing Docker-CE on Ubuntu
If not already installed, install the Docker Community Edition with the following command:

```bash
curl https://get.docker.com | sh && sudo systemctl --now enable docker
```

## Setting Up NVIDIA Container Toolkit
1. Set up the `stable` repository and GPG key for the container toolkit

```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
   && curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - \
   && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
```

1. Install the nvidia-docker2 package (and dependencies) after updating the package listing:
```bash
sudo apt-get update
sudo apt-get install -y nvidia-docker2
```

1. Restart the Docker daemon to complete the install:
```bash
sudo systemctl restart docker
```

1. Now test the setup by running a base CUDA container locally:
```bash
sudo docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
```

An output like the following should be observed:
```bash
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 470.86       Driver Version: 470.86       CUDA Version: 11.4     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  Off  | 00000000:26:00.0  On |                  N/A |
| 41%   30C    P8    19W / 215W |    690MiB /  7981MiB |     11%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
+-----------------------------------------------------------------------------+
```

## Testing Python/Tensorflow 2 Container from NGC Catalog
The NGC catalog are official containers supported by Nvidia and developers for running GPU-accelerated workloads within a container environment. Beyond having the required drivers installed from above, these containers contain what is required at runtime for using popular Python libraries that can take advantage of GPU acceleration. For this example, let's make sure a simple container running TensorFlow 2 works with our current setup.

**NOTE:** Before you can pull images from the 

1. With Docker 19.03+, running the following command should give us a TF2 installation to run:

   ```bash
   docker run --gpus all -it --rm -v /path/to/local/dir:/container_dir nvcr.io/nvidia/tensorflow:21.12-tf2-py3 bash
   ```
   Once in the command line, you can run the following within the `/workspace/cnn/` dir to test your GPU for training:

   ```bash
   python resnet.py --num_iter=90 --iter_unit=epoch \
   --precision=fp16 --display_every=100 \
   --export_dir=/tmp --batch_size=64
   ```