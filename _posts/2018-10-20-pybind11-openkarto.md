---
title: A Detailed Study of Wrapping OpenKarto with PyBind11
layout: post
published: False
---

In this post I break down in detail the various challenges of wrapping a `C++` library using
`pybind11` (you can find the result **pyOpenKarto** [here](https://github.com/safijari/pyOpenKarto) to get a pleasant python interface. It contains examples of how to do various things which
I had to dig through the documentation to find, so I'm hoping it can help out a weary traveler or two.

## Some background on OpenKarto

As the _about_ portion of this website shows, I'm at least part roboticist despite most of my 
time at [Simbe Robotics](www.simberobotics.com) being spent on computer vision and machine learning. As such I've had a few tussles with 
algorithms to map an environment using laser scanner data. Most recently I encountered 
(and quite enjoyed working with) [`OpenKarto`](https://github.com/ros-perception/open_karto) which 
was originally developed at SRI and currently maintained by [OSRF](https://www.openrobotics.org/).

## The need for a python wrapper and a solution

In addition to using this library onboard the robot using the excellent 
[`slam_karto`](http://wiki.ros.org/slam_karto) package, I realized we could make use of it to post
process some logged laser data and improve the overall accuracy of our data pipeline.

Small snag: our data pipeline is written in `python`. Now, this isn't _technically_ an issue, what with

1. `python` being designed with [C/C++ extensions](https://docs.python.org/2/extending/extending.html) in mind,
2. `cython` allowing you to [call C++ functions](https://cython.readthedocs.io/en/latest/src/userguide/wrapping_CPlusPlus.html),
3. `Boost.Python` allowing what always appears to be a [simple way to interface](https://www.boost.org/doc/libs/1_68_0/libs/python/doc/html/index.html) `python` with `C++` (if you can get it to work...)
4. and `pybind11` allowing for a very `Boost.Python` like interface but no `boost` requirement ... also it's header only!!!

Number 4 was brand new to me. I had tried (and mostly struck out, repeatedly) with all the other options. So I gave it a try. I started with the excellent [`python_example`](https://github.com/pybind/python_example) since that allows me to build/deploy with the `setup.py` file.

## Dealing with external dependencies

This is the first thing I needed to do a bit of digging to find as the example does not tackle it.
Building OpenKarto needs at least `libeigen3-dev` and `libboost-thread-dev` to function correctly (darn, 
I couldn't avoid boost entirely). The former is header only, the latter is dynamically linked 
(note that this means that `libboost-thread` must be present on the target system, I have yet to find a way
to bundle it with the resulting python package).

All of this needs to be specified in the `Extension` module for my library as shown here:

{% highlight python %}
ext_modules = [
    Extension(
        'openkarto',      # python module name
        ['src/PythonInterface.cpp', 
        'src/Mapper.cpp', 
        'src/Karto.cpp'], # All the cpp files that need to be built
        include_dirs=[
            'include',    # OpenKarto's necessary .h files
            '/usr/include/eigen3/', # Eigen 3's headers
            get_pybind_include(),
            get_pybind_include(user=True)
        ],
        libraries=['boost_thread'], # Tell the compiler to find libboost_thread
        library_dirs=['/usr/lib/'], # Unnecessary in this specific case
        language='c++'
    ),
]
{% endhighlight %}

As you can see the overall changes are minimal and pretty straight forward. Note that for most correctly
installed libraries on linux (`libboost-thread` included) you should not need to specify a library dir
as that is already searched.

## The interface file
