---
layout:     post
title:      Reading run length compressed BMP images
date:       2019-10-14 08:00:00
summary:    Explains how to read RLE8 and RLE4 compressed images
categories: ['Technology']
thumbnail: 
tags:
 - 4 bit bmp
 - 8 bit bmp
 - RLE
---
**Table of Contents**
* TOC
{:toc}

# Introduction #
Quite often, pixels near to each other have the same color, and it is redundant to store the same color information so many times. For instance, lets say 10 pixels in a row have a color x, rather than saving [x][x][x][x][x][x][x][x][x][x], we could just save [10,x] and save a lot of memory. This is the idea behind run length encoding. Although it doesnt achieve the same compression ratio as other compression methods, it is quick to execute and easy to understand.
In bmp images, two types of run length encoding can be used, RLE8 and RLE4, used with 8 bits per pixel and 4 bits per pixel images.

When a bmp image is compressed using RLE, it can happen that RLE is used for some pixels and no compression is used for the others. While writing code for compression and decompression, we should keep this in mind. Lets call reading the non compressed pixels absolute mode. 
The RLE mode is a simple RLE mechanism, the first value contains the count, the second value the pixel to be repeated. If the count value
is 0, the second value is special, like EOL (end of line) or delta.
In absolute mode, the second value contains the number of value to be copied litteraly. Each absolute run must be word-aligned ( We need to take care to ensure this, else the read values do not make sense) that means you may have to add an aditional padding byte which is not included in the count. After an absolute run, RLE compression continues.

# 8 bit Run Length Encoding #
In 8 bit RLE, each value is a byte. In RLE mode, the first byte contains the count, the second byte the pixel to be repeated. If the count byte is 0, the second byte is special, like EOL (end of line) or delta.

In absolute mode, the second byte contains the number of bytes to be copied litteraly. Each absolute run must be word-aligned ( We need to take care to ensure this, else the read values do not make sense) that means you may have to add an aditional padding byte which is not included in the count. After an absolute run, RLE compression continues.

| Second byte    | Meaning                                |
| -------------  |:--------------------------------------:|
|  0             | End of line                            |
|  1             | End of bitmap                          |
|  2             | Delta - horizontal and vertical offset |
|  3-255         | Switch to absolute mode                |






```ruby
  def decompress_RLE8
    a = Array.new(@height)
    @raw_data.sysseek(@offset)
    (1..@height).each do |j|
      i = 0
      row_array = []
      len = 0
      loop do

        first_byte = @raw_data.sysread(1).unpack('C')[0]
        second_byte = @raw_data.sysread(1).unpack('C')[0]
        if first_byte != 0
          (1..first_byte).each do
            row_array.push(@palette[second_byte])
            len += 1
          end
        elsif second_byte > 2
          sz = (second_byte+1) & (~1)

          if second_byte %2 ==1
            (1..(sz-1)).each do
              len += 1
              pixel_val =  @raw_data.sysread(1).unpack('C')[0]
              row_array.push(@palette[pixel_val])
            end
            garbage = @raw_data.sysread(1).unpack('C')[0]
          else
            (1..sz).each do
              len += 1
              pixel_val =  @raw_data.sysread(1).unpack('C')[0]
              row_array.push(@palette[pixel_val])
            end
          end
        elsif second_byte == 1
          #break
        elsif second_byte == 2
          puts "Error"
        end
        break if len >= @width

      end
      a[@height - j] = row_array
    end
    puts @raw_data.pos.to_s
    return a
  end
```

# 4 bit Run Length Encoding #

RLE4 compression has the same two modes of the RLE_8 compression, absolute and RLE. In the RLE mode, the first byte contains the count
of pixels to draw, the second byte contains in its two nibbles (4 bits each) the indices of the pixel colors, the higher 4 bits are the left pixel, the lower 4 bits are the right pixel. Note that two-color runs are possible to encode with RLE_4 through this. Absolute mode is similar to RLE8.

```ruby
  def decompress_RLE4
    a = Array.new(height)
    (1..@height).each do |j|
      i = 0
      row_array = []
      len = 0
      loop do

        first_byte = @raw_data.sysread(1).unpack('C')[0]
        second_byte = @raw_data.sysread(1).unpack('C')[0]
        if first_byte != 0
          pixel_two = second_byte & 15
          pixel_one = second_byte >> 4
          rolling_array = [pixel_one, pixel_two]
          k = 0
          (1..first_byte).each do
            row_array.push(@palette[rolling_array[k]])
            k = k ^ 1
            len += 1
          end
        elsif second_byte > 2
          sz = (((second_byte + 1)>>1) + 1) & (~1)
          if second_byte%2 == 1
            temp = 0
            (1..sz).each do
              len += 2
              two_pixels =  @raw_data.sysread(1).unpack('C')[0]
              pixel_two = two_pixels & 15
              pixel_one = two_pixels >> 4
              if temp<second_byte
                row_array.push(@palette[pixel_one])
                temp += 1
              end
              if temp<second_byte
                row_array.push(@palette[pixel_two])
                temp += 1
              end
            end
          else
            (1..sz).each do
              len += 2
              two_pixels =  @raw_data.sysread(1).unpack('C')[0]
              pixel_two = two_pixels & 15
              pixel_one = two_pixels >> 4
              row_array.push(@palette[pixel_one])
              row_array.push(@palette[pixel_two])
            end
          end
        elsif second_byte == 1
          break
        end
        break if len >= @width

      end
      a[@height - j] = row_array
    end
    puts @raw_data.pos.to_s
    return a
  end
```

