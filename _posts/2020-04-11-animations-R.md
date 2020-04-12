---
title: Making animations in R.
date: 2020-04-11
permalink: /posts/2020/04/make-animations-with-R/
tags:
  - R
  - data visualization
  - scientific animation
  - fun
---

Animating the results of a simulation comes in handy very frequently, especially when one wants to check the evolution of a system. Even though R does not have native tools especially desiged to generate .gif files, it can be easily done by putting a few lines of code together. To illustrate this, I'll show the steps to create [a simple animation](https://twitter.com/ratanegrx/status/1247655608544878599), but keep in mind that the process can be modified to make more complex animations. (There's nothing a quick Google search won't give away when it comes to making an R plot look prettier). The full code for this example is available at the end of this post, but I want to explain every bit first to make it clear. Let's start.

1. **Have a clear idea of what you want to show.**
This isn't usually a problem, but starting from an overly complicated idea of what you want to get can make things unnecesarily harder. Aim for the bare minimum and for functionality instead of thinking about the glossy details first. In this case, I'll reproduce an idea I had a few days ago. Suppose we have N bodies labeled 1, 2, 3, (...) N, each one:
   - Following a simple harmonic motion.
   - With an amplitude proportional to their labeled value.
   - With an angular frequency proportional to their labeled value.
   
  A possible mathematical description of this system could be:

 <img align="center" src="https://latex.codecogs.com/gif.latex?\dpi{120}&space;y_k(t)&space;=&space;k&space;\sin&space;\big{(}&space;2\pi&space;k&space;t&space;\big{)}" title="y_k(t) = k \sin \big{(} 2\pi k t \big{)}" />

So the least we'll need will be a time array.

2. **Define the data structures you'll be reading for the animation.**
This is trivial when all you have to do is plotting data from a simulation, since the results are usually available in some kind of readable file. In this case, I'll generate the "data" we are going to plot. Initially, we need a time array, so let's define it as a sequence of length 700. Seven hundred seems like an arbitrary number, but you'll see I have set it like that for a simple reason: I want that to be the number of frames to generate. In R, this is done with a single line:
```r
time = seq(0,1,len=700)
```
Which is all we need for now.

3. **Make sure the files will be saved somewhere safe.**
I know a few .png files don't seem dangerous, but you don't want to accidentally generate seven hundred files in your Desktop. So, to create a folder in your working directory, just type:
```r
dir.create("[INSERT DIRECTORY HERE]")
```

4. **Define a loop to automate the creation of frames.**
This can be easily done with a `for` command ranging from 1 to the number of desired frames. But it sometimes happens that we would end up with a few thousand frames, and we don't want that. So, in order to "sample" the evolution of the system every few interations, it can be handy to do this:

