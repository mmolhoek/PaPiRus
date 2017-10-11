# papirus

ruby gem to talk to the [PaPiRus](https://www.pi-supply.com/?s=papirus&post_type=product&tags=1&limit=5&ixwps=1) display

before you start playing make sure you got the edp-fuse setup

## epaper fuse driver installation instructions (if you have not done that already ::)
```bash
sudo apt-get install libfuse-dev -y

git clone https://github.com/repaper/gratis.git
cd gratis
make rpi EPD_IO=epd_io.h PANEL_VERSION='V231_G2'
make rpi-install EPD_IO=epd_io.h PANEL_VERSION='V231_G2'
systemctl enable epd-fuse.service
systemctl start epd-fuse
```

You can find more detailed instructions [https://github.com/repaper/gratis](here)

## gem installation

```bash
$ gem install papirus
```
## usage

```ruby
require 'papirus'

# first we get ourself a display
display = PaPiRus::Display.new()
```

# Playing with Chunky_PNG

First install chunky_png

```bash
$ (OSX) brew install chunky_png
$ (debian/ubuntu) sudo apt-get install chunky_png
$ (Windows) no idea (did not use windows for 20 year, yes that is possible)
$ gem install chunky_png
```

Then, start an irb session to play around
```ruby
irb
require 'papirus'
require 'papirus/chunky' # add's to_bit_stream function to chucky

#lets get a clean png of the size of the display to play with using chunky_png
image = ChunkyPNG::Image.new(display.width, display.height, ChunkyPNG::Color::WHITE)
#and we draw a circle on it which is about the size of the screen
image.circle(display.width/2, display.height/2, display.height/2-2)
```

have a look at [chunky_png](https://github.com/wvanbergen/chunky_png/wiki) for more examples

```ruby
#and last we dump the image as bitsteam to the display
display.show(image.to_bit_stream)

# now we could also change the circle and fast update the screen
image.replace!(ChunkyPNG::Image.new(display.width, display.height, ChunkyPNG::Color::WHITE))
image.circle(display.width/2, display.height/2, display.height/4)
display.show(image.to_bit_stream, 'F')

# or update the screen for multiple circles
display.clear
2.step(image.height/2-2, 5).each do |radius|
    image.replace!(ChunkyPNG::Image.new(display.width, display.height, ChunkyPNG::Color::WHITE))
    image.circle(display.width/2, display.height/2, radius)
    display.show(image.to_bit_stream, 'F')
end
```

## there are multiple screen commands ['F', 'P', 'U', 'C']

Full update (with screen cleaning):

```display.show(image.to_bit_stream); display.update``` or

```display.show(image.to_bit_stream, 'U')```

Fast update:

```display.show(image.to_bit_stream); display.fast_update```
```display.show(image.to_bit_stream, 'F')```

Partial update:

```display.show(image.to_bit_stream); display.partial_update``` or

```display.show(image.to_bit_stream, 'P')```

## Load an image from a png file with convert and chunky

First, let's use Image Magick's `convert` tool to convert any image into a b/w image the way the diplay likes it
```bash
convert in.jpg -resize '264x176' -gravity center -extent '264x176' -colorspace gray  -colors 2 -type bilevel out.png
```

Where
* the -resize scales the image to fit the display
* The -gravity and -extent combination (order is important!) makes sure the image stays at the size of the display and in the centre
* The -colorspace -colors -type combi makes the image a 1-bit grayscale b/w image

Then we use chucky with our extension to show

```ruby
irb
require 'papirus'
require 'papirus/chunky'
display = PaPiRus::Display.new()
image = ChunkyPNG::Image.from_file('out.png')
display.show(image.to_bit_stream(true))
```
result:
![that's me](https://raw.githubusercontent.com/mmolhoek/papirus/master/example_output.jpg)

# Playing with RMagic (not working yet)

First install rmagick

```bash
$ # install native Image Magick library
$ (OSX) brew install imagemagick@6 && brew link imagemagick@6 --force
$ (debian/ubuntu) sudo apt-get install imagemagick
$ (Windows) no idea (did not use windows for 20 year, yes that is possible)
$ # install the gem that talks to the native Image Magick library
$ gem install rmagick
```

Then, start an irb session to play around
```ruby
require 'papirus'
require 'papirus/rmagick'

display = PaPiRus::Display.new()
img = Magick::Image::read('/path/to/img/file.(png|jpg|etc)'.first
img.to_papirus(display)
```

## Testing without a PaPiRus display

If you want to test the gem, but don't have your PaPiRus available, you can do the following

* clone this repo
* run the createtestepd.sh script that is in the repo which creates the needed files and folders in /tmp/epd
* start irb
* require 'papirus'
* display = PaPiRus::Display.new(epd_path: '/tmp/epd')
* play with the examples above
* when you run `display.show` the **fake** display /tmp/epd/LE/display is filled with your image
* now you can use a bin editor like xxd to have a look at the result: `xxd -b /tmp/epd/LE/display`

## TODO

* make the image.to_bit_stream routine faster (as it is now to slow to do animations with partial updates)
* add support for reading the temperature of the display
* add support for changing the update rate
* make load png image with chunky_png work (now output is black)
* make a display.load(image) that takes multiple formats and figures out how to present them
* create an issue to add your own requests :)

## Other resources

* [pi supply python driver](https://github.com/PiSupply/PaPiRus)
