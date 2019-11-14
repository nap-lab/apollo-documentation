# Working Remotely with GPU resources

Connected to the naplab, we have several GPU resources. This page will give you a short introduction to working remotely with GPU resources.



## Rules

For the nap01 server, please follow the following rules:

1. Use nvidia-docker to run jobs
2. When starting a docker container, name the container with {ntnu\_username}\_...
3. When creating a docker image, name the image {ntnu\_username}/image\_name
4. ALWAYS check nvidia-smi, to be certain that nobody is using the GPU you want to use 



## Using docker

To get a short introduction to docker, we recommend you to read through the [Open AI Lab's tutorial on docker. ](https://www.ntnu.no/wiki/display/ailab/Getting+started+with+Docker)

### Docker tips & tricks

Docker commands can become very long, with several static settings. To make your life easier, you can create simple python scripts to start a docker container. For example:

{% code title="run\_docker" %}
```python

#!/usr/bin/env python3
import sys
import os
import random
gpu_id = sys.argv[1]
python_args = " ".join(sys.argv[2:])

docker_name = str(random.randint(0, 10000))


docker_container = "haakohu_{}".format(docker_name) # Replace username with your ntnu username
pwd = os.path.dirname(os.path.abspath(__file__))

cmd = [
    "nvidia-docker", 
    "run",
    f"-u 1123514", # Set your user ID. 
    f"--name {docker_container}", # Set name of docker container
    "--ipc host", # --ipc=host is recommended from nvidia
    "--rm", # remove container when exited / killed
    f"-v {pwd}:/workspace", # mount directories. This mounts current directory to /workspace in the container
    f"-e CUDA_VISIBLE_DEVICES={gpu_id}", # Set GPU ID 
    "--log-opt max-size=50m", # Reduce memory usage from logs
    "-it", # Interactive
    "haakohu/pytorch", # Docker image
    f"{python_args}" # python command
]
command = " ".join(cmd)
print(command)
os.system(command)

```
{% endcode %}

There is a couple of important settings to change here:

1. Change docker\_name to your ntnu username `{NTNU-USERNAME}_...` 
2. Change the `-u` argument in the cmd list. You can find your ID by logging onto the server, then run `id -u ntnu_username` , for example `id -u haakohu`. This is to prevent the docker container to save files as administrator, which can easily mess up your project files.
3. The `-v` argument to mount folders. In the script, we are only mounting your current directory to /workspace in the docker container. If you need to mount something else, you can add several -v arguments
4. The docker image. 

Save this with the filename `run_docker`and make it an executable by running

```text
chmod +x run_docker
```

Then, I can start the training script with on GPU id 0:

```text
./run_docker_example 0 python -m deep_privacy.train
```

If you want to start a job without GPU, you can run:

```text
./run_docker_example "" python -m deep_privacy.train
```



This will execute the following docker cmd:

```text
nvidia-docker run --name haakohu_5556_other --ipc host --rm -v /home/haakohu/DeepPrivacy:/workspace -e CUDA_VISIBLE_DEVICES=8 --log-opt max-size=50m -it haakohu/pytorch python -m deep_privacy.train
```



#### Pre-built docker images

Nvidia GPU Cloud has several pre-built docker images for Nvidia systems:

{% embed url="https://ngc.nvidia.com/catalog/containers/nvidia:tensorflow" %}



## Mounting a server disk to your local filesystem

Working remotely can be a hassle without mounting the remote filesystem. If you mount a folder to your local computer, you can use your favorite texteditor to work on it. 

We recommend you to use sshfs:

{% embed url="https://github.com/libfuse/sshfs" %}



## Storage folders on nap01.idi.ntnu.no

For larger datasets, we recommend you to store your datasets on a different disk than the main SSD.

This can be found under `/work/ntnu_username`. To get a directory for your username, contact Håkon.



## Utilizing the full potential of V100 cards

The V-100 cards are extremely powerful, and requires optimized code to realise the full computing potential.

### 1. nvidia-smi

You can see the utilization of the gpu's by running `watch -n 0,5 nvidia-smi`. Your code should be running at 90% + utilization most of the time. 

### 2. Utilizing tensor cores

The V100 cards has about 600 tensor cores, which has some weird requirements. Most DL libraries run your operations automatic on tensor cores if you satisfy the following requirements:

1. Number of filters in your CNN is divisible by 8
2. Your batch size is divisble by 8
3. Your parameters/input data is floating point 16

The first two requirements are rather easy to satisfy, however, training a CNN with floating point 16 is hard. To get proper training of your network with 16 bit floating point, you are required to train with mixed precision training. Therefore, we recommend the following two resources to get started with this:

1. [https://devblogs.nvidia.com/video-mixed-precision-techniques-tensor-cores-deep-learning/\#part2](https://devblogs.nvidia.com/video-mixed-precision-techniques-tensor-cores-deep-learning/#part2)
2. [https://github.com/NVIDIA/apex](https://github.com/NVIDIA/apex) - Highly recommended for Pytorch users!

**With my code, I got a 220% speed up without loosing any performance.**

### 3. Profiling your code

If your code is running slow and you can't find the bottleneck, profiling is your best friend.

You can use tools like nvprof, but there exists profiling tools to different DL libraries. For pytorch, we have the module `torch.utils.bottleneck`: [https://pytorch.org/docs/stable/bottleneck.html](https://pytorch.org/docs/stable/bottleneck.html).

