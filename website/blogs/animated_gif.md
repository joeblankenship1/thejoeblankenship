# Moving Pictures

## Animated GIFs in Linux

**Posted by Joe Blankenship on October 20, 2018**

This is a quickie blog as I'm currently slammed with work. However, I should have a nice write-up from my Python Immersion Course in November. :)

## Making Pictures Move

I recently delivered a reveal.js-based presentation at the NACIS 2018 annual meeting. I love the simplicity of this presentation format, but it is not without some challenges. I have had issues with embedding large or complicated, interactive content from other websites and I fully expect this to be the case given the static nature of reveal.js static pages. So in order to maintain a minimalist approach to my slides, but to have some sort of activity, I decided to sample the content using animated gifs.

There are several programs that can generate animated gifs, but not many on Linux. This is where my journey began.

After a little searching, I found references to a library named `ffmpeg` which allows me to convert videos into animated gifs. This process was much simpler than I thought it would be.

## Generating Animated GIFs

First we have to make sure `ffmpeg` is installed on your system:

```bash

ffmpeg -version

```

You should see an output that states the current installed version. If not, then install `ffmpeg`:

```bash

sudo apt-get install -y ffmpeg

```

Once you’ve install `ffmpeg`, run the version check again to verify the install.

We are now ready to convert some videos. I use RecordMyDesktop to make videos which outputs an ogv file. Once you have a functional ogv file, we can convert it to an mp4 video file:

```bash

ffmpeg -i output.ogv -f mp4 output.mp4

```

Our input is the ogv file, the file type is mp4, and then we name our output file. We are now ready to generate an animated gif. However, before we do this, we have the ability to generate a HD gif by using a sampled color palette:

```bash

ffmpeg -y -i output.mp4 -vf fps=10,scale=800:-1:flags=lanczos,palettegen palette.png

```

We identify the mp4 file we just generated, define the frames per second, define the scale of the video frames, and pass information to generate the palette file in png format.

Once a palette image is generated, we are now ready to create a HD animated gif:

```bash

ffmpeg -i output.mp4 -i palette.png -filter_complex "fps=10,scale=800:-1:flags=lanczos[x];[x][1:v]paletteuse" output.gif

```

We once again call our mp4 file along with the palette file, define the formatting variables, and name our final animated gif.

In the end, you should end up with a gif that resembles your original video.

```{image} ../images/animated_gif_bitnodes.gif
:align: center
```
