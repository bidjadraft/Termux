
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

```

# How to Use This Script

1. **Start the script**  
   Run the Python script. It will first check if you have a saved Gemini API key.

2. **Enter or confirm Gemini API key**  
   - If you don’t have the key saved, the script will ask:  
     `Gemini API:`  
     Enter your Gemini API key here.  
   - The script tests the key automatically. If invalid, it asks again.  
   - If no internet, it will ask if you want to retry or exit.

3. **Input the audio/video file path**  
   When prompted:  
   `Path:`  
   Enter the path to your audio or video file.  
   - If the file does not exist or format is unsupported, it will show an error and ask again.

4. **For video files:**  
   - The script extracts the audio automatically and processes that.

5. **Display file info and estimated parts**  
   The script shows:  
   - Total time and size of the file.  
   - Number of parts (each around 30 minutes).  
   - Estimated size per part.

6. **Decide on compression if parts are large (>20MB)**  
   If parts are large, it asks:  
   `Compress before splitting? (Y/N):`  
   - `Y`: It compresses the audio with lower bitrate and mono channel and shows progress bar.  
   - `N`: You can enter a new file path or continue with current file.

7. **If the audio is longer than 30 minutes:**  
   - The script splits the audio into 30-minute parts, showing a spinner for progress.  
   - For each part:  
     - Starts an upload session.  
     - Uploads the audio part with a spinner for upload status.  
     - Transcribes it using the Gemini API, showing a spinner for transcription.  
     - If any step fails (upload or transcription), it asks if you want to retry.  
   - After all parts are processed, it combines all transcriptions into one text file.

8. **If the audio is shorter than 30 minutes:**  
   - If bigger than 20MB, it asks:  
     `Compress? (Y/N):`  
     - `Y`: Compress and update file info.  
     - `N`: Proceed without compression.  
   - Uploads the file and transcribes it like in the multi-part process.

9. **Saving the transcript**  
   - The final transcript is saved in the same directory as the original file, with the suffix `_transcript.txt`.  
   - The script prints the saved file relative path.

10. **Cleanup**  
    - Temporary files (split parts, compressed files, extracted audio) are deleted automatically after success.

---




If you found this project helpful, consider supporting me:

[![Donate](https://i.imgur.com/LjjhKAO.png)](https://ko-fi.com/bidjadraft)
