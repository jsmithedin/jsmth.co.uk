+++
title = "Chili Plant Monitor"
date =  "2015-06-09"
tags = [
    "Python", 
    "bash", 
    "linux", 
    "android", 
    "chili plants", 
    "CriticalBlue", 
    "Raspberry Pi"
    ]
+++

![Plants](https://lh3.googleusercontent.com/S5B8FSHEnQkgtH6iYLuzqvuCBgXpjvDSwJmQfxfEzHQ=w1324-h993-no)

After moving desks around the turn of the year into a new room with windows, a bit of a rarity at CriticalBlue, a colleague and I decided it was time to add some greenery to our, at this point, boring corner office.  The obvious thing to grow was a bunch of chili plants!  Amazon provided me with a pack of easy to grow Apache F1 seeds, some pots and a bag of compost.  After covering my desk, the floor and myself in soil I had some lifeless pots of soil.  Weeks passed and some growth was observed!  However, I couldn't help but wonder whether it was warm enough on my draughty window sill for life to flourish?  What happened at the weekend when the office was empty, did the heating go off dropping the temperature to unbearable levels?  It was February in Edinburgh after all.  The answer to these questions lay in an unused Raspberry Pi I had at home!

#The Setup

I brought in my Raspberry Pi, a DS18B20+ Temperature sensor and some bread board. You can see how I linked it all up below and visit [modmypi](https://www.modmypi.com/blog/ds18b20-one-wire-digital-temperature-sensor-and-the-raspberry-pi) for a step by step guide on how to get started.  A simple example of how to read the temperature in Python can be found [here](https://github.com/jsmithedin/chilimonitor/blob/master/temperature/rpi/tempnow.py).

![RPi](https://lh3.googleusercontent.com/O5aMKIybpbpghXxTyCUGCbcZ0he7XT-6J4EsRnsUklE=w745-h993-no)
#Data!

Now I could ask my sensor the temperature at any time and get a reading back, progress!  Asking it repeatedly myself was fine but I really wanted something to monitor stuff when I wasn't there, after all that's when it's going to be coldest.  As part of some other (actual) work I was doing I'd been playing about with the [Highcharts](http://www.highcharts.com/) graphing library.  Wouldn't it be great to graph the last 24 hours of temperature readings!  I could see highs and lows then decide if there were any trends worth acting on.

To do this I wrote a script which constantly runs on the Raspberry Pi.  It gets the temperature reading every 15 minutes and stores it in a tuple with the timestamp.  This is then added to a deque collection which stores the last days worth of readings.  From this I then write a bit of javascript which has all the data in the right format for highcharts.  This is scp'd over to my workstation which is running a web server and there we go!  This script is available on GitHub [here](https://github.com/jsmithedin/chilimonitor/blob/master/temperature/rpi/temperature.py).  

You can see an example of the chart produced below.  It turns out that the temperature in the office was fine, it never really dropped much below 18°c.  This didn't stop me wanting to extend the monitoring though!  

![Chart](https://lh3.googleusercontent.com/OIxeQN5JCw2iKSCKcqRAD1O4iiJio7nmdJVNEFZDY9g=w1904-h425-no)    
#Vision!

The obvious next step was to take pictures of the plants as they grew.  I scrounged an old Android phone which was hanging about in the office and set it up in the very professional looking holster you can see below.  I wrote a script which runs as a cron job every 10 minutes on my workstation which instructs the phone to take a picture using ADB then pulls it across to the host machine.  It then overlays the current time and temperature on the image and sends it to another machine to be displayed on our company intranet.  This script is [here](https://github.com/jsmithedin/chilimonitor/blob/master/images/camshot.sh). 
![Monitor](https://lh3.googleusercontent.com/G1v6qg_aEifkr_9ZAND6K_hmUISRx569QjtJAvIhvoE=w1324-h993-no)

# Timelapse

What do you do with a bunch of images taken over a period of time of a subject which is constantly changing?  You make a timelapse!  I amended the script from above to store every image taken between 12 and 1pm each day, hoping that's when there would be most light.  

Making a timelapse from a bunch of images in Linux is really easy.  Make sure your images are sequentially numbered from oldest to newest.

```
ls -tr | cat -n | while read n f; do mv "$f" "$n.jpeg"; done
```

Then allow ffmpeg to do its stuff to turn this into a video.  This video might play too quickly, you can use mencoder to fix this.  See Google, mine turned ok without tweaking.  
```
ffmpeg -i %d.jpg -r 25 -s 1280x960 -b:v 30000k Flapse_25fps.mp4
```

This produces a pretty big file, about 75MB for 600 images.  I turned this into a webm for display on internet.  
```
avconv -i Flapse_25fps.mp4 -acodec libvorbis -aq 5 -ac 2 -qmax 25 -threads 8 timelapse.webm
```

Here is the final result! 
[![Time Lapse](http://img.youtube.com/vi/5eADIF1MAfM/0.jpg)](http://youtu.be/5eADIF1MAfM)

# Future Work

There are a few tweaks I'd like to make to this project!  It would be more efficient to poll the temperature reading from the Raspberry Pi than to push it to the server.  To do this I'd like to put together a REST API using Flask which would run on the RPi.  I'd then use the API from a script on my server which would save the results in a MongoDB database then generate the javascript for the graph from there.  This could all run from the same script which takes the images for the timelapse.
