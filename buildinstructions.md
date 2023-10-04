# Building meta-ros

## Pre-requisites
* Ubuntu 22.04.1 LTS (Jammy Jellyfish)
* XXX GB of storage space
* [Yocto Project Build Host Packages](https://docs.yoctoproject.org/brief-yoctoprojectqs/index.html)

## Setting up a Work Environment

These instructions assume that you are working in a project directory that we will refer to as **PROJECT_DIR**.

## Clone meta-ros

```
git clone -b build-next --single-branch  https://github.com/robwoolley/meta-ros.git build
```

Choose the MCF configuration
```
distro=ros2
ros_distro=humble
oe_release_series=honister
cfg=$distro-$ros_distro-$oe_release_series.mcf

mkdir conf
ln -snf ../conf build/.
cp build/files/$cfg conf/.
```

Create the build environment with the mcf tool
```
export MCF_META_ROS_REPO_URL="https://github.com/robwoolley/meta-ros.git"
export MCF_META_ROS_BRANCH="superflore/humble/2022-11-23"
export MCF_META_ROS_COMMIT="74ca1c51a6f27210143af7ca95ccf8c8cad033d6"
build/scripts/mcf -f conf/$cfg
```

```
source openembedded-core/oe-init-build-env
```

Edit conf/local.conf to include:
```
MACHINE = "qemux86-64"
INHERIT += "buildhistory"
BUILDHISTORY_COMMIT = "1"

INHERIT += "image-buildinfo"

INHERIT += "report-error"
```

Build ros-image-world
```
bitbake -k ros-image-world
```
