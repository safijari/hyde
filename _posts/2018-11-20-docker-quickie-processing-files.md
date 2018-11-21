---
title: "Docker Quickie: Extracting Laser Data From Bag Files without Installing ROS"
layout: post
---

So I had some bag files lying around that contained some laser data. 
I only needed the first scan to run a calibration routine on a system that did not
have ROS installed (and I didn't want to install ROS). I could have done the processing
in one of my VMs which I use for robot development but instead I decided try using
a Docker container to do this processing in place. It worked really well so I'm sharing
how to do this!

## The Python script to do the actual processing

{% highlight python %}
from glob import glob
import rosbag
import yaml
import json
import os
fnames = glob('/folder/*.bag')

print fnames

bags = [rosbag.Bag(f) for f in fnames]

for fname, bag in zip(fnames, bags):
    print "processing {}".format(fname)
    for topic, msg, t in bag.read_messages("/scan"):
        with open(os.path.splitext(fname)[0] + ".json", 'w') as ff:
            ff.write(json.dumps(yaml.load(str(msg))))
        break
{% endhighlight %}

The above script looks at the `/folder` directory to find all bag files. It then
opens each bag file, reads the first laser scan, outputs and outputs it to a `json`
with the same name as the bag file.

## The Docker command

The above script will work on files inside the `/folder` directory on a system that has
rosbag available (though you wouldn't normally have files in a folder on root like that).
The next step is to run this script inside a docker container that has ROS installed.

```
docker run --rm -v /home/jari/bags_directory/:/folder -v /home/jari/script_directory/:/script ros:kinetic-robot python /script/modify_bag_files.py
```

Here's a breakdown of that command:
- `docker run --rm` will run the container but will also do cleanup after it exits
- `-v /home/jari/bags_directory/:/folder` mounts the local folder `bags_directory` at the `/folder` path in the container!
- `-v /home/jari/script_directory/:/script` mounts the local folder containing the above python script under `/script` in the container
- `ros:kinetic-robot python /script/modify_bag_files.py` the first part runs the `kinetic-robot` container under the `ros` namespace from `dockerhub` (see all possible options [here](https://hub.docker.com/_/ros/)) and runs the python command inside it.

## A note on file ownership

Most docker containers don't define a user inside the environment. 
As such everything runs as root. There are [excellent reasons](https://medium.com/@mccode/processes-in-containers-should-not-run-as-root-2feae3f0df3b) for always setting
up a user before using images from `dockerhub` or other public sources but
when using containers from official sources I worry about this much less.
If you do something similar to the process I describe, however, the files output
to the mounted folder would be owned by root, and you will likely need to do
a `chown` on them.

## "What else is possible?" and other parting thoughts

This is a pretty standard pattern for using docker. One other example similar to this
that I've used before is building `jekyll` websites. If you don't have the patience
for getting the `ruby` dependencies right. You can also run servers that serve data
from your filesystem, databases that write to your filesystem rather than the container's
ephemeral filesyste, and much much more.
