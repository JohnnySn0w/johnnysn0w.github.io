---
layout: post
title: 88x31 Banners from the old Net
categories:
  - Projects
  - Movies
  - Media opinions
---


Yesterday I was scrolling my masto feed, when I spotted [a very interesting boost](https://kolektiva.social/@booters/112039018979714833).

Yes, the user posting it had gone through and scraped the Geocities backup for all of the 88x31pixel banners that were contained therein, and uploaded a sort of museum webpage, hosting all of them. They even did me the favor of posting a link to the de-duplicated zip file. Tiny at ~150MB, considering it holds almost 30,000 files.

That, in combination with a Youtube video I recently keyed onto, about the [creator's love of really obscure art](https://www.youtube.com/watch?v=d_n_gmINZTI), inspired me to try and do something, well, artsy! I too absolutely adore the way that some artists really dig into, trying to make art that is not only expressive, but is so on multiple levels, and isn't afraid to transgress socially acceptable aesthetics to make something that really evokes the kind of vibe they're going for. I mention in my About page, I love niche indies that speak to a specific person's life experience. I love art that is so very much about itself and nothing else.

Taking inspiration from maximalist ideas, and the general vibe of early internet aesthetics, I spent the entirety of yesterday—sans the runtime of The Beautiful Bones, which I watched with a friend—working on this project! that's 9am-8pmish. 

Being unemployed has perks, ahaha

So yeah! The project was almost fully formed but here's the gist:

![[Pasted image 20240306151532.png]]

Now, I later found that using small sites for cdn, is:

1. Known as hot linking, and is very frowned upon, because you're driving up costs for the hoster.
2. Not as much of a crime if you're using a corpo for it

And also, I eventually realized the sheer magnitude of the task, computationally.

This would not be something one could throw a web app together for, and expect to be able to dynamically render on a webpage. That was my first idea for the end product, you see.

No, even if I just run this process for one single image, say, one of the static 88x31 buttons, it takes up a bit of space in a number of ways, as seen in the stats for the final product:
![[Pasted image 20240306151927.png]]
that is a big video! (but actually just 9MB)

I think, though I didn't try, my browser would chug if I tried to pull that many images(despite many dupes, so not near as much RAM) into one pageview

Gif format would also have probably been a bad idea, I just don't think most gif renderers are capable of handling the task of an 8kpx wide image. So I decided to instead do a video format.

Here's the nitty gritty of the technical:

- one script, `gen_means.py`, converts every banner in the archive to an entry in a csv. This csv has the following header structure: `headers = ['Image Name', 'Flashy', 'Mean Red', 'Mean Green', 'Mean Blue']`
	- As stated before, I wanted to generate the mean of each banner, the RGB mean. This would later be useful in finding the pixel to banner mapping, using [euclidian distance](https://en.wikipedia.org/wiki/Euclidean_distance) for the RGB of the pixel to the mean RGB of the banner.
	- For Animated banners, I take the mean of each frame, and then generate a mean of those means for the overall image.
	- What's 'flashy'? basically, a good many of the banners are animated. Of those animations, I wanted to be able to filter out ones that had a high amount of statistical variation between frames. This comes out to calculating a standard deviation value, and then checking it against a semi-arbitrary(read: magic number I heuristically came to) of how big the stddev needed to be for a gif to be 'flashy'. 
	- Here's an example of a 'flashy' banner
		- ![[_hearts4_main_theq-fm.gif]]
	- The nice thing is, this script need only be run the one time, since this is a static dataset. Now, there is certainly room to consider that banners of this sizing exist and continue to be made, outside of the geocities archive. That could be considered Future Work.

One of the more interesting aspects of parsing all these files, was finding that some of them are zip bombs, in effect, and at least one, for actual. I watched my python process consume almost 10gigs of RAM trying to parse them when I set the protective `Image.MAX_IMAGE_PIXELS = 1000000000` value too high. Basically, the library I used to parse the images, Pillow, has a safeguard that pops a `DecompressionBombError` if it's beyond the maximum allowed pixels. I did make my max bigger than the default the library comes with, and at higher values than the current one, at least one gif is waybig.

I also had to set a truncated image value on, `ImageFile.LOAD_TRUNCATED_IMAGES = True`, as otherwise some of the banners wouldn't be parsed. This is an interesting side effect of the optimizations various programs would use in the era, to optimize for size. 