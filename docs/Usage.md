
# Singularity Usage

## 1. Typical Installation

Singularity images can be built by using the Singularity CLI. To install the required tools please check [Quick Start](https://sylabs.io/guides/3.0/user-guide/quick_start.html).

These steps requires a Linux system with root privileges. If it's not suitable for you, please refer to the next section.

## 2. Singularity through Docker

To obtain Singularity images without having to install Singularity and it's dependencies, Docker can be used with following two options:

### 2.1 Using Dockerfile

If you have a Dockerfile, it can be converted to a singularity definition file which will be used to build the singularity image.

```.bash
pip3 install spython
spython recipe Dockerfile &> singularity.def
```
To build the image from the definition file, use [Remote Builder](https://cloud.sylabs.io/builder). (It requires signing in)

### 2.2 Using an existing Docker Image

If you have a Docker image (in local filesystem, Docker Hub, or your private registry), it can be converted to a Singularity image.

Using a registry:

```.bash
# $OUTPUT_PATH is the path the resulting singularity image will be put.

docker run -v /var/run/docker.sock:/var/run/docker.sock \
		   -v $OUTPUT_PATH:/output \
		   --privileged -t --rm \
		   quay.io/singularity/docker2singularity \
		   pytorch/pytorch:latest
```

or using a locally built image from a Dockerfile:

```.bash
# Build the Dockerfile in the current directory and tag it as my_container
docker build -t my_container .
# $OUTPUT_PATH is the path the resulting image will be put.
docker run -v /var/run/docker.sock:/var/run/docker.sock \
		   -v $OUTPUT_PATH:/output \
		   --privileged -t --rm \
		   quay.io/singularity/docker2singularity \
		   docker.io/library/my_container
```

## 3. Usage

After building the Singularity image, to run your code in the container following commands can be used. Note that Singularity is currently installed on the "dl4" node and the commands above should be sent to this node over a Slurm job.

 - If you have built your image locally and obtain the .sif file:

```.bash
# Runs `run.py`in the container
singularity exec my_image.sif python3 run.py
```

 - If you have built your image and push it to the a library or Singulariy Hub. (If you have used the Remote Builder, it will provide the pull command)
 
```.bash
# Pull the image 
singularity pull $IMAGE_LOCATION
# e.g. singularity pull library://ertugungor/default/image:latest

# Runs `run.py`in the container
singularity exec library://ertugungor/default/image:latest python3 run.py        
```

- GPU support and mounting data volumes:
```.bash
# --nv # Nvidia support
# --bind $LOCAL_DIR:$CONTAINER_DIR. This parameter makes the files on path $LOCAL_DIR
# in your local file system available on path $CONTAINER_DIR in the container. Comma
# separate the path pairs for multiple binding.

singularity exec \
        --nv \ 
        --bind /home/ertugrul/my_data:/workspace/data, \
		       /home/ertugrul/my_data_2:/workspace/another_data \
		library://ertugungor/default/image:latest python3 run.py
```

For more, please refer to the [Quick Start](https://sylabs.io/guides/3.0/user-guide/quick_start.html) documentation.

