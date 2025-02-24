## How to create / update / run the docker image for non jetson devices

In order to build or run the docker image, you need to have:

- An ubuntu computer/server with a Nvidia CUDA GPU : https://developer.nvidia.com/cuda-gpus

Depending on your target GPU, you will need to change the CUDA_ARCH_BIN variable in the Dockerfile

TODO @tdurand improve this documentation

### 1. Run the docker image

// Uninstall nvidia drivers : sudo apt-get purge nvidia*

// TODO, docker image is compile for a cuda version (10.1) , need to install driver accordingly

Install nvidia driver, 

- maybe via cuda toolkit: https://developer.nvidia.com/cuda-downloads, you do not need cuda but it is a reliable way to install the nvidia driver with it

- TODO find way to just install latest nvidia driver without cuda stuff

- reboot sudo reboot

- Test install: `nvidia-smi`

Install docker: https://docs.docker.com/install/linux/docker-ce/ubuntu/

Install nvidia-docker : https://github.com/NVIDIA/nvidia-docker#ubuntu-140416041804-debian-jessiestretch

# Test nvidia-smi with the latest official CUDA image
sudo docker run --runtime=nvidia --rm nvidia/cuda nvidia-smi

__Our image is built for CUDA_ARCH_BIN=6.1__

```bash
# After installing docker-nvidia
sudo docker run --runtime=nvidia -p 8080:8080 -p 8090:8090 -p 8070:8070 -v /data/db:/data/db -d --restart unless-stopped opendatacam/opendatacam:v2.0.0-beta.3-dockernvidia
# Open browser at http://localhost:8080

# Run with custom config
sudo docker run --runtime=nvidia -v $(pwd)/config.json:/var/local/opendatacam/config.json -p 8080:8080 -p 8090:8090 -p 8070:8070 -v /data/db:/data/db --rm -it opendatacam/opendatacam:v2.0.0-beta.3-dockernvidia
```

### 2. Build the image

```bash
# Go to an empty folder
mkdir docker
cd docker
# get the docker file : https://github.com/opendatacam/opendatacam/blob/master/docker/run-cloud/Dockerfile
wget https://raw.githubusercontent.com/opendatacam/opendatacam/v2/docker/run-cloud/Dockerfile
# Build
# Takes a really long time the first time as it compile opencv
sudo docker build -t opendatacam .

# Test the image in interactive mode
sudo docker run --runtime=nvidia -p 8080:8080 -p 8090:8090 -p 8070:8070 -v /data/db:/data/db --rm -it opendatacam
```

### 3. Publish the docker image

```bash
# Log into the Docker Hub
sudo docker login --username=opendatacam
# Check the image ID using
sudo docker images
# You will see something like:
# REPOSITORY              TAG       IMAGE ID         CREATED           SIZE
# opendatacam             latest    023ab91c6291     3 minutes ago     1.975 GB

# Tag your image
sudo docker tag 7ef920844953 opendatacam/opendatacam:v2.0.0-beta.3-dockernvidia

# Untag image (if you made a tipo)
sudo docker rmi opendatacam/opendatacam:v2.0.0-beta.3-dockernvidia

# Push image
sudo docker push opendatacam/opendatacam:v2.0.0-beta.3-dockernvidia
```

### Improvements to make : docker image smaller 

For now the docker image is very large (12GB)

Need to try use the lightweight runtime of nvidia/cuda to have a build part and a run part

Would need to copy the opencv compiled file (maybe the deb as with the jetson), and the darknet compiled folder directly

Links:

https://github.com/TakuroFukamizu/nvidia-docker-darknet/blob/master/Dockerfile
https://medium.com/techlogs/compiling-opencv-for-cuda-for-yolo-and-other-cnn-libraries-9ce427c00ff8