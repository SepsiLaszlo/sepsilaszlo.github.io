---
layout: post
title:  "Debugging a huge Docker overlay2 folder"
date:   2024-05-19 11:24:50 +0200
categories: docker
---

I was working on a Ruby on Rails application, and the project reached a point where we could deploy it to a staging environment. We containerized the application using Docker and deployed it, everything was working fine. After a few weeks, we noticed that we are almost out of storage on our Virtual Machine.

We had 50 GB of storage and our backup was around 1 GB. The backup included the database dump for PostgreSQL and the storage folder for the images managed by ActiveStorage. This meant that something other than the application data occupied most of the storage, so I started to investigate the issue. I also noticed while running the `docker compose up --build` command, the `COPY . .` step was extremely slow. 

I checked the root directory of the project and the Dockerfile, but I didn't find anything suspicious. So I wanted to check what files take up the disk space. At the beginning, I tried to use the `du` command on the root folder, to find some large folders on the VM, but it felt really inefficient, so I started a search for a better tool for the job.

I found [ncdu](https://dev.yorhel.nl/ncdu). The ncdu command line utility provides an interactive way to search for large folders in the file system. Using ncdu I could identify that over 40 GB of storage is occupied by `/var/lib/docker/overlay2` folder. The overlay2 folder is used by the [OverlayFS storage driver](https://docs.docker.com/storage/storagedriver/overlayfs-driver) to store image and container layers.

![ncdu-example](/images/docker-overlay2.png)

My first idea was to prune all unused images, but it did not help.

```bash
docker system prune --all
```
After that, I checked the overlay2 folder. I discovered that I can drill down more to see the contents of each layer. I noticed that the largest file in the layer was db_dump.zip which should not be there. Gotcha!

```txt
--- /var/lib/docker/overlay2/8d9hltfi3vom14228wfxz3t6d/diff/my-rails-app -------------
                         /..
    1.0 GiB [##########] /db_dump.zip
   56.1 MiB [          ] /.git
   22.0 MiB [          ] /db
   13.5 MiB [          ] /app
    2.1 MiB [          ] /public
  200.0 KiB [          ] /test
  172.0 KiB [          ] /config
   84.0 KiB [          ] /spec
   28.0 KiB [          ] /bin
   16.0 KiB [          ] /lib
   12.0 KiB [          ] /tmp
   12.0 KiB [          ]  Gemfile.lock
    8.0 KiB [          ] /vendor
    4.0 KiB [          ] /log
    4.0 KiB [          ]  Gemfile
    4.0 KiB [          ]  README.md
``` 
So the issue was introduced when we started to crate automatic backups every midnight. The backup was placed to the working directory of the project, so every build copied them into the container, which added a new large layer to the overlay2 folder. To fix this, I improved the backup procedure to delete the compressed files after it uploaded them to S3. This prevented the overlay2 folder from expanding further, but there were several large unused layers that still contained the backup and occupied the storage. 

I tried to stop and remove the containers and images from the VM and prune the docker build cache, but it had no effect. I manually checked if these layers are in use. To do this, I printed the output of the inspect command to a file for each container and image. 

```shell
docker inspect $(docker ps -aq) > all_container_and_images.json
docker inspect $(docker images -q) >> all_container_and_images.json
```
**WARNING - modifying the contents in the overlay2 folder could break docker or lead to data loss!** 

Then I searched in for the layers, that I needed to remove. Fortunately they were no longer in use, so I could use `rm -rf` to remove these layers.
