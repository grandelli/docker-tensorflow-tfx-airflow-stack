# docker-tf-airflow-stack
A Docker AI software stack based on TensorFlow / TFX / Jupyter / Airflow.

## Getting Started

The goal of this project is to provide guidelines (and some config file) to deploy with low effort a software stack for data scientists based on Docker.

Different references are already available on the Web for the configuration of each of the aforementioned components, but not all these docs are updated (e.g. TensorFlow on Docker 19.03), and they typically cover only the basic setup needs. 

Have a look at these guidelines if you are interested in any of these topics:
* you aim to setup a data science env on a single server for collaborative working
* you would like to leverage on GPUs deployed in your env using Docker. 
* you are interested in Apache Airflow to orchestrate your ML pipelines
* you want to setup a pre-prod env, where you can both develop new models, then deploying them and finally validating the whole production cycle, since these guidelines install both a TensorFlow + Jupyter container ad a TensorFlow Serving TFX container.
* you want to operate Docker as non-root user

### Prerequisites
Heads-up:
* this procedure has been validated on Linux Centos 7. While I don't see any significant problem in running the same Docker commands in other environments, installation of pre-requisite packages / configuration procedures might require a different syntax (e.g. on Ubuntu-like platforms).
* guidelines are focused on Docker 19.03, the first version natively supporting NVIDIA GPU
* guidelines are mainly conceived for those willing to exploit GPUs on Docker. However, I will also provide alternative instructions for CPU-only envs

One very important aspect to highlight is that, **in order to use GPU, you need to install CUDA Toolkit in your env**. These guidelines won't describe how to install CUDA. Please refer to [NVIDIA documentation](https://docs.nvidia.com/cuda/archive/10.0/) for this topic.

Furthermore, **it is mandatory to respect the versioning matrix between CUDA and TensorFlow**, if you don't want to waste a lot of time.
Thereby, below I'm reporting the different product versions adopted:
* Apache Airflow: 1.10.3 [web](https://github.com/puckel/docker-airflow)
* Tensorflow: 1.14 [web](https://hub.docker.com/r/tensorflow/tensorflow/)
* Tensorflow Serving TFX: 1.14 [web](https://hub.docker.com/r/tensorflow/serving)
* CUDA Toolkit: 10.0 (please, avoid using 10.1!) [web](https://docs.nvidia.com/cuda/archive/10.0/)

### Docker
Please refer to the official Docker [documentation](https://docs.docker.com/install/) for additional info.

As already mentioned, I'll be explicitly referring to Docker 19.03. Its advantage is to natively suppport NVIDIA GPUs, hence lowering the effort to enable GPU support.

Linux pre-requisite packages to install are:
``` sh
# yum install -y yum-utils \
device-mapper-persistent-data \
lvm2
```

Docker 19.03 is not the default version supported by Centos. Hence, let's start by removing existing versions on your env:
``` sh
# yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

Add the official Docker repository:
``` sh
# yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

Install Docker 19.03:
``` sh
# yum install docker-ce docker-ce-cli containerd.io
# systemctl start docker

```
To check if setup was successful, type the following:
```
# docker --version
Docker version 19.03.1, build 74b1e89
```

Finally,  run docker's "hello world":
```
# docker run hello-world
```

So far so good (if you didn't get any error)... congrats!

Now it's time to fine-tune our Docker installation. Throughout this section we will dig into the following configurations:
* I don't personally like working as root user, hence let's configure Docker to accept commands from normal users
* Docker by default is installed in /var/lib/docker folder. What if you have a dedicated partition and would like Docker to store all the images over there? We need to change this as well.

## Authors

* **Gabriele Randelli** - [grandelli](https://github.com/grandelli)

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details
