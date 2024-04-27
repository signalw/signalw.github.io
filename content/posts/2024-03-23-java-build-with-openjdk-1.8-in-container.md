+++
title = 'Build Java application with OpenJDK 1.8 inside a docker container'
date = 2024-03-23T00:00:00-04:00
comments = true
math = false
+++

If you write and build Java applications a lot, in this day of age, chances are that you may be building applications with different versions of Java. While there are a lot of solutions to maintain and switch between multiple JDK versions in your local environment, if you only need to perform a one-off task with a specific version, it is not always worth it if you may never use it again in the near future.

Recently I came across a project that I need to build with Java 8 and Gradle 4. I do not have that as our other projects are already modernized to use Java 11 and Java 17. Since this is a one-off task, I find that using docker container for the build is super easy and clean. Here are the steps.

1. Find the correct Docker image. The Eclipse Temurin project provides numerous images for different platforms with different Java versions. Here is their official site on Docker: https://hub.docker.com/_/eclipse-temurin. The image I used is `eclipse-temurin:8`.

2. Launch a Docker container from the image and obtain a shell. Remember that the path to the project need to be mounted so that it can be accessible inside the container.
```
$ docker run --rm -v /path/to/project:/path/to/project -it eclipse-temurin:8 /bin/bash
Unable to find image 'eclipse-temurin:8' locally
8: Pulling from library/eclipse-temurin
71dca2167f9f: Pull complete
099ab792921b: Pull complete
ef7620c0cc62: Pull complete
0cb2ab6d6d84: Pull complete
3b4ca9405ba8: Pull complete
Digest: sha256:b7e1c71ef083272612ac43c2fc36977c08a3479e23fa09cdaebdf2a1c7fae096
Status: Downloaded newer image for eclipse-temurin:8
```

3. Change directory into the project and start building your application. You can make sure that it has the correct Java version.
```
cd /path/to/project
root@2b30416dcdf5:/path/to/project# java -version
openjdk version "1.8.0_402"
OpenJDK Runtime Environment (Temurin)(build 1.8.0_402-b06)
OpenJDK 64-Bit Server VM (Temurin)(build 25.402-b06, mixed mode)
root@2b30416dcdf5:/path/to/project# ./gradlew build
...
```
