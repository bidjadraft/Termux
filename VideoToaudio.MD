
# Audio and Video Transcription Using Google Gemini API in Termux for Free

[![watch video on youtube](https://img.youtube.com/vi/rP7Nzy-No3s/hqdefault.jpg)](https://www.youtube.com/watch?v=rP7Nzy-No3s)
> **Note**  
> Click image to watch video.

# Overview
This script is written in Python and automates the process of converting audio or video files into written text using the Google Gemini API. It is designed to run in the Termux environment or any similar Linux system, handling large files efficiently by splitting and compressing them when needed, then uploading in parts and combining the resulting texts into a single file.

# Features

- Supports common audio formats:  
  `WAV, MP3, M4A, FLAC, OPUS, OGG`

- Supports popular video formats:  
  `MP4, MOV, MKV, AVI, FLV, WM`

- Supports long clips of 2 hours or more

- Supports most languages including `English, Arabic, French,` and others

- Simple to use without typing codes

# Screenshots 

![Screenshot 1](https://i.imgur.com/b5OQZL5.jpeg)

![Screenshot 2](https://i.imgur.com/DixjIsR.jpeg)

---
# Requirements
1. First, you need to obtain a Gemini API Key from the [Google AI Studio](https://aistudio.google.com/app/apikey) website. If you don't know how to get it or face any issues, watch [this video](https://youtu.be/SMTLu2AT5tQ) for guidance.

2. **Now you need to install this on [Termux](https://termux.dev/en/):**
```bash
pkg install python
```
```bash
pkg install ffmpeg
```
```bash
pip install requests
```
```bash
termux-setup-storage
```

3. **create new file:**
 `nano filename.py`

4. **copy the code below and paste it into the file**

5. **to save the file:**
Press `Ctrl + X`, then `Y` to confirm saving, and finally `Enter` to confirm the filename.


# How to Use This Script



If you found this project helpful, consider supporting me:

[![Donate](https://i.imgur.com/LjjhKAO.png)](https://ko-fi.com/bidjadraft)
