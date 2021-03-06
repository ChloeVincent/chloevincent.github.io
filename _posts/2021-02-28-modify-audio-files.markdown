---
layout: post
title:  "Modify audio and video files"
date:   2021-02-11 11:00:00 +0100
categories: ffmpeg
---


convert mp3 to wav: `ffmpeg -i input.mp3 output.wav`

`for i in *.mp3; do ffmpeg -i "$i" "${i%.*}.wav"; done`

trim  `ffmpeg -ss 00:01:00 -i input.mp4 -to 00:02:00 -c copy output.mp4`

Can use `-t` instead of `-to` to indicate a duration instead of an end time. `-c copy` does a faster job, but it might have weird results in some cases.