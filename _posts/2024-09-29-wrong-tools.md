---
title: 'On using wrong tools'
date: 2024-09-29
permalink: /posts/2024/09/on-using-wrong-tools/
tags:
  - reflection
  - SOAP API
  - webservice
  - WSDL
  - integration
  - R
  - Shiny
---

About two years ago I bought an AirTag. I usually go days without driving, so I end up not being sure about where my car is parked. The AirTag is a simple solution for this: I left it in my glovebox and now I can check my car's location, almost live, in my phone.

Of course, I soon started being curious about its inner workings. Very soon I found out your geolocation info is saved in a temporary JSON file in $HOME/Library/Caches/com.apple.findmy.fmipcore/Items.data, so I started doing some experimentation. After struggling for awhile, I ended up with a bash script that read my car's coordinates, which is not very useful per se. If I knew the distance from my house, I could do some more interesting stuff, like making a pop-up notification appear whenever my car starts moving (in case my partner uses it, for example). But transforming coordinates to distance requires trigonometry, and bash (as far as I know) does not have built-in trigonometric functions. So the next step was obvious: defining the sine and cosine functions as their Taylor series, with enough terms to make the results acceptable for my purpose.

In short, I ended up doing a lot more work than it would've been necessary had I used a better suited language, like Python. Why use bash, then? Well, a few reasons come to mind. First, it was fun. I had never done anything like that with bash, and it seemed like a fun idea. Second, it felt like a fun challange: how many of those functions that I usually take for granted could I define by mylsef? Would the results be any good? Would I be able to embark on the terrible journey of parsing a JSON file using regular expressions instead of, well, a JSON parser?

All of that script is now obsolete, since Apple decided to encrypt the Items.data file and I don't feel the need to research how to decrypt it. But it was a fun little project, and it was fun to use the wrong tool for a job and still getting the results I wanted. Sometimes I feel like too much focus is put on details and we end up forgetting that the end goal is to solve some problem, automate some task. Using bash was probably a terrible idea, but I had fun doing it and it felt good to make most of the work myself, and understand completely every bit of the code.

On a related note, I've spent a few weeks thinking about a problem in my spare time. It's a very specific problem, and I'll spare the minor details because they are frankly boring for those who have not encountered it before. In short, it goes like this:

Many of the messages sent over the internet rely on SOAP APIs, a specific type of API where information is exchanged in XML documents. XML documents need a specific formatting, and one needs to pay attention to some details, like namespaces and such. Since it would be too much trouble to type these XMLs by hand, a protocol was devised at the very beginning of the century to describe precisely what structure these documents should have: how many and what type of fields to fill, what namespaces to use, where to send it all. This protocol is known as WSDL (Web Services Description Language), and one finds that a WSDL document is, itself, a XML document. (This is becoming somewhat self-referential, right?). At this point, one would expect that anyone working with SOAP APIs would have a way of generating these documents without having to open a plain-text editor. And, of course, one would be wrong: it is entirely common for a modern integrator not to have a WSDL generator. 

Because of this, I decided to write a WSDL generator myself. The details, as I said, are incredibly dull. But it became a challenge, and I decided to not only write a generator, but a whole interface so that others could use it too. It is now live, and it can be found [here](https://malmriv.shinyapps.io/WSDLGenerator/). How is this related to what I was tellin before? The thing is, I wrote it in R. I know R is often considered a language reserved to those in academia, to data scientists, to economists and bioinformaticians. But, just like bash can be used to much more than it's often given credit for, R can be used (has been used) to deploy whole web-apps. In this case, I took a problem and cut through every minor sub-task until it was solved, and I did it in R, and at no point in that process the language itself was a barrier.

![](https://github.com/malmriv/malmriv.github.io/blob/master/_posts/images/wsdl-generator.png?raw=true)

Had I decided to use a more proper tool, doing both of these things would have felt like a task. Maybe I did not feel like honing my Python skills, and maybe in that case I would've chosen to go to the gym instead of spending a couple hours coming up with a few lines of code that solve a problem I encounted almost weekly. But I decided to use the "wrong" tool, one that I knew how to use already, and came up with a very good solution.

In the end, the best tool for the job isn't always the most obvious one —sometimes it's the one that keeps you curious and makes you enjoy the process.  Regardless of the language, what's often important is finding creative ways to solve problems and, of course, having a good time.
