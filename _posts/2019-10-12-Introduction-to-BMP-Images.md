---
layout:     post
title:      Introduction to BMP images
date:       2019-10-12 08:00:00
summary:    Explaining how images are stored in bmp format.
categories: 
thumbnail: 
tags:
 - Bitmap
 - BMP
---
**Table of Contents**
* TOC
{:toc}

# Introduction #

## Reading image files ##
What is the best way to read an image file?
Should we read the whole file, store it and then extract information? but modern images are large, and it is redundant to have two copies of information. So we read it byte by byte as a stream. This works for bmp images because the data is arranged in the form of bytes.

To accomplish this, I've used methods from IO class of ruby. IO.sysopen('filepath') creates a file descriptor, and IO.new(filedescriptor) creates a stream from which we can read bytes.

Each byte represents a number between 0-255. We can read n number of bytes at once using sysread(n). To make sense of this, we use unpack() method in ruby and pass an argument specifying the format of storage (Little Endian or Big Endian if more than one byte to be read) and the number of bytes to unpack. For instance, we pass 'n' for 16 bit unsigned big endian representation.

BMP files are structured in the following way:
Bitmap info header - not optional - 14 bytes - contains basic information about the file
DIB header - not optional - generally 40 bytes - contains additional information such as number of colors, type of compression etc.
Color palette - if less than or equal to 8 bits/pixel
Pixel array - the actual data we are interested in - might contain pointers to the color palette / actual pixel data depending on number of bits per pixel

# BMP header #
  
## Code to read the header of BMP ##
```ruby
    file_descriptor = IO.sysopen(path)
    @raw_data = IO.new(file_descriptor)
    @signature = @raw_data.sysread(2)
    @size = @raw_data.sysread(4).unpack('L')[0] # Because little endian format
    @raw_data.sysseek(10)
    @offset = @raw_data.sysread(4).unpack('L')[0]
    @info_header_size = @raw_data.sysread(4).unpack('L')[0]
    @width = @raw_data.sysread(4).unpack('L')[0]
    @height = @raw_data.sysread(4).unpack('L')[0]
    @planes = @raw_data.sysread(2).unpack('S')[0]
    @bits_per_pixel = @raw_data.sysread(2).unpack('S')[0]
    @numcolors = 2**@bits_per_pixel
    @compression = @raw_data.sysread(4).unpack('N')[0]
    @compressed_size = @raw_data.sysread(4).unpack('L')[0]
    @x_pixels_per_m = @raw_data.sysread(4).unpack('L')[0]
    @y_pixels_per_m = @raw_data.sysread(4).unpack('L')[0]
    @colors_used = @raw_data.sysread(4).unpack('L')[0]
    @num_of_imp_colors = @raw_data.sysread(4).unpack('L')[0]
    read_to_palette if @bits_per_pixel <= 8
    read_masks if @bits_per_pixel >= 16
```
Here, @raw_data is the byte stream. First two bytes indicate signature, and should be equal to "BM" for a bitmaps. Next 4 bytes indicate size of the image and are stored in little endian format (unpack('L') converts reads little endian format). Next 4 bytes are reserved (and unused), so we move 4 bytes to position 10 in the stream by using sysseek(10). @offset refers to the position in the stream where the pixel by pixel image data starts (4 bytes long and Little endian format). 
Next 4 bytes indicate the size of infoheader, which is normally 40 bytes. @width and @height indicate width and height of the image in pixels. @planes indicates the number of planes present in the image - should be 1 for bmp. @bits_per_pixel tells us how many bits are needed to store data of one pixel, and based on this we read the pixel data. @numcolors refers to how many colors are supported, which is 2^n for n bit bmp. @compression refers to the type of compression used in storing bmp data, 0 refers to no compression, 1 refers to 8 bit Run Length Encoding(RLE8) and 2 refers to 4 bit Run Length Encoding (RLE4). @compressed_size tells us the size of bitmap after compression. @x_pixels_per_m and @y_pixels_per_m tell us about the pixel density per meter in x and y axis respectively. @colors_used tells us how many colors are used in the image. 
If @bits_per_pixel is <=8, the colors of image are stored in a palette and each pixel points to a color in the palette. So, while reading the image, read_to_palette method is called to store the palette entries. We will talk about masks at a later stage.
# BMP Palette #
## The code to read palette information ##
```ruby
  def read_to_palette
    @palette = []
    (1..@colors_used).each do |i|
      b = @raw_data.sysread(1).unpack('C')[0]
      g = @raw_data.sysread(1).unpack('C')[0]
      r = @raw_data.sysread(1).unpack('C')[0]
      temp = @raw_data.sysread(1).unpack('C')
      @palette[i - 1] = [b, g, r]
    end
  end
```
Pixel data is stored in B G R format in the images (B is blue, G is green and R is red).
Each scan line is zero padded to the nearest 4-byte boundary. If the image has a width that is not divisible by four, say, 21 bytes, there would be 3 bytes of padding at the end of every scan line.




