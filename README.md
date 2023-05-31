# test-tensorflow-rocm
A tutorial for installing the rocm/tensorflow docker image and a script to test if tensorflow is working on your GPU, intended for Arch Linux.

## Prerequisites
* An internet connection.
* More than 25 GB of storage. The extracted rocm/tensorflow image is 24.3GB. I got this working on a 30 GB partition, but it's close.

## Necessary packages
```
sudo pacman -S base linux linux-firmware docker
```
You apparently do not need to do anything with kernel modules? I was very surprised, but I guess Arch has that support baked in.

I also had installed `vim networkmanager efibootmgr dust`, but you shouldn't need those for this tutorial.

## Start docker.
```
sudo systemctl start docker
```

## Download the image.
Sometimes, `docker pull` errors like "unauthorized: authentication required", maybe from clock drift? Enabling systemd-timesyncd may fix it.
```
sudo systemctl enable systemd-timesyncd --now
```

If your downloads get interrupted a lot, maybe enable the experimental containerd-snapshotter, which should let `docker pull` resume after interruption.
However, this is very experimental and causes problems, like downloading everything at once and downloading duplicates.
If you only have 30 GB of storage, your pull will fail due to not enough space even though a normal pull would succeed.
```
sudo mkdir /etc/docker
sudo vim /etc/docker/daemon.json
# Insert:
{ "features": { "containderd-snapshotter" : true } }
```
See also my post on SO: https://stackoverflow.com/a/76375406/6286797

Then download the image:
```
docker pull rocm/tensorflow:latest
```

## Run the image.
The `-it` flag in drun gives you an interactive terminal inside the container. Consider adding the alias to .profile or .bash_aliases or whatever. Copied from https://hub.docker.com/r/rocm/tensorflow
```
alias drun='sudo docker run -it --network=host --device=/dev/kfd --device=/dev/dri --ipc=host --shm-size 16G --group-add video --cap-add=SYS_PTRACE --security-opt seccomp=unconfined -v $HOME/dockerx:/dockerx'
drun rocm/tensorflow:latest
```

## Test if your GPU is working.
```
git clone https://github.com/mpeschel10/test-tensorflow-rocm
cd test-tensorflow-rocm
python test_tensorflow.py
```


