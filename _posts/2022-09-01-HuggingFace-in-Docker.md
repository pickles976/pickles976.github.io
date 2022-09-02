---
layout: post
title: Running HuggingFace Transformers in a Docker container
---

When I was working on a [project to try generating NES music with Huggingface transformers](https://www.youtube.com/watch?v=7pEvu_nC2LQ&ab_channel=Seabass), I was surprised by how little resources there were on making a compatible Docker image.  

Maybe it's because it's pretty standard practice to train toy models in Google colab, but what if I want to do inference with my trained model in some sort of server, in this case Flask?  

It turns out that setting up Docker with Python and CUDA enabled is pretty easy. Nvidia provides base images with various CUDA runtimes, so all we need to do is add Python and configure the GPU in our compose file.  

In your Dockerfile:  

FROM nvidia/cuda:11.6.0-runtime-ubuntu20.04

RUN apt-get update
RUN apt-get install -y software-properties-common gcc
RUN apt-get install -y python3.10 python3-distutils python3-pip python3-apt

EXPOSE 5000

COPY ./requirements.txt ./requirements.txt

CMD ["your", "commands", "here"]

In your docker-compose.yaml:  

version: "3.8"  # optional since v1.27.0
services:
web:
    build: .
    image: {your image name}
    ports:
    - "5000:5000"
    deploy:
    resources:
        reservations:
        devices:
            - driver: nvidia
            count: 1
            capabilities: [gpu]

Yes it's literally that easy. This setup should work for any models using transformers.py >= 4.5.1 or torch >= 1.6.0  

In my [project](https://github.com/pickles976/chiptune-ai) I used [ai-textgen](https://github.com/minimaxir/aitextgen) which I found a lot easier to use than regular GPT transformers as someone coming from a hobby background.