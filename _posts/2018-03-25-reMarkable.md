---
title: reMarkably Hacky - academic paper reader
---

*TL;DR: bought an e-reader. It didnt do what I wanted so i made [this](https://github.com/lschwetlick/maxio/tree/master/tools) and [this](https://github.com/lschwetlick/rMsync). Now it's not as bad anymore.*

For a while now I've been keeping an eye out for an e-reader suitable for reading academic papers. 
Mainly I like the idea of reducing the chaos of loose sheets of paper floating around of my desk.
It just turns out most e-readers are (understandably) much more suited to e-book formats than pdfs. 

So I was looking for the following features:
- Large enough to read regular pdfs without needing to zoom in (peferably A4 size)
- E-ink screen, so I can read outside and reduce time spent staring at a screen
- Has a pen, so I can mark the text and take notes in the margins
- Allows me to get the PDF with the annotations back out of the device with OCR and stuff intact

So, long story short, I bought a [reMarkable](https://www.remarkable.com). Out of the box it falls tragically short of my expectations but after a bit of tinkering it fulfills my criteria. I mean I guess we could get into a whole story about how that time could have been spent organizing my printed papers and writing my thesis, not to mention used to money to buy an iPad and Apple pencil, but lets roll with it.

![]({{site.blog_url}}/resources/images/blog2/rM.jpg){:height="250px" class="shadow center", }

**Size**: The screen diagonal is 10.3". This ist just *barely* large enough for me to read papers comfortably. The resizing options on the device are not good.

**E-ink**: Check. It's not brilliant or particularly snappy, but good enough for my use case

**Pen**: The drawing and writing is clearly what the developers spent all their time on. It's surprisingly fast and exact, except maybe at the very edges.

**Handling Files**: This is the part that made me seriously sad. And then made me feel a strong urge to fix it.
The app is just terrible. 
The intended use is to connect the device to a wifi and sync with their provided cloud service. The app also connects to that service and lets you upload and view files. It also lets you export files to pdf and png (or so I was informed by their customer support before buying).

![]({{site.blog_url}}/resources/images/blog2/rMSupport.png){:height="250px" class="shadow"}

Spoiler Alert: it doesn't. At least not to any reasonable standard. The PDF export usually misses some of the pages, creates a file that far exceeds any reasonable expectations in filesize, and has no OCR (i.e. the text of the exported pdf can be neither searched nor marked). 

The PNG export, while at least exporting all the pages, creates images of random image sizes, so if you merge them into a pdf if might look something like this:

![]({{site.blog_url}}/resources/images/blog2/pngsizes.png){:height="500px" class="shadow"}

All of this quite apart from the fact that I have no interest in storing my data on this companies cloud.

At this point I was half-conviced I would send it back and buy a small iPad with a pen (price point is not all that different). But I do like the e-ink screen and the fact that there are fewer distractions,  so here is the system I came up with to turn the reMarkable into an academic paper reading device.

## How I made it workable

So the reMarkable was missing two critical features for my purposes.

1. A way to export pdfs that keeps the OCR and the notes
2. A way to sync the reMarkable with my folder of papers on my computer without using the rM cloud

The reMarkable kindly allows an alternative access point to the app. If you plug in into your computer you can `ssh` into it from the commandline using the credentials given on the about page.

```
ssh root@10.11.99.1
```

...Or just copy the entire directory over to be able to play with it locally:

```
scp root@10.11.99.1:/home/root/.local/share/remarkable/xochitl /directory/of/your/choice
```

Now I'm not going to repeat all the things that make the reMarkable tick, there's plenty of that in the [reMarkable Wiki](http://remarkablewiki.com/) and on the [reHackable Github](https://github.com/reHackable) and on other blogs like [this one](http://blog.lucafluri.ch/2017-11-21/customizing-remarkableTablet).

Suffice it to say, that the reMarkable saves annotation information in their custom format: .lines files, and never actually touches the pdf (in the course of this project I've been told that pdf if actually a way overloaded, terribly complicated file format that only very few people want to mess with and that thats why we can't just add a layer of annotations). 

In the "xochitl" (great name btw) folder on the reMarkable you will find each pdf file, conveniently renamed with a hash, a file of meta information and a .lines file.

![]({{site.blog_url}}/resources/images/blog2/rMFilestructure.png) 

### PDF export

Now, [reHackable](https://github.com/reHackable) have already decoded the .lines format and written a function that can turn .lines files into svgs. This means you end up with all the marks you made with the pen in an svg (seperate from the original pdf).So this is super useful for quick drawings and sketches but not yet so helpful for the annotation problem.

[Someone else](https://github.com/phil777/maxio) had already had the idea that to use the python version of the library Cairo to draw the contents of the .lines file (using the reHackable's decoding) onto a blank pdf of the correct dimensions.

The idea is that this annotation pdf can then be "stamped" onto the original using PDftk,

```
pdftk input.pdf multistamp annot.pdf output final.pdf 
```

leaving you with the original OCR'd file with the annotations just pasted over the top.

The original code that I found in some git issue thread surprisingly worked pretty okay just as it was!
I branched  it, fixed some issues with it and added some funtionality, but the [rm2pdf](https://github.com/lschwetlick/maxio/tree/master/tools) function works pretty well now . Cairo doesnt do dynamical pen width of line structure so well, but I haven't found that too bothersome for highlighting and annotating text.
For drawings and stuff I would recommend the [rm2pdf](https://github.com/lschwetlick/maxio/tree/master/tools) function though.

I had originally hoped to be able to use real pdf annotations, that I could edit on the computer as well. This doesnt seem particularly realistic right now, not least because the remarkable can't actually display standard pdf annotations. 

### Syncing

Now that I had figured out that I can export annotated files, all that was left was to write a script I can start when I plug the reMarkable in, to sync the contents with a predefined directory on my computer  [(Find it here)](https://github.com/lschwetlick/rMsync). 

I went with python because I much prefer it over shell scripts, but I have since been informed that using os.system("whatever terminal command") is deprecated and frowned upon. It does do exactly what I want though, so... sorry I guess.

First thing the script does is pull a full device backup from the remarkable. This isn't strictly necessary, but I dont mind having it sitting around. 
The remotely accessing the reMarkable requires a password. I tried for a long time with various hacks to get python to provide the password without user input. I gave up. This is clearly not a thing the terminal wants you to do. 
"os.system" kindly waits for user input at the password stage (as opposed to the newer/not deprecated "subprocess", which just keeps running the script without waiting). So now I just type it in or use [these instructions](http://remarkablewiki.com/tech/ssh) to verify your computer.

Next, the script iterates over the files, extracting their real, non-hashed name, and if they have an associated lines file (i.e. they have been annotated) it uses the above method to export annotated pdfs to a reference folder.
Notebooks (just drawings or notes that are independent of pdfs) are also exported, but using the reHackable svg script and then a pdf conversion, because their strokes just look a bit nicer than the Cairo ones.

Lastly, the script checks if there are any files in the local directory that are not on the reMarkable. It then uses the endpoint described in the [wiki](http://remarkablewiki.com/tips/client) to copy those over and convert them to reMarkable readable hashed files. 
Eventually I'd like to do this in a custom way as well, because this way, they don't arrive on the reMarkable sorted into folders (if someone has done this already, I'd love to hear about it!).


### Paper and Citation Management
I would like to add that this whole endevour also included figuring out the logistics of how to keep an ordered record of the papers I need.
What I've found to solve this for me is JabRef. JabRef is not a pdf organizer. It saves records of literature meta data and links to files (like a pdf) in BibTex format (it can also read existing libraries) but provides a pretty good interface to browse and sort.
Critically, it doesn't blow up if I replace a file with another of the same name (e.g. when syncing new annotated files over).

To be honest, it also gets a point in my book for not attempting to be a social media website and a cloud service, but just quietly goes about it's business doing a good citation managing job.
Fair warning though, it's developed and maintained by nice people as a side project and it does sometimes crash (never anyhing that a program restart doesn't cure).

### Extra

I made a sleeve. The 75€ for the original sleeve seemed kind of steep and its super easy to do. One large piece of felt and a thick rubber band gave me this:
![]({{site.blog_url}}/resources/images/blog2/sleeve.jpg){:height="500px" class="shadow"}

### Extra 2
I wanted a backup pen, because my posessions have an upsetting habit of being where I am not. 
[This one](https://www.amazon.de/gp/product/B06Y3F5W87/ref=oh_aui_detailpage_o03_s00?ie=UTF8&psc=1) feels bad in your hand, is made of cheap plastic and is way too small. But it does work in a pinch and,at 4,50€ a piece, is dirt cheap.