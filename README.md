# Maintaining meta-ros

## Pre-requisites
* Ubuntu 22.04.1 LTS (Jammy Jellyfish)
* XXX GB of storage space
* [Yocto Project Build Host Packages](https://docs.yoctoproject.org/brief-yoctoprojectqs/index.html)
* Additional Packages
   * python3.10-venv
   * git

## Setting up a Work Environment

These instructions assume that you are working in a project directory that we will refer to as **PROJECT_DIR**.

## Install Superflore

The installation instructions for Super Flore can be found here: https://github.com/ros-infrastructure/superflore#installation

The instructions below have been adapted to use a temporary fork of meta-ros that includes support for the Humble Hawksbill release.

```
cd ${PROJECT_DIR}
git clone https://github.com/ros-infrastructure/superflore.git
cd superflore
python3 -m venv venv
source venv/bin/activate
python3 ./setup.py install
```

## Superflore OE Recipe Generation Scheme

Details on the OpenEmbedded recipe generation scheme using Super Flore can be found here: https://github.com/ros/meta-ros/wiki/Superflore-OE-Recipe-Generation-Scheme

### Installation

Create a new directory to store the source list for the rosdep tool. It is important to export the ROSDEP_SOURCE_PATH so we can avoid using "sudo" in the following steps.
```
cd ${PROJECT_DIR}
export ROSDEP_SOURCE_PATH=$PWD/rosdep
mkdir ${ROSDEP_SOURCE_PATH}
```

From inside the Python venv, run the *rosdep init* command to initialize the ROSDEP_SOURCE_PATH. (ie ${PROJECT_DIR}/superflore/venv//bin/rosdep init)
```
$ rosdep init
Wrote /opt/projects/milestone18/rosdep/20-default.list
Recommended: please run

        rosdep update
```
Then run *rosdep update* to download the latest manifests from GitHub.
```
$ rosdep update
reading in sources list data from /opt/projects/milestone18/rosdep
Hit https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/osx-homebrew.yaml
Hit https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/base.yaml
Hit https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/python.yaml
Hit https://raw.githubusercontent.com/ros/rosdistro/master/rosdep/ruby.yaml
Hit https://raw.githubusercontent.com/ros/rosdistro/master/releases/fuerte.yaml
Query rosdistro index https://raw.githubusercontent.com/ros/rosdistro/master/index-v4.yaml
Skip end-of-life distro "ardent"
Skip end-of-life distro "bouncy"
Skip end-of-life distro "crystal"
Skip end-of-life distro "dashing"
Skip end-of-life distro "eloquent"
Add distro "foxy"
Add distro "galactic"
Skip end-of-life distro "groovy"
Add distro "humble"
Skip end-of-life distro "hydro"
Skip end-of-life distro "indigo"
Skip end-of-life distro "jade"
Skip end-of-life distro "kinetic"
Skip end-of-life distro "lunar"
Add distro "melodic"
Add distro "noetic"
Add distro "rolling"
updated cache in /home/rwoolley/.ros/rosdep/sources.cache
```

After the rosdep update is complete, we shall retrieve a copy of the rosdistro repository.

```
cd ${PROJECT_DIR}
git clone https://github.com/ros/rosdistro.git
ROSDISTRO_URL="https://raw.githubusercontent.com/ros/rosdistro/master/rosdep"
sed -i -e "s|${ROSDISTRO_URL}|file://${PROJECT_DIR}/rosdistro/rosdep|" ${ROSDEP_SOURCE_PATH}/20-default.list
```


### Clone the meta-ros repository
```
cd ${PROJECT_DIR}
git clone -b honister https://github.com/ros/meta-ros
```