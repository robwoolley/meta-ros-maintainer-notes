# Testing meta-ros

## Pre-requisites
* Ubuntu 22.04.1 LTS (Jammy Jellyfish)
* XXX GB of storage space
* [Yocto Project Build Host Packages](https://docs.yoctoproject.org/brief-yoctoprojectqs/index.html)

## Setting up a Work Environment

The following steps are based on the wiki: [OpenEmbedded Build Instructions · ros/meta-ros Wiki (github.com)](https://github.com/ros/meta-ros/wiki/OpenEmbedded-Build-Instructions)

### Build Environment Setup
1. All packages listed on the wiki installed except socat, python3-pexpect, python3-jinja2
   ```
   sudo apt-get install gawk wget git diffstat unzip gcc-multilib \
        build-essential chrpath cpio python3-pip \
        xz-utils debianutils iputils-ping \
        python3-git libegl1-mesa libsdl1.2-dev xterm \
        g++-multilib locales lsb-release python3-distutils time \
        liblz4-tool zstd file
   sudo locale-gen en_US.utf8
   ```

2. Same as the wiki except with simplified comments:

   ```
   mkdir $NEW_PROJECT_DIR
   cd $NEW_PROJECT_DIR
   git clone -b build --single-branch https://github.com/ros/meta-ros build
   mkdir conf
   ln -snf ../conf build/.

   # Example 1
   # distro=ros1
   # ros_distro=noetic
   # oe_release_series=honister

   # Example 2
   # distro=ros2
   # ros_distro=galactic
   # oe_release_series=hardknott

   distro=ros2
   ros_distro=galactic
   oe_release_series=dunfell

   cfg=$distro-$ros_distro-$oe_release_series.mcf
   cp build/files/$cfg conf/.

   # Optional override the default meta-ros with your own repo, branch, and commit
   # export MCF_META_ROS_REPO_URL=https://github.com/kuta42/meta-ros
   # export MCF_META_ROS_BRANCH=dunfell-next
   # export MCF_META_ROS_commit=HEAD

   # use the HEAD of MCF_META_ROS_BRANCH, not "HEAD".
   # export MCF_META_ROS_COMMIT="" 

   # Clone the OpenEmbedded metadata layers and generate conf/bblayers.conf .
   build/scripts/mcf -f conf/$cfg

   # Set up the shell environment for this build and create a conf/local.conf . We expect all of the variables below to be unset.
   unset BDIR BITBAKEDIR BUILDDIR OECORELAYERCONF OECORELOCALCONF OECORENOTESCONF OEROOT TEMPLATECONF
   source openembedded-core/oe-init-build-env

   # The current directory is now the build directory; return to the original.
   cd -

   # An OpenEmbedded build produces a number of types of build artifacts, some of which can be shared between builds for
   # different OpenEmbedded DISTRO-s and ROS distros. Create a common artifacts directory on the separate disk under which all
   # the build artifacts will be placed. The edits to conf/local.conf done below will set TMPDIR to be a subdirectory of it.

   mkdir -p /path/to/my/artifacts
   ```

3. Add the text to the bottom of conf/local.conf

   ```
   # Set ROS_COMMON_ARTIFACTS to:
   ROS_COMMON_ARTIFACTS ?= "/path/to/my/artifacts"

   # Set MACHINE to one of the machines defined in the MCF configuration
   # To see a list of supported machines do:
   #    $ grep -A1 "Supported MACHINE" build/files/$cfg
   #
   #    # Supported MACHINE-s
   #    Machines = ['qemux86', 'raspberrypi4']
   ```

# Build Multiple Configurations

Herb provided a useful script called [meta-ros-mcf-build-multiple.bash](https://gist.github.com/kuta42/63dcf1bf8097a5eb522728ea9c295c9c).  This script can be used to cycle through supported configurations to automate build combinations at scale.

1. Create a new build environment.
   ```
   mkdir meta-ros-multi-dir
   cd meta-ros-multi-dir
   git clone -b build --single-branch https://github.com/ros/meta-ros build
   mkdir conf
   ln -snf ../conf build/.
   ```
2. Download meta-ros-mcf-build-multiple.bash from Herb's GitHub gist
   ```
   curl "https://gist.githubusercontent.com/kuta42/63dcf1bf8097a5eb522728ea9c295c9c/raw/275006ac52fc711ef72f87726a43450ebdaa2c17/meta-ros-mcf-build-multiple.bash" -o build/scripts/meta-ros-mcf-build-multiple.bash
   chmod +x build/scripts/meta-ros-mcf-build-multiple.bash
   ```

3. You may optionally uncomment and modify lines 243 to 252 in the script to specify your own fork. For example, Herb's meta-ros for Milestone 17 was:
   ```
    # declare branch=$(python3 -c "exec(open('conf/$cfg').read()); print(OE_Branch)")
    # declare -x MCF_META_ROS_REPO_URL=https://github.com/kuta42/meta-ros.git
    # declare -x MCF_META_ROS_BRANCH=${branch}-next
    # declare -x MCF_META_ROS_COMMIT=""
   ```

4. You may wish to remove configuration files for any combinations that you wish to skip.  For example, ROS Rolling is under development so you may not want to include it in regression builds.
   ```
   rm build/files/*rolling*
   ```

5. If you wish to keep your new artifacts separate from the existing ones, modify line 76 to set ROS_COMMON_ARTIFACTS to a new artifact directory.
   ```
   ROS_COMMON_ARTIFACTS ?= "/path/to/my-new-artifacts
   mkdir -p /path/to/my-new-artifacts
   ```
6. Run meta-ros-mcf-build-multiple.bash using tee to save the build output and still print to the standard output.

   ```
   bash build/scripts/meta-ros-mcf-build-multiple.bash | tee -a build.log
   ```
# Runtime Tests

There are some basic tests that can be executed at runtime listed on the wiki: [Image Sanity Testing](https://github.com/ros/meta-ros/wiki/OpenEmbedded-Build-Instructions#image-sanity-testing)

The ROS 2 tests are duplicated below:
```
# ping ros.org
# source ros_setup.sh                 # Skip if built with "ros-implicit-workspace" in IMAGE_FEATURES.
# echo $LD_LIBRARY_PATH               # If set, all the paths must exist.
# ros2 topic list
/parameter_events
/rosout

# ros2 msg list                       # dashing
# ros2 interface list -m              # eloquent and later
...

# (sleep 5; ros2 topic pub /chatter std_msgs/String "data: Hello world") &
# ros2 topic echo /chatter
publisher: beginning loop
publishing #1: std_msgs.msg.String(data='Hello world')

data: Hello world

publishing #2: std_msgs.msg.String(data='Hello world')

data: Hello world```
```

Other tests we might consider adding include:
* timer_lambda
* publisher_lambda / subscriber_lambda