---
layout: post
title:  "Modify audio and video files"
date:   2021-02-28 11:00:00 +0100
categories: ffmpeg
---


# Converting to match requirements from linguistic software

To convert from mp3 to wav, simply use the following command line: `ffmpeg -i input.mp3 output.wav`.

In order to convert multiple files at once use: `for i in *.mp3; do ffmpeg -i "$i" "${i%.*}.wav"; done`.

# Make a file Instragramo compatible with subtitles
The goal was to put "Boxing in English with Alex" episodes, from [Planet Citizens][planet-citizens],  in their Instagram stories (which requires a specific format).
#### Cut to 1 minute
To trim a file:  `ffmpeg -ss 00:01:00 -i input.mp4 -to 00:02:00 -c copy output.mp4`

Can use `-t` instead of `-to` to indicate a duration instead of an end time. 
`-c copy` does a faster job, but it might have weird results in some cases.

#### Encoding
Re-encoding might be a good thing to do to keep quality since it will do a proper job in encoding the video (instead of keeping the former encoding which might be not the best choice for the chosen portion of the video, especially if the portion does not start with an I-frame). 
See [this page][trim-doc] for more informations.

I found Instagram specifications on [a blog][insta-blog]: 
* H.264 code
* AAC audio
* 3500 kbps bitrate
* Frame rate of 30 fps (frames per second)
* Video can be a maximum of 60 seconds
* Maximum video width is 1080 px (pixels) wide
* Videos should be 920 pixels tall

Since I am not sure what I should do, I decided to use a website that downloads from youtube and creates videos directly usable for instagram. 
I then compare the encoding of the resulting video with one I cut from the original file with ffmpeg using `diff ffmpegTube2gram.txt ffmpegBoxingInEnglish2-1.txt`. 
The two files are copied from the error message I got when running the command `ffmpeg -i input.mp4`. An output in expected but this gives me the encoding. 

I used the following command to get the right encoding necessary to upload to Instagram. Cut to 1 minute, this resulted in a 24 Mb video file.

```
ffmpeg -i input.mp4 -acodec aac -b:a 128k -vcodec mpeg4 -b:v 3200k output.mp4
``` 

#### Subtitles

I wanted to ["burn" subtitles to the video][ffmpeg-subtitles], so that they appear on Instagram.
The command for this is the following, forcing the font size to 30: 

```
ffmpeg -i input.mp4 -vf subtitles=subtitles.srt:force_style='Fontsize=30' output.mp4
```


In my case I wanted to add one line for French and one line for English subtitles. 
I did not manage to add subtitles from 2 different files, so I needed to merge the two files. 
For this I used a simple text editor.
The subtitle files have the following structure:

```
1
00:00:03,880 --> 00:00:05,760
I’m from Northampton, England, UK.

2
00:00:05,760 --> 00:00:08,320
My dad is English and my mom is from the Philippines.
```

I first removed the lines starting with a number (using the regular expression `^[0-9].*`).
I then removed all empty lines (replace 2 new lines with 1).
In order to distinguish the languages, I added [formatting][srt-format] with html tags at the beginning and end of each line `<font color="#808080" size="25"> ... </font>`.
And finally I copy-pasted the lines to the other file resulting in the following structure:
```
1
00:00:03,880 --> 00:00:05,760
I’m from Northampton, England, UK.
<font color="#808080" size="25">Je viens de Northampton, Angleterre, Royaume-Uni.</font>
```

Thanks to this structure, I can have two different colors in the subtitle (and even different fontsize).

#### Final command
```
ffmpeg -i BoxingInEnglish2.mp4 -vf subtitles=EP2.srt:force_style='Fontsize=30' -acodec aac -b:a 128k -vcodec mpeg4 -b:v 3200k -ss 00:02:00 -t 00:01:00 BIE2sub-3.mp4

```



[trim-doc]: https://ottverse.com/trim-cut-video-using-start-endtime-reencoding-ffmpeg/
[insta-blog]: https://www.oberlo.com/blog/best-instagram-video-format
[ffmpeg-subtitles]: https://trac.ffmpeg.org/wiki/HowToBurnSubtitlesIntoVideo
[srt-format]:https://en.wikipedia.org/wiki/SubRip#Formatting
[planet-citizens]: https://www.instagram.com/planet_citizens/


