---
layout:     post
title:      Reading BMP images into Ruby array
date:       2019-10-13 08:00:00
summary:    Explains how pixel data of different types of bitmaps is read
categories: 
thumbnail: 
tags:
 - 1 bit bmp
 - 8 bit bmp
 - 24 bit bmp
---
**Table of Contents**
* TOC
{:toc}

# Introduction #
In the previous post, we have seen how the header of BMP is read, and how the color palette is read. In this post we will see how the actual pixel data of different types of bmp images is stored into ruby array.
We use this method to read the pixel values into a Ruby array:
```ruby
def read_to_array
    @raw_data.sysseek(@offset)
    a = Array.new(@height)

    if @bits_per_pixel <= 8
      a = read_from_palette
    elsif @bits_per_pixel > 8
      a = read_from_pixel_array
      # puts "Location of pointer after reading: #{@raw_data.pos}"
    end

    a
  end
```
In the earlier post, we had seen that the actual pixel data begins at @offset from the beginning of the stream. So, we use sysseek(@offset) to reach the starting of pixel data. We start reading the data from here.

# BMP with 1 bit per pixel #
Since @bits_per_pixel<8, each pixel value is a pointer to an entry in the color palette. Note that 1 bit per pixel doesnt mean a black and white image or a greyscale image, it just means there are only two colors in this image (could be yellow and blue or cyan and magenta or whatever you want!). The color palette has two entries.
Below is a part of the method read_from_palette which we call in the read_to_array method described earlier.
```ruby
  
    if @bits_per_pixel == 1
      row_length = ((@width / 32).floor * 32 + 32) / 8
      (1..@height).each do |i|
        colors = []
        row_length_data = ''
        (1..row_length).each do
          row_length_data += @raw_data.sysread(1).unpack('C')[0].to_s
        end
        (1..@width).each do |j|
          colors[j - 1] = @palette[row_length_data[j].to_i]
        end
        a[@height - i] = colors
      end
```
First of all, here we are calculating the row length, in terms of multiple of 4 bytes (remember how a 21 byte row in an image was padded with zeros to make the width 24). For example, consider an image with width 124 pixels. Since 1 bit per pixel, it means one row of pixels has 120 bits of data, ie 15 bytes. It is padded with zeros to make the row length a multiple of 4 bytes. (120/32).floor = 3, (3*32+32)/8 = 16. So, the row is 16 bytes long.
Now, we need to extract data bit by bit to save to the array. Here, it is done by converting the entire row into a string of ones and zeros, and reading element by element @width times and storing into array. 

# BMP with 8 bits per pixel #
```ruby
    elsif @bits_per_pixel == 8
      if @compression != 0
        a = decompress_RLE8
        return a
      end
      (1..@height).each do |i|
        row_array = Array.new(@width)
        (1..@width).each do |j|
          row_array[j - 1] = @palette[@raw_data.sysread(1).unpack('C')[0]]
        end
        @raw_data.sysread((@width * 3) % 4) # *Note*
        a[@height - i] = row_array
      end
    end
```
Reading a 8 bit image into an array is quite straightforward. Since 8 bit bmp can be compressed using RLE8, we first check for compression. We will talk about compressed images in a later post.
If the image is not compressed, we begin reading. We know that pixel data is stored in rows, and there are @height number of rows. Each row is padded to the nearest multiple of 4 bytes. We read @width number of bytes for each row and append to the row_array, then account for any padding in the line marked *Note*. Then we append the row_array to the original array for storing pixel data (a). 



# BMP with 24 bits per pixel #
Method given below is an illustration of how 24 bit bmp is read.
```ruby
    if @bits_per_pixel == 24
      (1..@height).each do |i|
        row_array = Array.new(@width)
        (1..@width).each do |j|
          b = @raw_data.sysread(1).unpack('C')[0]
          g = @raw_data.sysread(1).unpack('C')[0]
          r = @raw_data.sysread(1).unpack('C')[0]
          row_array[j - 1] = [b, g, r]
        end
        @raw_data.sysread((@width * 3) % 4) # *Note*
        a[@height - i] = row_array
      end
    end
```
When @bits_per_pixel is more than 8, it is rather expensive to store the colors in a palette. So, they are directly stored in the pixel data. The pixel data is stored as @height number of rows each containing data for @width number of pixels along with padding. The color data is stored in the order B G R, and each pixel has three bytes, one for each color. We read @width\*3 number of bytes for each row and append to the row_array, then account for any padding in the line marked *Note*. Then we append the row_array to the original array for storing pixel data (a). 

#More examples#
  