```r
for(i in 1:700) {
      if(i%%2 == 0) {
        (...)
    }
}
```
The `if` statement checks wether the current value of `i` is a multiple of 2, and only lets *something* happen in case it is. (We'll see what that something is now). For a system with many known states, getting a wieldy amount of frames is straightforward: just change the divisor in the modulo (`%%`) function.

5. **Create a blank .png file.**
(And don't forget to close it when you're done, which can be done with `dev.off()`). The main difficulty in this step is creating an adequate name for each file. Luckily, there exist a function to concatenate strings and numbers together: the `paste` function. It works by concatenating its arguments and returning a string. The arguments to concatenate can be something we write, a variable containing a string, or a number. The option `sep=""` allows us to choose the separation character, which in this case would be nothing. Therefore, something like:
```r
png(paste("[SAME DIRECTORY YOU CREATED EARLIER]/",i,".png",sep=""),width=550,height=550)
```

Would create a 550 x 500 image named `i.png` (where `i` is the number corresponding to the current iteration) stored in the folder we previously created. It is important to close the file after we're done adding things to it, so don't forget to add `dev.off()`. 

6. **Define the actual plot.**
We need nothing more than an empty plot, but it's still necessary to define things like axis limits, axis labels, title, legend or whatever we might need. How do we achieve an empty plot? We simple choose to plot the point at (0,0) (or any other point, for that matter) selecting `type = "n"`, which results in `n`othing being plotted. The limits are going to be the same for each axis: from -16 to +16, since I want to work with the numbers 1 to 15 and I still want a little margin to make sure nothing goes off-screen. Therefore: 
```r 
plot(0,0,type="n",xlim=c(0,16),ylim=c(-16,16))
```
If you need any help with any command, you can always type `?plot` to see what things you can control. (You can change pretty much every aspect of a plot in R, so don't be too worried about being limited here).

7. **Add whatever you need to this plot.**
R has many functions to add elements to a plot. You can add arrows, points, segments, lines, text and most simple shapes. You can combine them in any way you like, and something as simple as a point can be changed in colour, size and shape. In this case, I'll be adding points like this:

```r
for(j in 1:15) {
        lines(j,j*sin(2*pi*j*time[i]),col=rainbow(15)[j],type="p",cex=4,pch=19)
        text(j,j*sin(2*pi*j*time[i]),labels=j,col="white")
      }
```

Let's break it down. The `for` loop goes from 1 to 15 because those are the "bodies" I want to plot. The `lines` command allows me to add elements to an existing plot, but they don't need to be lines. I specify I want to add something in the position where the body should be according to the formula I came up with in the first section. Using the `type` command, I can decie wether I want to add lines, points or any other thing. Since I selected `type = "p"` (points), I can change the shape of the points with `pch`, which is the less intuitive command I've ever met. Checking any online table, `19` seems to plot full circles. That could work. The `cex` command changes the size of the point in terms of how many times bigger than the standard size we want it to be. The `col=rainbow(15)[j]` thing allows me to set a colour for each point. Something like `col="red"` would have made every point red, but I'm *this* extra and decided to make every point a different colour. `rainbow(15)` returns colour array with 15 different colours to choose from, ordered as in a rainbow. To select one in those 15 colours, we specify which one we want by typing `[j]` next to it, being `j` the current iteration. The second line in the loop adds a text corresponding to the number of the body.


And that's it! The entirety of the code turns out to be: 
```r
time = seq(0,1,len=700)
dir.create("folder")
for(i in 1:700) {
      if(i%%100 == 0) {
      png(paste("folder/",i,".png",sep=""),width=550,height=550)
      par(bg="black",col.lab="black",col.main="black",mar=c(0,0,0,0))
      plot(0,0,type="n",xlim=c(0,16),ylim=c(-16,16))
      for(j in 1:15) {
        lines(j,j*sin(2*pi*j*time[i]),col=rainbow(15)[j],type="p",cex=4,pch=19)
        text(j,j*sin(2*pi*j*time[i]),labels=j,col="white")
      }
      dev.off()
    }
}
```

After executing it, a folder will appear and the desired number of frames will start popping up:

![](https://github.com/malmriv/malmriv.github.io/blob/master/_posts/images/create-folder.png?raw=true)



This is often sufficient to check the evolution of the system, but if we wish to actually have a finished .gif image we can merge them all together. It's easy enough to do with free software like Gimp, but the whole process can be condensed in a single line with programs like [ImageMagick](https://imagemagick.org/index.php). One can just open a terminal, navigate to the folder where the frames are stored and type something like:
```
convert %d.png[1-700] animation.gif
```
Where `file%d.png` refers to files following the sequence file1.png, file2.png, (...) and `[1-700]` refers to the interval of images to turn into an .gif file. The resulting animation, as I said before, can be seen [here](https://twitter.com/ratanegrx/status/1247655608544878599) (I don't think it'd be very polite to make you load a 7 Mb .gif file). That's pretty much it. The code, as you can see, is not very difficult to handle. All it takes is some familiarity with R.

## Troubleshooting.

*Q. I can't create any new .png files all of a sudden. Why?*
A. If the `for` loop that creates new files is interrupted in the middle of an iteration, R will have trouble deciding which file to write onto the next time it's called to do so. If this happens, we need to close every open file. A quick way of doing so is executing `dev.off()` a few times, but if lots of files have remained open one can just close a lot of them at the same time with a loop:
```r
for(i in 1:1000) dev.off()
```
Eventually we will get a notice indicating that there are no more files left to close: `Error in dev.off() : cannot shut down device 1 (the null device)`.
