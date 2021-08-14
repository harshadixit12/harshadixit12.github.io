---
layout:     post
title:      Overview of Image processing library
date:       2020-02-08 14:00:00
summary:    Explains how the library is structured, and why it has been structured so.
categories:
thumbnail:
tags:
 - Image processing
---
**Table of Contents**
* TOC
{:toc}

# Introduction #
It has been a while since I last published something, I have been busy reading books, and exploring
image processing libraries in other platforms, namely opencv in python.

To start with, I believe certain features are necessary for an image processing library -
| Index  | Feature                                     |
| -----  |:-------------------------------------------:|
|  1     | Reading and writing images (bmp, png, jpeg) |
|  2     | Transform RGBA to RGB, HSV, grayscale etc   |
|  3     | Blur, erosion, dilation, kernel transforms  |

If you notice, there are two separate types of functionality here.
First, we need an Image class, which should encapsulate the properties of images amd should provide high level methods to read and write them. The high level methods should hide the complexity in implementation, so we
need classes with methods to read and write different formats of images which will be called from methods in Image class - lets call these image format classes. Second, we need a class which
offers image processing functionality.
This is an argument for having Image processing to be defined in a separate class from image class, and that in turn should be separate
from classes dealing with image formats.

So, we have bmp.rb which contains the bmp class, image.rb defined Image class, imgp.rb
contains Imgproc class with static methods for image processing - these accept and return Image object.
So far, we have discussed about reading of bmp images (that means I have written the code for it),
I am still understanding and coding writing of bmp images. Rest of the features will be added eventually.

Since we have already discussed about bmp images, in this post, I will elaborate on Image and Imgproc classes.

# 1. Image class #
When you hear image, what properties can you think of?
Surely image should contain pixels, so we have an array of pixels named data as a member.
Surely every image has height and width, so we have those as members
We have another member called channels, which tells us how many channels the image contains -
it could be RGB, or RGBA where A refers to alpha channel used for transparency, or it could be a
single channel - for grayscale image.
Since the image object has members which are interdependent, we must not allow them to be modified
by methods which do not belong to the class, so all of these have only attribute readers.

Image class also has two methods, read and write, which wrap over the methods from classes of different
image formats.


```ruby
class Image
  attr_reader :data, :height, :width, :channels

  def initialize(path)
    @data = read(path)
  end

  def read(path)
    is_bmp = 1
    if(is_bmp)
      bmp_instance = Bmp.new(path)
      a = bmp_instance.read_to_array
      @height = bmp_instance.height
      @width = bmp_instance.width
      @channels = a[0][0].length
    end
    a
  end

  def write(path)
    is_bmp = true
    if(is_bmp)
      a = Bmp.write_to_bmp(path,@data,8,0)
    end
  end

end
```

# 2. Imgproc Class #

I am a beginner myself when it comes to image processing, however I have studied digital signal processing as an elective in my Uni, so I might just be able to understand certain mathematical aspects of it. Lets talk about that later, for now, to process images, what functionalities would we want? For one, would want to convert images into Grayscale, we could extend this to Hue Saturation and Value as well (HSV). Secondly, we would want to be able to scale, rotate and blur the images. Third, we would want to erode and dilate images - more on this later.

Here is a high level overview of Imgproc class, I will discuss each of the methods in later posts.

```ruby
class Imgproc

  # Converts RGB image to grayscale
  def self.bgr_to_grayscale(img)

  end

  # Converts RGB to Hue Saturation and Value
  def self.bgr_to_hsv(img)

  end
  #utility function for bgr_to_hsv
  def self.bgr_to_hsv_util(array)

  end
  #function to blur the image
  def self.blur(img, blurtype)

  end

  def self.dilate(img)

  end

  def self.erode(img)

  end

  def self.scale(img)

  end

  def self.rotate(img)

  end

  def self.kernel_transform(img, kernel)

  end

  # To remove alpha channel
  def self.alpha_removal(img)

  end

end
```
