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
that is a big video!

I think, though I didn't try, my browser would chug if I tried to pull that many images(despite many dupes) into one pageview

So I decided to instead do a video format. Gif format would also have probably been a bad idea, I just don't think most gif renderers are capable of handling the task of an 8kpx wide image.

Here's the nitty gritty of the technical:

- one script converts every banner in the archive to an entry in a csv. This csv has the following header structure: `headers = ['Image Name', 'Flashy', 'Mean Red', 'Mean Green', 'Mean Blue']`
	- What's flashy? basically, a good many of the banners are animated. Of those animations, I wanted to be able to filter out ones that had a high amount of statistical variation between frames. This comes out to calculation a standard devi