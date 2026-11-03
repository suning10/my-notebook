# AMI (AWS Machine Image)
- Customize EC2 instance (software, operating system...)
- faster boot, pre-packed software
- specific to AZ (can copy from one to another)
- need to maintain ourself

## How to use AMI
- start EC2 and customize
  - create a image
  - can view from image ->AMI
- stop it
- build an AMI -> this will create snapshot
- launch instance from another AMI
  - create instance -> from AMI

## difference between Docker Image
- AMI Is a full virtual machine 
  - has Kernel + OS ...
  - output is EC2 instance 
  - VM Isolation 
- Docker
  - no kernel 
  - output is a Docker Container
  - for microservice