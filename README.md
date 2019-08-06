# docker-tf-airflow-stack
A Docker AI software stack based on TensorFlow / TFX / Jupyter / Airflow.

## Getting Started

The goal of this project is to provide guidelines (and some config file) to deploy with low effort a software stack for data scientists based on Docker.

Different references are already available on the Web for the configuration of each of the aforementioned components, but not all these docs are updated (e.g. TensorFlow on Docker 19.03), and they typically cover only the basic setup needs. 

Have a look at these guidelines if you are interested in any of these topics:
* you aim to setup a data science env on a single server for collaborative working
* you would like to leverage on GPUs deployed in your env using Docker. Indeed, these guidelines are focused on Docker 19.03, which natively supports NVIDIA GPU
* you are interested in Apache Airflow to orchestrate your ML pipelines
* you want to setup a pre-prod env, where you can both develop new models, then deploying them and finally validating the whole production cycle, since these guidelines install both a TensorFlow + Jupyter container ad a TensorFlow Serving TFX container.

### Prerequisites

What things you need to install the software and how to install them

```
Give examples
```

### Installing

A step by step series of examples that tell you how to get a development env running

Say what the step will be

```
Give the example
```

And repeat

```
until finished
```

End with an example of getting some data out of the system or using it for a little demo

## Running the tests

Explain how to run the automated tests for this system

### Break down into end to end tests

Explain what these tests test and why

```
Give an example
```

### And coding style tests

Explain what these tests test and why

```
Give an example
```

## Authors

* **Gabriele Randelli** - [grandelli](https://github.com/grandelli)

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details
