# MNIST example - Keras version
This classic example of hand-written text recognition is well suited both as a lightweight test when learning FEDn and developing on FEDn in psedo-distributed mode. A normal high-end laptop or a workstation should be able to sustain at least 5 clients. The example is also useful for general scalability tests in fully distributed mode. 

> Note that this example shows how to configure FEDn for training, and how to configure and start clients. We assume that a FEDn network is aleady up and running with a blank, unconfigured Reducer. If this is not the case, start here: https://github.com/scaleoutsystems/fedn/blob/master/README.md

## Local training and test data
This example ships with the mnist dataset from https://s3.amazonaws.com/img-datasets/mnist.npz in 'data/mnist.npz'.
This dataset will be partitioned to simulate unique datasets for each client. 

## Configuring the Reducer  
Navigate to 'https://localhost:8090' (or the url of your Reducer) and follow instructions to upload the compute package in 'package/package.tar.gz' and the initial model in 'initial_model/initial_model.npz'.

## Create partitions
To simulate the scenario of each client using a unique dataset we will make partitions in 'data/clients' of the complete dataset in 'data/mnist.npz'. The python script below will create 10 partitions:

``` bash
python create_data_partitions.py
```
Alternativly, create the disired number of partitions (e.g. 5):

``` bash
python create_data_partitions.py 5
```

## Attaching a client to the federation

1. First, download 'client.yaml' from the Reducer 'Network' page, and replace the content in your local 'client.yaml'. 
2. Start a client. Here there are different options (see below): 
    - Docker 
    - docker-compose
    - [Native client (OSX/Linux)](https://github.com/scaleoutsystems/examples/tree/main/how-tos/start-native-fedn-client)

### Docker
1. Build the image

``` bash
docker build . -t mnist-client:latest
```

2. Start a client (edit the path of the volume mounts to provide the absolute path to your local folder.)
```
docker run -v /absolute-path-to-this-folder/data/clients/0:/app/data:ro -v /absolute-path-to-this-folder/client.yaml:/app/client.yaml --network fedn_default mnist-client fedn run client -in client.yaml --name client0
```
(Repeat above steps as needed to deploy additional clients but change the data partition (e.g. .../clients/1) and the --name).

### docker-compose
To start 2 clients: 

```bash
docker-compose -f docker-compose.yaml -f private-network.yaml up 
```

> If you are connecting to a Reducer with a distributed setup or in Studio, you should omit 'private-network.yaml'. 

### Start training 
When clients are running, navigate to the 'Control' page of the Reducer to start the training. 

### Configuring the client
We have made it possible to configure a couple of settings to vary the conditions for the training. These configurations are expsosed in the file 'settings.yaml': 

```yaml 
# How much to bias the client data samples towards certain classes (non-IID data partitions)
bias: 0.7
# Parameters for local training
batch_size: 32
epochs: 1
```

## Creating a compute package
Whenever you make updates to the client code (such as altering any of the settings in the above mentioned file), you need to re-package the compute package:

```bash
tar -czvf package.tar.gz client
```
To clear the system and set a new compute package, see: https://github.com/scaleoutsystems/fedn/blob/master/docs/FAQ.md

For an explaination of the compute package structure and content: https://github.com/scaleoutsystems/fedn/blob/develop/docs/tutorial.md
 
## Creating a new initial model
The baseline CNN is specified in the file 'client/init_model.py'. This script creates an untrained neural network and serializes that to a file.  If you wish to alter the initial model, edit 'init_model.py' and 'models/mnist_model.py' then regenerate the initial model file (install dependencies as needed, see requirements.txt):

```bash
python init_model.py
```

## When you are done
 Don't forget to tear down docker-compose resources, e.g.:
 ```bash
docker-compose -f docker-compose.yaml -f private-network.yaml down
 ```

## Troubleshooting

### Tensorflow error when building docker image

The error message
 ```bash
 ERROR: No matching distribution found for tensorflow>=2.3.2
 ```
could be caused by your device having a processor with arm architecture. 
This will make Docker download images compiled for aarch64 platform (if another platform is not specified), 
and unfortunately Tensorflow doesn't support the aarch64 platform. 
So a solution is to simply specify another platform that works with Tensorflow. 
This can be done by changing to e.g. python:3.8.9-bullseye in the Dockerfile, as below.
  ```bash
FROM python:3.8.9-bullseye
COPY requirements.txt /app/
WORKDIR /app
RUN pip install -r requirements.txt
RUN pip install -e git://github.com/scaleoutsystems/fedn.git@develop#egg=fedn\&subdirectory=fedn ```
 ```

## License
Apache-2.0 (see LICENSE file for full information).

