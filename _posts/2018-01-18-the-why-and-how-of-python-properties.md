---
title: The Why and How of Python Properties
layout: post
---

**Note: **This is another dirty repost from my `Medium` account. I seem to get no visibility there and I hate how you share code in it. Since I'm back on the blog, figured I'd repost. Enjoy :) 

#### **A Real World Problem**

About a year ago we hit an interesting roadblock at my current job: our computer vision pipeline was consuming too much memory. Machines with 10 gigs of available RAM plus a sizeable swap partition were thrashing hard. After some investigation we realized that this was a result of us loading all the relevant input images for a job into memory at the start of the pipeline. This was not an issue when the pipeline was first put together where the largest number of images per job was about 20 but that number suddenly started to grow. We discovered this issue on a job that had 210 images which, at 52 megs an image in memory, turned into 11 gigs of ram use.

Sooooooo yeah. We had a problem. The simple answer to this dilemma was to only load the image into memory when it was needed and not hanging onto unnecessary references to it. This is a decent solution (though it has problems of its own which are solvable but I won’t get into that in this article) but one that seemingly required a large amount of refactoring. To understand why, allow me to throw some (very simplified) example code at you:
    
   
{% highlight python %}
# The class that holds onto the image
class ImageHolder:
    def __init__(self, image_path):
        self.image = load_image_from_path(image_path)


holders = []
# The bit of code that loads the images
for image_path in large_list_of_paths:
    holders.append(ImageHolder(image_path))


# Various parts of the pipeline that do something with the images
for holder in holders:
    do_something(holder.image)
{% endhighlight %}

#### **A Real World Solution**

The most straightforward solution to this in most other languages (e.g. C++) is this:
    
    
{% highlight python %}
class ImageHolder:
    def __init__(self, image_path):
        self.image_path = image_path


    def get_image(self):
        return load_image_from_path(self.image_path)
{% endhighlight %}

But this creates a problem: we now have to make sure all references to `ImageHolder.image` are changed to `ImageHolder.get_image()`. We also need to ensure that programmers remember to change future code as well (though some would argue that we should have done this from the start and I would agree if properties didn’t exist). While neither of these are particularly monumental tasks, they are still annoyances and they mess with the semantics of the code itself (i.e. `operate_on(holder.image)` reads significantly better than `operate_on(holder.get_image())` ). Thankfully, in python we can instead just do this:
    
    
{% highlight python %}
class ImageHolder:
    def __init__(self, image_path):
        self.image_path = image_path


    @property
    def image(self):
        return load_image_from_path(self.image_path)
{% endhighlight %}

And now `ImageHolder.image` (which doesn’t _look _like a function call) returns the image, just as if it were a field pointing to an image. **This **is the magic of python properties. They rescued us from an initial bad design (not making getters and setters from the get go) while keeping our code from looking clunky.

#### **Another, less “Real Worldish” Example**

There are of course a myriad of other reasons to use abstractions like this and it can really help keep the total size of the _state_ in an object small and improve maintainability. Consider the naive implementation of a `Circle` class:
    
    
{% highlight python %}
PI = 3.141
class Circle:
    def init(self, radius):
        self.radius = radius
        self.area = PI * self.radius ** 2
        self.circumference = 2 * PI * self.radius
{% endhighlight %}

And use it in the following way:
    
    
{% highlight python %}
c = Circle(1)
print c.area  # prints 3.141
print c.circumference  # prints 6.282


c.radius = 2
print c.area  # prints 3.141 which is incorrect
print c.circumference  # prints 6.282 which is incorrect
{% endhighlight %}

Once we change the `radius`, the `area` and `circumference` fields become useless until updated (we can use setters here to alleviate this situation but that’s another article). Compare to the following implementation:
    
    
{% highlight python %}
PI = 3.141
class Circle:
    def init(self, radius):
        self.radius = radius


    def area(self):
        return PI * self.radius ** 2


    def circumference(self):    
        return 2 * PI * self.radius
{% endhighlight %}

Running the same tests on it again, we get:
    
    
{% highlight python %}
c = Circle(1)
print c.area  # prints 3.141
print c.circumference  # prints 6.282


c.radius = 2
print c.area  # prints 12.564 which is correct!
print c.circumference  # prints 12.564 which is correct!
{% endhighlight %}

It only makes sense for only`radius` to be the _state _of this object because both other values can be derived from it. Python properties allow us to encapsulate this reality without making the syntax look clunky.

#### **Conclusion**

I wanted to show the usefulness of Python properties in a real world context. I hope this article does that. A more detailed discussion of properties (as well as setters) can be found [here](https://www.programiz.com/python-programming/property). I will talk setters in another article in which I’ll explain how to update an image that isn’t loaded into memory (among other things).
