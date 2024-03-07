---
layout: post
title: 88x31 Banners from the old Net, an Art Project
categories:
  - Projects
  - Movies
  - Media opinions
---


Yesterday I was scrolling my masto feed, when I spotted [a very interesting boost](https://kolektiva.social/@booters/112039018979714833).

Yes, the user posting it had gone through and scraped the Geocities backup for all of the 88x31pixel banners that were contained therein, and uploaded a sort of museum webpage, hosting all of them. They even did me the favor of posting a link to the de-duplicated zip file. Tiny at ~150MB, considering it holds almost 30,000 files.

That, in combination with a Youtube video I recently keyed onto, about the [creator's love of really obscure art](https://www.youtube.com/watch?v=d_n_gmINZTI), inspired me to try and do something, well, artsy! I too absolutely adore the way that some artists really dig into, trying to make art that is not only expressive, but is so on multiple levels, and isn't afraid to transgress socially acceptable aesthetics to make something that really evokes the kind of vibe they're going for. I mention in my About page, I love niche indies that speak to a specific person's life experience. I love art that is so very much about itself and staying true to that.

Taking inspiration from maximalist ideas, and the general vibe of early internet aesthetics, I spent the entirety of yesterday—sans the runtime of The Beautiful Bones, which I watched with a friend—working on this project! that's 9am-8pmish. 

Being unemployed has perks, ahaha

I will be very clear, in that I was fairly heavily assisted by the Robot in this endeavor. Otherwise, I very much doubt I could have worked with these unfamiliar image libraries so quickly.

So yeah! The project was almost fully formed but here's the gist I sent to a friend:

![me gisting in discord](/assets/Pasted%20image%2020240306151532.png)

Let me be more concise though:
>For any given input image, convert each single pixel of that image into a RGB value, and swap it with a 88x31px banner whose RGB color mean is nearest that value.

Now, I later found that using others sites for CDN, is:

1. Known as [hot linking](https://en.wikipedia.org/wiki/Inline_linking), and is very frowned upon, because you're driving up costs for the hoster.
2. Not as much of a crime if you're using a corpo for it.

And also, I eventually realized the sheer magnitude of the task, computationally.

This would not be something one could throw a web app together for, and expect to be able to dynamically render on a webpage. That was my first idea for the end product, you see.

No, even if I just run this process for one single image, say, one of the static 88x31 buttons, it takes up a bit of space in a number of ways, as seen in the stats for the final product:
![final video dimensions](/assets/Pasted%20image%2020240306151927.png)
that is a big video! (but actually just 9MB)

I think, though I didn't try, my browser would chug if I tried to pull that many images(despite many dupes, so not near as much RAM) into one pageview

Gif format would also have probably been a bad idea, I just don't think most gif renderers are capable of handling the task of an 8kpx wide image. So I decided to instead do a video format.

Let's move to the nitty gritty of the technical.

## The Technical
One script, `gen_means.py`, converts every banner in the archive to an entry in a csv. This csv has the following header structure: `headers = ['Image Name', 'Flashy', 'Mean Red', 'Mean Green', 'Mean Blue']`

- As stated before, I wanted to generate the mean of each banner, the RGB mean. This would later be useful in finding the pixel to banner mapping, using [euclidian distance](https://en.wikipedia.org/wiki/Euclidean_distance) for the RGB of the pixel to the mean RGB of the banner.
	- For Animated banners, I take the mean of each frame, and then generate a mean of those means for the overall image.
	- What's 'flashy'? Basically, a good many of the banners are animated. Of those animations, I wanted to be able to filter out ones that had a high amount of statistical variation between frames. This comes out to calculating a standard deviation value, and then checking it against a semi-arbitrary(read: magic number I heuristically came to) of how big the stddev needed to be for a gif to be 'flashy'. 
	- Here's an example of a 'flashy' banner
	 ![a flashy banner](/assets/banner.gif)
	- The nice thing is, this script need only be run the one time, since this is a static dataset. Now, there is certainly room to consider that banners of this sizing exist and continue to be made, outside of the Geocities archive. That could be considered Future Work.

One of the more interesting aspects of parsing all these files, was finding that some of them are [zip bombs](https://en.wikipedia.org/wiki/Zip_bomb), in effect, and at least one, for actual. I watched my python process consume almost 10gigs of RAM trying to parse them when I set the protective `Image.MAX_IMAGE_PIXELS` value too high. Basically, the library I used to parse the images, [Pillow](https://pillow.readthedocs.io/en/stable/), has a safeguard that pops a `DecompressionBombError` if it's beyond the maximum allowed pixels. I did make my max bigger than the default the library comes with, and at higher values than the current one, at least one gif is waybig.

I also had to set a truncated image value on, `ImageFile.LOAD_TRUNCATED_IMAGES = True`, as otherwise some of the banners wouldn't be parsed. This is an interesting side effect of the optimizations various programs would use in the era, to optimize for size. Unfortunately, I have yet to find a solution to all of the various curveballs these optimizations are throwing me. Lots of out-of-bounds errors when copying individual frames for final video composition. I've been glancing at hex by dropping them into [cyberchef](https://gchq.github.io/CyberChef/). I can't say I've learned a lot, but it's been interesting to really see how messy some of these files are.

After the means are recorded to file, I then run another script, one that runs through an inputted image, and generates a mapping csv, one filename per pixel coordinate, in the form of `columns=['x', 'y', 'func_result']`, where `func_result` is the filename of the nearest(Euclidian) banner to the pixel. 

Finally, there's a third script that takes that csv, and performs the marriage of the images into frames, and from frames into video. This one breaks the most, and is where I'm left squashing the most bugs. As I said before, many of the issues come from optimizations in the banners. A browser like Firefox can handle these fine, as can some of my system image viewers, but Pillow simply isn't having it! I'll have to get more creative in how I'm parsing frames.

For the final script, the following technical details arise:
- If you're making a looped video, it should loop everything in a nice, clean way. This means the frames need to somehow line up all the different framerates of the gifs.
- I still need the dimensions of the image that was parsed as input, currently I don't have that information fed in dynamically. Semi-easy fix once the scripts are chained in a more formal manner.
- Working with small image inputs is best. I don't even know if the system as is could handle something even normal size, like 400x400.
### Updates
#### 7pm 3/6/24
- A fun error here, is with images just too large, it'll generate a canvas size that ffmpeg refuses to operate on. I didn't even realize there were technical limits on video size. ![](/assets/Pasted%20image%2020240306194035.png)
#### 9:30pm
- Have gotten it working completely. Had to set a section that creates downscaled images, basically, by dividing the max (8k) resolution by the banner sizes, I can get the max height and width of a base image to process. If an image is above either of these calc'd dimensions, I can generate a whole integer ratio to downscale it.
- Using the downscaled image, the max RAM usage is ~3gigs
- Going to call it for the night, but later I want to make the user interface smoother
#### 10pm
- i lied
- still here,
- trying to parallelize via multiprocessing. Had a memory error, looking into using shared memory bc it looked fine at the beginning, and I know I have the RAM for the base process.

## Final Results
The input image:
![](/assets/acraker2001_back_button.jpg)


The output video:
(This does not work on Firefox for some reason)
<video width="100%" height="auto" controls><source src="https://raw.githubusercontent.com/JohnnySn0w/8831-collager/main/output_video.mp4" type="video/mp4"/>Your browser does not support the video tag.</video>
### Closing thoughts
- Might finagle alpha channel back in. I just wanted to simplify things for the first go. There are some pretty heavily transparent banners, so they're certainly available for the palette.
- I half wanted to dive into computational shader development for this. I still might, that would go under Future Work though.
- This was really fun, I haven't really done generative art in a while.
- I came out of this realizing how much AI can help, but also how much it takes away in the feeling of the creative effort. For other projects, I will likely be using it more minimally. 

