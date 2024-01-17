# Run x86 on ARM -- Docker

Create a new docker container such a way that the network is shared with the host `--net=host`. To list docker containers run `sudo docker ps --all`. To start a container run `sudo docker start <container_name>` & to attach to it run `sudo docker attach <container_name>`. Build a new docker container:
```bash
sudo apt install docker.io -y
# create a file called "Dockerfile"
docker build -t my-kali-image .
# then run the container. Every time you run the container a new container will be formed. 
```

### Contents of `Dockerfile`:

```bash
# Use Kali Linux rolling edition as the base image
FROM kalilinux/kali-rolling

# Suppress errors related to invoke-rc.d during package installation
RUN echo '#!/bin/sh\nexit 0' > /usr/sbin/policy-rc.d

# Run updates and install packages

# Adding new architecture before the first update and then installing all packages
RUN dpkg --add-architecture amd64 && \
    apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y \
        kali-linux-headless \
        qemu-user-static \
        binfmt-support \
        libc6:amd64 \
        proxychains4:amd64 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Set the default command to Bash
CMD ["/bin/bash"] 
```

```bash
docker run --net=host -it kalilinux/kali-rolling /bin/bash
# If the command ran successfully, you should be inside a docker container using kali image.

# Run the following inside the docker container
apt update && apt -y install kali-linux-headless
apt install -y qemu-user-static binfmt-support
dpkg --add-architecture amd64 
apt update
apt install libc6:amd64
apt install -y qemu-user-static binfmt-support

# Then try
apt install proxychains4:amd64
```

### When things stop working inside the container:

```bash
mv /var/lib/dpkg/info /var/lib/dpkg/info_old
mkdir /var/lib/dpkg/info
apt-get update && apt-get -f install
mv /var/lib/dpkg/info/* /var/lib/dpkg/info_old
rm -rf /var/lib/dpkg/info
mv /var/lib/dpkg/info_old /var/lib/dpkg/info
```

### Running files inside QEMU:

```bash
qemu-x86_64-static ./ssh_client
# qemu does not understand files in $PATH so use absolute paths.
qemu-x86_64-static /usr/bin/proxychains4
```

