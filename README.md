# A Docker AI stack based on TensorFlow / TFX / Jupyter / Airflow
A Docker AI software stack based on TensorFlow / TFX / Jupyter / Airflow.

Latest Update: 2019-08-07

## Getting Started

The goal of this project is to provide a guide (and some configuration file) to deploy with low effort an AI software stack for ML practitioners willing to ease the installation effort via Docker.

Different references are already available on the Web for the configuration of each of the aforementioned components, but not all of them are updated (e.g. the TensorFlow official documentation for the GPU version with Docker is out of date), and they typically cover only the basic setup steps. 

Have a look at this guide if you are interested in any of these topics:
* you aim to setup a data science env on a single remote server for collaborative working
* you would like to leverage on GPUs deployed in your env using Docker. 
* you are interested in Apache Airflow to orchestrate your ML pipelines
* you want to setup a single env where you can test the whole ML design life cycle: developing new models, deploying them in production-like stage, finally orchestrating the different training and prediction pipelines
* you want to operate Docker as non-root user

## Prerequisites
* this procedure has been validated on Linux Centos 7. While I don't see any significant problem in running the same Docker commands in other Linux-like environments, installation of pre-requisite packages / configuration procedures might require a different syntax (e.g. *apt-get* instead of *yum* on Ubuntu-like platforms).
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

Please refer to our [WiKi](https://github.com/grandelli/docker-tensorflow-tfx-airflow-stack/wiki) for setup, configuration and running.
