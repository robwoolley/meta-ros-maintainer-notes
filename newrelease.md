# Adding a new Yocto Release

## Milestone Release Modifications

On the meta-ros master branch, create 2 new commits, one to update the README.md and the other to update milestone.conf.

Update README.md to provide details on the new milestone.

Update milestone.conf to increment the milestone number and set the release date.

## Create New Branch

```
git checkout master
git branch <newrelease>-next
```

Update the README.md to specify the branch name.

# Master Branch Modifications

The first commit after the branch should modify all the layer.conf files to reflect the current release series.

```
meta-ros-common: ROS_OE_RELEASE_SERIES: switch from honister to kirkstone

* follow oe-core:
  https://git.openembedded.org/openembedded-core/commit/meta/conf/layer.conf?id=4a180aa5b30cc0906072d5b1e970eea41f1ce642

Signed-off-by: Rob Woolley <rob.woolley@windriver.com>
```
