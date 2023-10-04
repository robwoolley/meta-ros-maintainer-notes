# Adding a new ROS Distro

### Copy rolling to new release

```
cp -ra meta-ros2-rolling meta-ros2-humble
cd meta-ros2-humble
rm -rf generated-recipes/ files/rolling/generated/ conf/ros-distro/include/rolling/generated/
git add .
mv ./files/rolling ./files/humble
git mv ./conf/ros-distro/include/rolling ./conf/ros-distro/include/humble
git mv ./classes/ros_distro_rolling.bbclass ./classes/ros_distro_humble.bbclass
git mv ./recipes-core/packagegroups/packagegroup-ros-world-rolling.bb ./recipes-core/packagegroups/packagegroup-ros-world-humble.bb
```
Update rolling -> humble in various places:
```
vi classes/ros_distro_humble.bbclass \
      conf/layer.conf \
      conf/ros-distro/include/humble/ros-distro.inc \
      conf/ros-distro/include/humble/ros-distro-preferred-providers.inc \
      conf/ros-distro/include/humble/ros-distro-preferred-versions.inc \
      recipes-core/packagegroups/packagegroup-ros-world-humble.bb
```

Update ROS_DISTRO_CODENAME and ROS_DISTRO_BASELINE_PLATFORM in ros-distro.inc

Commit the changes, starting in meta-ros2-humble
```
git add .
git commit -sv
```

Using this commit message:
```
{humble} initial version from meta-ros2-rolling without generated-recipes

meta-ros$ cp -ra meta-ros2-rolling meta-ros2-humble
meta-ros$ cd meta-ros2-humble
meta-ros2-humble$ rm -rf generated-recipes/ files/rolling/generated/ conf/ros-distro/include/rolling/generated/
meta-ros2-humble$ git add .
meta-ros2-humble$ mv ./files/rolling ./files/humble
meta-ros2-humble$ git mv ./conf/ros-distro/include/rolling ./conf/ros-distro/include/humble
meta-ros2-humble$ git mv ./classes/ros_distro_rolling.bbclass ./classes/ros_distro_humble.bbclass
meta-ros2-humble$ git mv ./recipes-core/packagegroups/packagegroup-ros-world-rolling.bb ./recipes-core/packagegroups/packagegroup-ros-world-humble.bb

Update rolling -> humble in various places:
meta-ros2-humble$ vi classes/ros_distro_humble.bbclass \
  conf/layer.conf \
  conf/ros-distro/include/humble/ros-distro.inc \
  conf/ros-distro/include/humble/ros-distro-preferred-providers.inc \
  conf/ros-distro/include/humble/ros-distro-preferred-versions.inc \
  recipes-core/packagegroups/packagegroup-ros-world-humble.bb

Update ROS_DISTRO_CODENAME and ROS_DISTRO_BASELINE_PLATFORM in ros-distro.inc

Signed-off-by: Rob Woolley <rob.woolley@windriver.com>
```


### Reference
Example commit of rolling to galactic from Martin:
```
commit 83712dd94e2ce49d260e5e73a7e20923fea82cd7
Author: Martin Jansa <martin.jansa@lge.com>
Date:   Mon May 24 06:37:32 2021 -0700

    {galactic} initial version from meta-ros2-rolling without generated-recipes

    meta-ros$ cp -ra meta-ros2-rolling meta-ros2-galactic
    meta-ros$ cd meta-ros2-galactic
    meta-ros2-galactic$ rm -rf generated-recipes/ files/rolling/generated/ conf/ros-distro/include/rolling/generated/
    meta-ros2-galactic$ git add .
    meta-ros2-galactic$ git mv ./files/rolling ./files/galactic
    meta-ros2-galactic$ git mv ./conf/ros-distro/include/rolling ./conf/ros-distro/include/galactic
    meta-ros2-galactic$ git mv ./classes/ros_distro_rolling.bbclass ./classes/ros_distro_galactic.bbclass
    meta-ros2-galactic$ git mv ./recipes-core/packagegroups/packagegroup-ros-world-rolling.bb ./recipes-core/packagegroups/packagegroup-ros-world-galactic.bb

    Update rolling -> galactic in various places:
    meta-ros2-galactic$ vi classes/ros_distro_galactic.bbclass \
      conf/layer.conf \
      conf/ros-distro/include/galactic/ros-distro.inc \
      conf/ros-distro/include/galactic/ros-distro-preferred-providers.inc \
      conf/ros-distro/include/galactic/ros-distro-preferred-versions.inc \
      recipes-core/packagegroups/packagegroup-ros-world-galactic.bb

    Signed-off-by: Martin Jansa <martin.jansa@lge.com>
```

