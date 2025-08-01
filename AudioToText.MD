
​# Audio and Video Transcription Using Google Gemini API in Termux for Free

​￼## Overview
This script is written in Python and automates the process of converting audio or video files into written text using the Google Gemini API. It is designed to run in the Termux environment or any similar Linux system, handling large files efficiently by splitting and compressing them when needed, then uploading in parts and combining the resulting texts into a single file.

​￼## Features

​￼- Supports common audio formats:  
  `WAV, MP3, M4A, FLAC, OPUS, OGG

​￼- Supports popular video formats:  
  `MP4, MOV, MKV, AVI, FLV, WM`

- Supports long clips of 2 hours or more

- Simple to use without typing commands

​￼### Screenshots 

![Screenshot 1](https://i.imgur.com/b5OQZL5.jpeg)

![Screenshot 2](https://i.imgur.com/DixjIsR.jpeg)

| File Size       | Duration         | Action                                   | Speed               |
| --------------- | ---------------- | ---------------------------------------- | ------------------- |
| Less than 20 MB | Less than 30 min | Upload the file directly                 | Fastest             |
| Less than 20 MB | More than 30 min | Split the file, then upload              | Fastest             |
| More than 20 MB | Less than 30 min | Compress the file, then upload           | Good                |
| More than 20 MB | More than 30 min | Compress the file, then split and upload | Best for long clips |
