# docker-tf-airflow-stack
A Docker AI software stack based on TensorFlow / TFX / Jupyter / Airflow.

## Getting Started

The goal of this project is to provide guidelines (and some config file) to deploy with low effort a software stack for ML practitioners based on Docker.

Different references are already available on the Web for the configuration of each of the aforementioned components, but not all these docs are updated (e.g. running TensorFlow GPU version with Docker CE 19.03), and they typically cover only the basic setup needs. 

Have a look at these guidelines if you are interested in any of these topics:
* you aim to setup a data science env on a single server for collaborative working
* you would like to leverage on GPUs deployed in your env using Docker. 
* you are interested in Apache Airflow to orchestrate your ML pipelines
* you want to setup a pre-prod env, where you can both develop new models, then deploying them and finally validating the whole production cycle, since these guidelines install both a TensorFlow + Jupyter container ad a TensorFlow Serving TFX container.
* you want to operate Docker as non-root user

## Prerequisites
Heads-up:
* this procedure has been validated on Linux Centos 7. While I don't see any significant problem in running the same Docker commands in other environments, installation of pre-requisite packages / configuration procedures might require a different syntax (e.g. on Ubuntu-like platforms).
* guidelines are focused on Docker 19.03, the first version natively supporting NVIDIA GPU
* guidelines are mainly conceived for those willing to exploit GPUs on Docker. However, I will also provide alternative instructions for CPU-only envs

One very important aspect to highlight is that, **in order to use GPU, you need to install CUDA Toolkit in your env**. These guidelines won't describe how to install CUDA. Please refer to [NVIDIA documentation](https://docs.nvidia.com/cuda/archive/10.0/) for this topic.

