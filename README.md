## CloudFormation templates for Batch and GPUs

These templates will build two things:

1. An AMI based on the AWS deep learning AMI that includes the ECS agent and an appropriate Docker configuration for 
   use with nVidia's GPUs

2. An AWS Batch compute environment along with a job queue that will exercise the GPU AMI

### Why?

This is here to help people get started using Batch with GPUs; in order to leverage the GPUs in the P and G family instances 
you need an AMI that has the ECS agent (along with Docker) and then a mechanism to export the drivers into the container so that
they can access the GPU. 

### How it works

#### AMI Creation 

1. Provisions an EC2 instance based on the Deep Learning AMI (current AMI is the default, you may need to change this based on region and availability)

2. Injects the nvidia-docker RPM into the running system

3. Pulls down the `nvidia/cuda` image with CUDA 9.0 and CUDNN 7

4. Verifies that it can run `nvidia-smi` in a container once it's pulled the image

5. Links the latest driver to `/var/lib/nvidia-docker/volumes/nvidia_driver/latest` so that it can easily be used by Batch containers

6. Signals to the `AMICreate` wait condition when it's done (which, in turn invokes Lambda to create the AMI)

7. Outputs the new AMI ID for use in the `batch-environment.yaml` template


#### Batch Environment 

The Batch environment template creates a Batch compute environment that is managed but limited to a specific type of GPU-compatible instance 
(by default `g3.4xlarge` instances but you can change that or use a list). 

It also creates a job queue that uses this new GPU-enabled compute environment and a job definition that mounts the GPU drivers from above 
located in `/var/lib/nvidia-docker/volumes/nvidia_driver/latest` as `/usr/local/nvidia` inside the container and exercises `nvidia-smi` 
to test that the GPU works. 
