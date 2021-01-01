---
layout: post
title: Frame Time Analysis
---

Ever play a game that feels chunky but the frame rate counter looks fine? It takes a single frame coming out a fraction of a second too late to get me to notice, but most frame rate monitors squash outlier frames to simplify output. But these dropped frames are probably the most important of them all:
![1.png]({{ site.baseurl }}/images/2021-01-01-frame_time_analysis/1.png)

Fraps is probably the better monitor for frame rates but it's in-game output is heavily averaged and hides outliers. The csv output when running a benchmark (default `F11`) is very detailed, but difficult to easily parse. Therefore, I wrote a script to do some data analysis and plot graphs:

## Usage:

1. [Download fraps](https://fraps.com/download.php).
2. Run the benchmark tool and copy any number of frametime result files (in the form `* frametimes.csv`) to the same directory as the script.
3. Execute the script, it will load and analyse all relevent csv files in the same directory.

## Dependencies:
- matplotlib
- pandas
- scipy

## Examples:

### Touhou 10
As a benchmark, I used Touhou 10, since it has a really stable framerate with v-sync:

![2.png]({{ site.baseurl }}/images/2021-01-01-frame_time_analysis/2.png)

With an output of:
```
Analysis of th10 2021-01-01 18-20-55-38 frametimes.csv
Frame Time Average 16.673191570881226
Frame Time Standard Deviation 0.17137165234384277
Frames Per Second Average 59.98284730571625
Frames Per Second Standard Deviation 0.6164196631241836
FPS Percentiles:
01: 58.32177222060414 10: 59.343659130021884
99: 61.81883951574242 90: 60.59651225166108
```
Which reflects the stable gameplay, with no outlier frames. The stable fps really compliments the gameplay, where a single frame dropped can easily lead to a death.

### Halo 3
I also wanted to try out the port of Halo 3 on MCC for the pc. This is probably the best port out of the MCC, but it still doesn't feel perfect in combat:

![3.png]({{ site.baseurl }}/images/2021-01-01-frame_time_analysis/3.png)

The frame drops are noticeable in both combat and non-combat sections. This makes me think theres some sort of loading going on in the background thats storage limited. But it could just be the usual spaghetti thats keeping MCC running. The overall impact isn't too bad in theory, since the percentiles are quite good:

```
Frame Time Average 5.382341668561036
Frame Time Standard Deviation 5.900677549318819
Frames Per Second Average 220.48527275079527
Frames Per Second Standard Deviation 73.69543615204748
FPS Percentiles:
01: 79.23299856778335 10: 138.45813195990996
99: 426.27931886219443 90: 322.6847370119227
```
And Halo 3 isn't too stressful, so the drops aren't too big of a problem. Still a pretty good port.

### Tekken
I swear anytime I play Tekken on highest settings I always drop combos with correct inputs. Even when it only using 70% of my GPU with vsync and a constant 60 fps, and theres no way I'm just bad at Tekken (maybe). The dropped frames I suspected are pretty evident:

![4.png]({{ site.baseurl }}/images/2021-01-01-frame_time_analysis/4.png)

So my FADCs aren't just subpar, just swallowed by dropped frames (some atleast, I suspect I could also just be bad). Its pretty annoying to run games at very low settings just to avoid the one in a thousand dropped frames. One would expect a bit more love spent to optimise these aspects by developers, which leads to the original purpose of this script.

## Motivation
The initial motive was to explain some of some chunky behaviour of my engine. The gameplay is really fun, with relativly high framerates, but the occasional frame drops are hard to ignore:

![5.png]({{ site.baseurl }}/images/2021-01-01-frame_time_analysis/5.png)

The frame drops occur once every 10ms. Interestingly, the garbage collection is also on a 10ms timer. Initially I though the frame drops have to do with either the script queue being saturated when picking up powerups or a different tick rate between position updates and rendering. At this point, I'm pretty sure its the garbage collector locking all other threads when being run. 

Unfortunatly, I wrote the garbage collector to not lock other threads when reclaiming rescources. This might be tricky to fix, since it will involve finding a tacit lock. 

## Conclusion
Generally, I dislike reviews of video games that don't give any performance metrics. And any performance metrics tend to be an mean. Some ports do suffer pretty badly from dropped frames, and it would be nice to get a bit of a warning for some example hardware. On the other side of the coin, I dislike games that don't try to optimise away frame drops, especially when its a bottle neck that could be avoided with a bit of threading, a relaxed lock or a smart design patern. 
This script is released under the Unlicense, so feel free to use and modify as much as desired.