## Modify ros-generate-* scripts

Modify ros-generate-cache.sh and ros-generate-recipes.sh to increment the minor number in SCRIPT_VERSION and extend the ROS_DISTRO case statement to include the new release.

eg.

```
commit b1af5155fc0ccc2adad5c7d4559cabdb62b8705f
Author: Martin Jansa <martin.jansa@lge.com>
Date:   Mon May 24 06:55:04 2021 -0700

    ros-generate-{cache,recipes}: add support for galactic (v1.7.0)

    Signed-off-by: Martin Jansa <martin.jansa@lge.com>
```

### Generate the cache

The format of the ros-generate-cache.sh executable is:

```
sh scripts/ros-generate-cache.sh <ROS_DISTRO> <ROS_DISTRO_RELEASE_DATE>  <ROS_ROSDISTRO_CHECKOUT_PATH> <ROS_ROSDISTRO_COMMIT> [BRANCH_NAME]
```

Execute ros-generate-cache.sh using the environment variables set previously.
```
cd meta-ros
sh scripts/ros-generate-cache.sh ${ROS_DISTRO} ${ROS_DISTRO_RELEASE_DATE}  ${ROS_ROSDISTRO_CHECKOUT_PATH} ${ROS_ROSDISTRO_COMMIT}
```

Example commit message:

```
commit c55fc1c3cce258308fdc2db40c97c1e7c94b52bb
Author: Martin Jansa <martin.jansa@lge.com>
Date:   Mon May 24 06:58:59 2021 -0700

    {galactic} Update cache.yaml and rosdep files for galactic release 2021-05-23 as of 20210523200442

    https://discourse.ros.org/t/ros-2-galactic-geochelone-released/20559

    Signed-off-by: Martin Jansa <martin.jansa@lge.com>
```

## Generate the recipes
Execute ros-generate-recipes.sh to create the recipes for the ROS_DISTRO.
```
sh scripts/ros-generate-recipes.sh ${ROS_DISTRO}
```

Example commit message:
```
commit bc30978597fe0cade4d1cb4de3c4d87e28455e70
Author: Martin Jansa <martin.jansa@lge.com>
Date:   Mon May 24 07:07:21 2021 -0700

    {galactic} Sync to files/galactic/generated/cache.yaml as of 20210523200442

    Recipes generated by superflore for all packages in ROS distribution galactic.

    This pull request was generated by running the following command:
```

## Add support to packagegroup-ros-world

```
commit d35bc32c1ba52ce1f73b588214d4efd53372dccf
Author: Martin Jansa <martin.jansa@lge.com>
Date:   Mon May 24 07:19:17 2021 -0700

    common: packagegroup-ros-world: add support for ROS2 galactic

    Signed-off-by: Martin Jansa <martin.jansa@lge.com>
```

## Rename bbappends

```
commit f9787970e34d417dc305e4dcc4565235067ebe40
Author: Martin Jansa <martin.jansa@lge.com>
Date:   Mon May 24 07:25:20 2021 -0700

    {galactic} rename bbappends to match new versions

    Signed-off-by: Martin Jansa <martin.jansa@lge.com>
```

## Add an mcf file

Copy an existing mcf file to create a new configuration.
```
cd meta-ros
git checkout build
cd files
cp ros2-galactic-honister.mcf ros2-humble-honister.mcf
```

Modify the new file to set the variables (like ROS_Distribution) appropriately.  If necessary set the MetaRos_* variables as well.

```
files/*-galactic-{dunfell,gatesgarth,hardknott,honister}.mcf: Add initial version
Signed-off-by: Rob Woolley <rob.woolley@windriver.com>
```