Furthermore, **it is mandatory to respect the versioning matrix between CUDA and TensorFlow**, if you don't want to waste a lot of time.
Thereby, below I'm reporting the different product versions adopted:
* Docker CE 19.03 [web](https://docs.docker.com/install/)
* Docker-Compose 1.24.1 [web](https://docs.docker.com/compose/)
* Apache Airflow: 1.10.3 [web](https://github.com/puckel/docker-airflow)
* Tensorflow: 1.14 [web](https://hub.docker.com/r/tensorflow/tensorflow/)
* Tensorflow Serving TFX: 1.14 [web](https://hub.docker.com/r/tensorflow/serving)
* CUDA Toolkit: 10.0 (please, avoid using 10.1!) [web](https://docs.nvidia.com/cuda/archive/10.0/)

I'm assuming you know basic Linux and Docker commands here.

## Installation

### Docker
Please refer to the official Docker [documentation](https://docs.docker.com/install/) for additional info.

As already mentioned, I'll be explicitly referring to Docker CE 19.03. Its advantage is to natively suppport NVIDIA GPUs, hence lowering the effort to enable GPU support.

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
1. I don't personally like working as root user, hence let's configure Docker to accept commands from normal users
2. Docker by default is installed in /var/lib/docker folder. What if you have a dedicated partition and would like Docker to store all the images over there? We need to change this as well.

##### Non-root User
If not already available, as root create docker group on Linux:
```
# groupadd docker
```

Then, add the user you want to use to this group:
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

Now upload on your server the file daemon.json to /etc/docker folder (create it if not existing). Edit the file and change the following line:
```
    "data-root": "/home/docker",
```

replacing /home/docker with your destination folder.
Last, restart the service:
```
# systemctl start docker
```

### NVIDIA Docker
Skip this section if your env is not equipped with GPUs, or you're not interested in running TensorFlow on GPUs!

The NVIDIA Docker plugin enables deployment of GPU-accelerated applications across any Linux GPU server with NVIDIA Docker support. Official NVIDIA Docker documentation is available [here](https://github.com/NVIDIA/nvidia-docker).

As already mentioned, life is much easier with Docker 19.03, since the old nvidia-docker package is not anymore required. We need to install only one more package, the NVIDIA Container Toolkit, by typing (as root):

```
# distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
# curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.repo | sudo tee /etc/yum.repos.d/nvidia-docker.repo
# sudo yum install -y nvidia-container-toolkit
# sudo systemctl restart docker
```

Pay attention to restart Docker , or you won't see any difference.
In order to test if it worked out, type (as non root):
```
$ docker run --gpus all nvidia/cuda:10.0-base nvidia-smi
```

The important param is **--gpus all**, which comes out with the new Docker version.

### Docker-Compose
Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration.

Within the scope of this project, since we have four different containers, we will be relying on Compose.

**Heads-up**: the current version of Compose (v1.24.1 - 2019/08/06) is not compliant to the the new GPU capabilities of Docker CE 19.03 (--gpus param), hence we can't run GPU-enabled TensorFlow images through Compose. Nevertheless, it's a good exercise to have a YML file orchestrating all of them. In case your ML training is really consuming and you need GPUs, I will provide commands to run TF containers with GPU enabled without Composer. Obviously, this is not relevant for those of you interested in CPU-only TensorFlow.

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

## ML Stack Configuration & Startup
The first thing to do is to upload the Docker-compose configuration file docker-compose.yml in a folder accessible from your non-root user. Let's highlight few relevant sections of this file before starting it up:

The first important remark is that Dockers's container should be **stateless** and without data persistency. As a matter of fact, a good practice in favour of container re-use and sharing is never to commit any change. If so, where are we storing our trained models, our Python code, or our Airflow DAGs?
Docker owns the concept of [volume](https://docs.docker.com/storage/volumes/), folders residing on the host file system, which can be mounted to the containers at startup. These folders will allow for persistency.

Wherever in my Docker-Compose file you find out a section like this:
```
        volumes:
            - ${HOME}/airflow-dags:/usr/local/airflow/dags
```
you know I'm mounting a local file system folder as a container folder.

The second remark is that one advantage of Docker-Compose over plain Docker is the automatic [networking](https://docs.docker.com/compose/networking/) functionality to bridge the different containers and let them interact. Imagine the following scenario in our ML stack: we have a ML model already trained and deployed in the TensorFlow Serving container. We would like to test it by submitting some prediction requests, and we would like to create these requests through the TensorFlow Client container, using Jupyter notebook. Needless to say, we need to invoke the Serving container, which turns into networking between the client and the server. Here is where Compose steps up. In the Compose YML file each container has a name, which is also the reachable hostname:
```
   tf-serving:
        image: tensorflow/serving:latest-gpu
```

### Airflow Container
Airflow container is named **webserver** and is devoted to ML pipelines orchestration and execution. It relies on another Postgres container. 

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

### TensorFlow & Jupyter Container
TensorFlow & Jupyter (**tf-jupyter**) is the container devoted to ML development, model training and evaluation.

Some comments:

1. Jupyter notebook runs on port **8888**, and for the ease of access the JUPYTER_TOKEN is statically set as **jupyter**
2. We are relying on two different volumes, one to store trained models and the other one to save Jupyter notebooks:
```
        volumes:
            - ${HOME}/notebooks:/tf/notebooks
            - ${HOME}/tf-models:/tf/models
```
3. The tricky part... the Docker image to use:

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

**Head-up**: even if you're adopting the GPU version you have to understand, as already mentioned, that Docker-COmpose currently **does not implement native Docker GPU support**. Thereby, if you really need to use GPUs, use Docker-Compose only to start Airflow, then adopt plain Docker to launch TensorFlow & Jupyter Client. A command example is:
```
$ docker run --gpus all -it --rm -v /home/randelli/notebooks:/tf/notebooks -p 8888:8888 tensorflow/tensorflow:latest-gpu-py3-jupyter
```
**Head-up**: Jupyter is a local notebook, which is typically accessible through local host at http://localhost:8888. Since in our case we are running on a remote server, we need to establish an SSH tunnel to access it. Not going into details on how to establish an SSH tunnel here.

### TensorFlow Serving (TFX) Container
TensorFlow Serving (**tf-serving**) is the container devoted to deploy and run ML trained models.

Some comments:

1. TensorFlow Serving runs on port **8501**
2. Unfortunately when launching TensorFlow Serving we need to explivitly address the desired model, hence we should have a single container per trained model. In the configuration file this requires to mount the specific folder where the model has been stored from the TensorFlow & Jupyter client. In my example I'm assuming to have already trained a model **fashion_model**, but feel free to rename it as you want:
```
        volumes:
            - ${HOME}/tf-models:/models/fashion_model
```
3. The tricky part... the Docker image to use:

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

**Head-up**: even if you're adopting the GPU version you have to understand, as already mentioned, that Docker-COmpose currently **does not implement native Docker GPU support**. Thereby, if you really need to use GPUs, use Docker-Compose only to start Airflow, then adopt plain Docker to launch TensorFlow Serving TFX. A command example is:
```
$ docker run --gpus all -it --rm -v /home/randelli/notebooks:/tf/notebooks -p 8888:8888 tensorflow/serving:latest-gpu
```

### Service Startup
To launch all the configured services, simply enter the folder where your docker-compose.yml is stored and type:
```
$ docker-compose -f docker-compose.yml up -d
```
If you want to stop all of them, similarly type:
```
$ docker-compose stop
```

## Future Improvements
* To analyze the adoption of Jupyter Hub to remove the local constraint of Jupyter Notebook
* To update the guide as soon as there will be a new Docker-Compose release fully compliant to Docker's native GPU support
* To run a single TensorFlow Serving container supporting multiple trained models in order not to hardcode the trained model name in the Compose config file

## Authors

* **Gabriele Randelli** - [grandelli](https://github.com/grandelli)

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details
