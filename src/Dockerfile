# Sample dockerfile to show root user privileges
FROM ubuntu:latest
RUN apt-get update
RUN apt-get install -y python python3-pip wget
RUN pip install Flask
# Add a nonroot user named as nonroot
RUN useradd -ms /bin/bash nonroot
#Run Container as nonroot
USER nonroot