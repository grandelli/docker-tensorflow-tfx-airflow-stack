# A Docker AI stack based on TensorFlow / TFX / Jupyter / Airflow
A Docker AI software stack based on TensorFlow / TFX / Jupyter / Airflow.

## Getting Started

The goal of this project is to provide a guide (and some configuration file) to deploy with low effort an AI software stack for ML practitioners willing to ease the installation effort via Docker.

Different references are already available on the Web for the configuration of each of the aforementioned components, but not all these docs are fully updated (e.g. the TensorFlow official documentation for the GPU version with Docker is out of date), and they typically cover only the basic setup steps. 

Have a look at this guide if you are interested in any of these topics:
* you aim to setup a data science env on a single remote server for collaborative working
* you would like to leverage on GPUs deployed in your env using Docker. 
* you are interested in Apache Airflow to orchestrate your ML pipelines
* you want to setup a single env where you can test the whole ML design life cycle: developing new models, deploying them and finally orchestrating the different training and prediction pipelies
* you want to operate Docker as non-root user

## Prerequisites
* this procedure has been validated on Linux Centos 7. While I don't see any significant problem in running the same Docker commands in other Linux-like environments, installation of pre-requisite packages / configuration procedures might require a different syntax (e.g. on Ubuntu-like platforms).
* this guide is focused on Docker CE 19.03, the first version with native support for NVIDIA GPUs
* guidelines are mainly conceived for those willing to exploit GPUs on Docker. However, I will also provide alternative instructions for CPU-only configuration

One very important aspect to highlight is that, **in order to use GPUs, you need to install CUDA Toolkit in your env**. This guide won't describe how to install CUDA. Please refer to [NVIDIA documentation](https://docs.nvidia.com/cuda/archive/10.0/) for this topic.

