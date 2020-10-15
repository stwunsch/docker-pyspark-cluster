# How to run a (Py)Spark cluster in standalone mode with docker

This repository explains how to run (Py)Spark 3.0.1 in standalone mode using docker. The standalone mode ([see here](https://spark.apache.org/docs/latest/spark-standalone.html)) uses a master-worker architecture to distribute the work from the application among the available resources.

## Install the required software

For the following instructions, I'll assume you have `python3`, `docker` and a Linux distribution running.

```bash
# Create and source a virtual Python environment
python3 -m venv py3_venv
source py3_venv

# Install the dependencies from the requirements.txt
pip install requirements.txt
```

## Have a look at the Dockerfile

The `Dockerfile` ([see here](Dockerfile)) installs the required packages and downloads from [https://downloads.apache.org/](https://downloads.apache.org/) the Spark release 3.0.1. After pointing with the environment variables `PATH` and `SPARK_HOME` to the executables, the container is ready to run the master and worker services. Note that Spark ships with the scripts `start-master.sh` and `start-slave.sh`, which will be used in the following.

## Have a look at the docker-compose.yml

The [`docker-compose.yml`](docker-compose.yml) has all information required to run the standalone Spark cluster. The master service publishes the ports 7077 for Spark and 8080 for the web UI and is simply started with `start-master.sh` delivered with Spark itself. The additional `tail -F /dev/null` is just there to keep the container alive since the Spark process is started in the background. The workers are started the same way but with `start-slave.sh spark://spark-master:7077`. `spark-master` is the container name given to the master and also serves as hostname in the set up network. The number of cores and allocated memory per worker can be set via the environment variables `SPARK_WORKER_CORES` and `SPARK_WORKER_MEMORY`. Also note the dedicated network `spark-net`, which is set up in the configuration file. This dedicated network is required to link the master and the workers so that they can talk to each other. The network is built in `bridge` mode, which only supports linking containers on the same host. For a multi-node setup you have to built such a network in the `overlay` mode.

## Build the containers and run the standalone Spark cluster

The helper `docker-compose` is able to spawn a multi-container application in a coordinated way. Also, the application allows to scale the number of workers easily with `--scale`, see the following commands to build and run the containers.

```bash
# Build the required images
docker-compose build

# Execute the Spark master and 4 workers
docker-compose up --scale worker=4
```

## Test it!

You'll find [here](test.py) a small test script, which calculates Pi with the just set up standalone Spark cluster. Take care that you set the variable `hostname_master` to the hostname of the device, which runs the cluster. In the script, I assume you run it on `localhost`. Also note the additional configurations of the `SparkSession` setting the hostname of the Spark driver, which submits the tasks to the cluster, which is in our case spawned with running the example Python script.

```bash
python3 test.py
```
