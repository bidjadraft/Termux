
# Audio and Video Transcription Using Google Gemini API in Termux for Free

## Overview
This script is written in Python and automates the process of converting audio or video files into written text using the Google Gemini API. It is designed to run in the Termux environment or any similar Linux system, handling large files efficiently by splitting and compressing them when needed, then uploading in parts and combining the resulting texts into a single file.

## Features
-
- Supports common audio formats:  
  `WAV, MP3, M4A, FLAC, OPUS, OGG

- Supports popular video formats:  
  `MP4, MOV, MKV, AVI, FLV, WM`

- Supports long clips of 2 hours or more

- Simple to use without typing commands

### Screenshots 

![Screenshot 1](https://i.imgur.com/b5OQZL5.jpeg)

![Screenshot 2](https://i.imgur.com/DixjIsR.jpeg)

| File Size       | Duration         | Action                                   | Speed               |
| --------------- | ---------------- | ---------------------------------------- | ------------------- |
| Less than 20 MB | Less than 30 min | Upload the file directly                 | Fastest             |
| Less than 20 MB | More than 30 min | Split the file, then upload              | Fastest             |
| More than 20 MB | Less than 30 min | Compress the file, then upload           | Good                |
| More than 20 MB | More than 30 min | Compress the file, then split and upload | Best for long clips |
# How This Code Works

1. **Setup and Constants**  
   - Creates a temporary directory for intermediate files.  
   - Defines supported audio and video file extensions and sets constants like part duration (30 minutes).

2. **Internet and API Key Handling**  
   - Checks for internet connectivity.  
   - Reads Gemini API key from a local file or prompts user for it.  
   - Tests the API key validity by making a test request to Gemini API.

3. **File Type and Duration Processing**  
   - Supports extraction of audio from supported video files.  
   - Determines the duration of the audio file using ffprobe.

4. **Compression Logic**  
   - If file size is large (>20MB per part), offers to compress the audio using ffmpeg with low bitrate and mono channel.  
   - Shows compression progress via a console progress bar.

5. **Splitting Audio**  
   - Splits long audio files (>30 minutes) into 30-minute parts using ffmpeg.  
   - Displays a spinner during the splitting process.

6. **Uploading Files**  
   - Starts a resumable upload session with the Gemini API.  
   - Uploads file bytes with progress spinner and error handling.  
   - Supports retry on failures or network issues.

7. **Transcription Request**  
   - Sends the uploaded file URI along with a prompt to Gemini API's transcription endpoint.  
   - Shows spinner while waiting for API response.  
   - Supports retrying if transcription fails or parsing errors occur.

8. **Combining and Saving Output**  
   - Combines transcription results from parts into one final transcript text.  
   - Saves the transcript to a text file in the original file's directory.  
   - Prints the relative path of the saved transcript file.

9. **Cleanup**  
   - Deletes temporary parts, compressed files, and extracted audio after successful transcription.

10. **User Interaction**  
    - Provides prompts for retrying operations if errors occur.  
    - Allows users to re-enter file paths or decide on compression steps.

Overall, the script converts audio or video files into accurate text transcripts by leveraging Gemini's generative language API, handling large files by splitting, compressing when needed, and managing network reliability gracefully.