Furthermore, **it is mandatory to respect the versioning matrix between CUDA and TensorFlow**, if you don't want to waste a lot of time.
Below I'm reporting the different product versions adopted in this guide:
* Docker CE 19.03 [web](https://docs.docker.com/install/)
* Docker-Compose 1.24.1 [web](https://docs.docker.com/compose/)
* Apache Airflow: 1.10.3 [web](https://github.com/puckel/docker-airflow)
* Tensorflow: 1.14 [web](https://hub.docker.com/r/tensorflow/tensorflow/)
* Tensorflow Serving TFX: 1.14 [web](https://hub.docker.com/r/tensorflow/serving)
* CUDA Toolkit: 10.0 (please, avoid using 10.1!) [web](https://docs.nvidia.com/cuda/archive/10.0/)

Finally, I'm assuming you know basic Linux and Docker commands.

## Installation

### Docker
Please refer to the official Docker [documentation](https://docs.docker.com/install/) for additional info.

As already mentioned, I'll be explicitly referring to Docker CE 19.03. Its advantage is to natively suppport NVIDIA GPUs, lowering the effort to leverage on GPUs.

Linux pre-requisite packages to install are:
``` sh
# yum install -y yum-utils \
device-mapper-persistent-data \
lvm2
```

Docker CE 19.03 is not the default version supported by Centos. Hence, let's start by removing existing versions on your env:
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

Install Docker CE 19.03:
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

#### Docker Configuration

Now it's time to fine-tune our Docker installation. Throughout this section we will dig into the following configurations:
1. I don't personally like working as root user, hence let's configure Docker to accept commands from non-root users
2. Docker is by default installed in */var/lib/docker* folder. What if you have a dedicated partition and would like Docker to store all the images over there? We need to change this as well.

##### Non-root User
If not already available, as root create a new *docker* group on Linux:
```
# groupadd docker
```

Then, add the non-root user you want to this group:
```
# usermod -aG docker [non-root user]
```

To check if everything worked fine, log as [non-root user] and try again the "hello world":
```
$ docker run hello-world
```

##### Move Docker's root folder
Let'start by stopping Docker's daemon (as root):
```
# systemctl stop docker
```

Change Docker's root folder:
```
# mv /var/lib/docker [dest folder]
```

Now upload on your server the file daemon.json to */etc/docker* folder (create it if not existing). Edit the file and change the following line:
```
    "data-root": "/home/docker",
```

replacing */home/docker* with your destination folder.
Last, restart the service:
```
# systemctl start docker
```

### NVIDIA Docker
Skip this section if your env is not equipped with any GPU, or you're not interested in running TensorFlow on GPUs!

The NVIDIA Docker plugin enables deployment of GPU-accelerated applications across any Linux GPU server with NVIDIA Docker support. Official NVIDIA Docker documentation is available [here](https://github.com/NVIDIA/nvidia-docker).

As already mentioned, life is much easier with Docker CE 19.03, since the old nvidia-docker package is not anymore required. We need to install only one more package, the NVIDIA Container Toolkit, by typing (as root):

```
# distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
# curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.repo | sudo tee /etc/yum.repos.d/nvidia-docker.repo
# sudo yum install -y nvidia-container-toolkit
# sudo systemctl restart docker
```

Pay attention to restart Docker , or you won't see any difference!
In order to test if it worked out, type (as non root):
```
$ docker run --gpus all nvidia/cuda:10.0-base nvidia-smi
```

The important param is **--gpus all**, which comes out with the new Docker version.

### Docker-Compose
Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration.

Within the scope of this project, since we have four different services, we will be relying on Compose.

**Heads-up**: the current version of Compose (v1.24.1 - 2019/08/06) is not compliant to the the new GPU capabilities of Docker CE 19.03 (--gpus param), hence we can't run GPU-enabled TensorFlow containers via Compose. Nevertheless, it's a good exercise to have a YML file orchestrating all of them. In case your ML training is really compute-demanding and you need GPUs, I will provide commands to run TF containers with GPU enabled without Composer. Obviously, this is not relevant for those of you interested in CPU-only TensorFlow.

To install Docker-Compose, type (as root):
```
- curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
- chmod +x /usr/local/bin/docker-compose
- ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose 
```

To test it, type (as non-root):
```
$ docker-compose --version
```

## AI Service Configuration & Startup
The first thing to do is to upload the Docker-Compose configuration file *docker-compose.yml* in a folder accessible from your non-root user. Let's highlight few relevant sections of this file before launching it.

The first important remark is that Dockers's container should be **stateless** and without data persistency. As a matter of fact, a good practice in favour of container re-use and sharing is to avoid any commit. Where are we storing our trained models, our Python code, or our Airflow DAGs?
Docker owns the concept of [volume](https://docs.docker.com/storage/volumes/), folders residing on the host file system, which can be mounted to the containers at startup. These folders will allow for data storage.

Wherever in my Docker-Compose file you find out a section like this:
```
        volumes:
            - ${HOME}/airflow-dags:/usr/local/airflow/dags
```
you know I'm mounting a local file system folder as a container folder.

The second remark is that one advantage of Docker-Compose over plain Docker is the automatic [networking](https://docs.docker.com/compose/networking/) functionality to bridge the different containers and let them interact. Imagine the following scenario in our AI application: we have a ML model already trained and deployed in the TensorFlow Serving container. We would like to test it by submitting some prediction requests, and we would like to create these requests through the TensorFlow & Jupyter service. Needless to say, we need to call the TF Serving node, which requires networking between the ctwo services. In the Compose YML file each container/service has a name, which correspond to its hostname in the bridged network:
```
   tf-serving:
        image: tensorflow/serving:latest-gpu
```

### Airflow Service
Airflow service is named **webserver** and is devoted to ML pipelines orchestration and execution. It relies on another Postgres container. 

Two comments on its current configuration are:

1. I personally avoid using port 8080, since this is a common Web Server port, hence I'm mapping service port 8080 as port **8088**
```
        ports:
            - "8088:8080"
```

2. As already described, we are relying on volumes to store DAGs. In particular:
```
        volumes:
            - ${HOME}/airflow-dags:/usr/local/airflow/dags
```

Once started up everything, you can access Airflow console at http://your-lab-ip:8088/admin/

### TensorFlow & Jupyter Service
TensorFlow & Jupyter (**tf-jupyter**) is the service devoted to ML development, model training and evaluation. It embeds TF libraries, as well as Jupyter notebook.

Some comments on this service:

1. Jupyter notebook runs on port **8888**, and for the ease of access the JUPYTER_TOKEN is statically set as **jupyter**
2. We are relying on two different volumes, one to store trained models and the other one to save Jupyter notebooks:
```
        volumes:
            - ${HOME}/notebooks:/tf/notebooks
            - ${HOME}/tf-models:/tf/models
```
3. The tricky part is the Docker image to use (change in the Docker-Compose config file accordingly):

        i. [TensorFlow GPU]
```
    tf-jupyter:
        image: tensorflow/tensorflow:latest-gpu-py3-jupyter
```

        ii. [TensorFlow CPU]
```
    tf-jupyter:
        image: tensorflow/tensorflow:latest-py3-jupyter
```

**Heads-up**: even if you're adopting the TF GPU version you have to understand, as already mentioned, that Docker-Compose currently **does not implement any native Docker GPU support**. Thereby, if you really need to use GPUs, use Docker-Compose only to start Airflow, then adopt plain Docker to launch TensorFlow & Jupyter Client. A command example is:
```
$ docker run --gpus all -it --rm -v ${HOME}/notebooks:/tf/notebooks -p 8888:8888 tensorflow/tensorflow:latest-gpu-py3-jupyter
```
**Heads-up**: Jupyter is a local notebook, which is typically accessible through local host at http://localhost:8888. Since in our case we are running on a remote server, we need to establish an SSH tunnel to access it. Not going into details on how to establish an SSH tunnel here.

Once started up everything, and created SSH tunneling, you can access Jupyter notebook at http://localhost:8888/

### TensorFlow Serving (TFX)
TensorFlow Serving (**tf-serving**) is the container devoted to deploy and run ML trained models.

Some comments:

1. TensorFlow Serving runs on port **8501**
2. Unfortunately, when launching TensorFlow Serving we need to explicitly address the desired model, hence we should have a single container per each trained model. In the configuration file this requires to mount the specific folder where the model has been stored from the TensorFlow & Jupyter client. In my example I'm assuming to have already trained a model **fashion_model**, but feel free to rename it as you want:
```
        volumes:
            - ${HOME}/tf-models:/models/fashion_model
```
3. The tricky part is the Docker image to use (change in the Docker-Compose config file accordingly):

        i. [TensorFlow Serving GPU]
```
    tf-serving:
        image: tensorflow/serving:latest-gpu
```

        ii. [TensorFlow Serving CPU]
```
    tf-serving:
        image: tensorflow/serving:latest
```

**Heads-up**: even if you're adopting the GPU version you have to understand, as already mentioned, that Docker-Compose currently **does not implement any native Docker GPU support**. Thereby, if you really need to use GPUs, use Docker-Compose only to start Airflow, then adopt plain Docker to launch TensorFlow Serving TFX. A command example is:
```
$ docker run --gpus all -it --rm -v /home/randelli/notebooks:/tf/notebooks -p 8888:8888 tensorflow/serving:latest-gpu
```

### Service Startup
To launch all the configured services, simply enter the folder where your *docker-compose.yml* is stored and type:
```
$ docker-compose -f docker-compose.yml up -d
```
If you want to stop services, type:
```
$ docker-compose stop
```

## Future Improvements
* Analyze the adoption of Jupyter Hub to remove the local constraint of Jupyter Notebook
* Update the guide as soon as there will be a new Docker-Compose release fully compliant to Docker's native GPU support
* Run a single TensorFlow Serving container supporting multiple trained models, not to hardcode the trained model name in the Docker-Compose config file

## Authors

* **Gabriele Randelli** - [grandelli](https://github.com/grandelli)

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details