
# Audio and Video Transcription Using Google Gemini API in Termux for Free

# Overview
[![watch video on youtube](https://img.youtube.com/vi/rP7Nzy-No3s/hqdefault.jpg)](https://www.youtube.com/watch?v=rP7Nzy-No3s)

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

```python
import os
import subprocess
import requests
import sys
import re
import threading
import time
import random

API_KEY_FILE = "gemini_api_key.txt"
TEMP_DIR = "temp"
os.makedirs(TEMP_DIR, exist_ok=True)

PART_DURATION = 30 * 60  # 30 دقيقة بالثواني
BASE_PATH_TO_REMOVE = "/storage/emulated/0/"

SUPPORTED_AUDIO_EXTS = {
    ".wav": "audio/wav",
    ".mp3": "audio/mpeg",
    ".m4a": "audio/mp4",
    ".flac": "audio/flac",
    ".opus": "audio/opus",
    ".ogg": "audio/ogg"
}

SUPPORTED_VIDEO_EXTS = ['.mp4', '.mov', '.mkv', '.avi', '.flv', '.wmv', '.webm']

def check_internet_connection():
    try:
        requests.get("https://www.google.com", timeout=3)
        return True
    except Exception:
        return False

def wait_for_network(operation_text="waiting for Network"):
    dots = ['', '.', '..', '...']
    i = 0
    while not check_internet_connection():
        print(f'\r{operation_text}{dots[i % len(dots)]}   ', end='', flush=True)
        i += 1
        time.sleep(1)
    print('\r' + ' ' * (len(operation_text) + 5) + '\r', end='', flush=True)

def test_api_key(api_key):
    url = f"https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key={api_key}"
    payload = {"contents": [{"parts": [{"text": "API key test"}]}]}
    headers = {"Content-Type": "application/json"}
    try:
        response = requests.post(url, headers=headers, json=payload, timeout=10)
        if response.status_code == 200:
            return True
        else:
            if not check_internet_connection():
                return "no_internet"
            return False
    except Exception:
        if not check_internet_connection():
            return "no_internet"
        return False

def get_api_key():
    if os.path.exists(API_KEY_FILE):
        with open(API_KEY_FILE, "r") as f:
            stored_api_key = f.read().strip()
            valid = test_api_key(stored_api_key)
            if valid is True:
                return stored_api_key
            elif valid == "no_internet":
                while True:
                    retry = input("no internet. Try again (Y/N)? ").strip().lower()
                    if retry == 'y':
                        return get_api_key()
                    elif retry == 'n':
                        print("Exiting due to no internet connection.")
                        sys.exit(0)
                    else:
                        print("Please enter Y or N.")
    while True:
        api_key = input("Gemini API: ").strip()
        valid = test_api_key(api_key)
        if valid is True:
            with open(API_KEY_FILE, "w") as f:
                f.write(api_key)
            return api_key
        elif valid == "no_internet":
            while True:
                retry = input("no internet. Try again (Y/N)? ").strip().lower()
                if retry == "y":
                    break
                elif retry == "n":
                    print("Exiting due to no internet connection.")
                    sys.exit(0)
                else:
                    print("Please enter Y or N.")
        else:
            print("Invalid key.")

def get_audio_duration(input_path):
    try:
        cmd = [
            "ffprobe", "-v", "error",
            "-show_entries", "format=duration",
            "-of", "default=noprint_wrappers=1:nokey=1",
            input_path
        ]
        result = subprocess.run(cmd, stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True, check=True)
        return float(result.stdout.strip())
    except Exception:
        return None

def spinner_task(text, done_flag):
    symbols = ['|', '/', '-', '\\']
    idx = 0
    while not done_flag[0]:
        sys.stdout.write('\r' + text + ' ' + symbols[idx % len(symbols)])
        sys.stdout.flush()
        idx += 1
        time.sleep(0.1)
    sys.stdout.write('\r' + ' ' * (len(text) + 2) + '\r')
    sys.stdout.flush()

def start_upload_session(file_path, mime_type, api_key):
    size = os.path.getsize(file_path)
    metadata = {"file": {"displayName": os.path.basename(file_path)}}
    headers = {
        "X-Goog-Upload-Protocol": "resumable",
        "X-Goog-Upload-Command": "start",
        "X-Goog-Upload-Header-Content-Length": str(size),
        "X-Goog-Upload-Header-Content-Type": mime_type,
        "Content-Type": "application/json"
    }
    url = f"https://generativelanguage.googleapis.com/upload/v1beta/files?key={api_key}"
    while True:
        try:
            resp = requests.post(url, headers=headers, json=metadata)
            if resp.status_code != 200:
                print("Upload session failed:", resp.text)
                return None
            return resp.headers.get("X-Goog-Upload-URL")
        except Exception:
            if not check_internet_connection():
                wait_for_network("waiting for Upload session")
                continue
            else:
                print("Upload session failed due to network error")
                retry = input("Retry upload session start? (Y/N): ").strip().lower()
                if retry != "y":
                    return None

def upload_file_data(upload_url, file_path, mime_type, part_idx=None, total_parts=None):
    size = os.path.getsize(file_path)
    with open(file_path, "rb") as f:
        data = f.read()
    headers = {
        "X-Goog-Upload-Command": "upload, finalize",
        "X-Goog-Upload-Offset": "0",
        "Content-Type": mime_type,
        "Content-Length": str(size)
    }
    operation_text = f"Uploading file {part_idx}/{total_parts}..." if part_idx and total_parts else "Uploading file..."
    while True:
        done_flag = [False]
        spinner_thread = threading.Thread(target=spinner_task, args=(operation_text, done_flag))
        spinner_thread.start()
        try:
            resp = requests.post(upload_url, headers=headers, data=data)
            done_flag[0] = True
            spinner_thread.join()
            if resp.status_code != 200:
                print(f"\nUpload failed (part {part_idx}):", resp.text)
                retry = input("Retry upload (Y/N)? ").strip().lower()
                if retry == "y":
                    continue
                else:
                    return None
            return resp.json().get("file", {}).get("uri")
        except Exception:
            done_flag[0] = True
            spinner_thread.join()
            if not check_internet_connection():
                wait_for_network("waiting for Network")
                continue
            else:
                retry = input("\nUpload failed due to error. Retry (Y/N)? ").strip().lower()
                if retry == "y":
                    continue
                else:
                    return None

def send_request_to_gemini(file_uri, mime_type, prompt, api_key, part_idx=None, total_parts=None):
    payload = {
        "contents": [
            {
                "parts": [
                    {"text": prompt},
                    {
                        "fileData": {
                            "fileUri": file_uri,
                            "mimeType": mime_type
                        }
                    }
                ]
            }
        ]
    }
    headers = {"Content-Type": "application/json"}
    operation_text = f"Transcribing {part_idx}/{total_parts}..." if part_idx and total_parts else "Transcribing..."

    max_retries = 10
    base_delay = 1  # بالثواني

    for attempt in range(1, max_retries + 1):
        done_flag = [False]
        spinner_thread = threading.Thread(target=spinner_task, args=(operation_text, done_flag))
        spinner_thread.start()
        try:
            resp = requests.post(
                f"https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key={api_key}",
                headers=headers,
                json=payload,
                timeout=60
            )
            done_flag[0] = True
            spinner_thread.join()
            if resp.status_code == 200:
                try:
                    return resp.json()["candidates"][0]["content"]["parts"][0]["text"]
                except Exception:
                    print(f"\nError parsing transcription response for part {part_idx}. Retrying...")
            else:
                print(f"\nAPI error (part {part_idx}): {resp.status_code}. Retrying...")

        except Exception:
            done_flag[0] = True
            spinner_thread.join()
            if not check_internet_connection():
                wait_for_network("waiting for Network")
                continue
            else:
                print("\nAPI request failed due to network error. Retrying...")

        delay = base_delay * (2 ** (attempt - 1)) + random.uniform(0, 0.5)
        delay = min(delay, 60)  # حد أقصى للانتظار 60 ثانية
        print(f"Waiting for {delay:.1f} seconds before retry #{attempt}...")
        time.sleep(delay)

    print(f"\nFailed transcription for part {part_idx} after {max_retries} retries.")
    return None

def compress_with_progress(input_path, duration_sec):
    base, ext = os.path.splitext(input_path)
    ext = ext.lower()
    if ext in ['.wav', '.flac']:
        compressed_file_path = base + "_resized_small.opus"
        codec = 'libopus'
    else:
        compressed_file_path = base + "_resized_small" + ext
        if ext == '.mp3':
            codec = 'libmp3lame'
        elif ext == '.opus':
            codec = 'libopus'
        elif ext in ['.m4a', '.aac']:
            codec = 'aac'
        elif ext == '.ogg':
            codec = 'libvorbis'
        else:
            codec = 'aac'
    cmd = [
        "ffmpeg", "-y", "-i", input_path,
        "-c:a", codec,
        "-b:a", "32k",
        "-ar", "16000",
        "-ac", "1",
        "-preset", "ultrafast",
        "-threads", "4",
        compressed_file_path
    ]
    time_pattern = re.compile(r"time=(\d{2}:\d{2}:\d{2}\.\d{2})")
    try:
        process = subprocess.Popen(cmd, stderr=subprocess.PIPE, universal_newlines=True, bufsize=1)
        last_progress = -1
        for line in process.stderr:
            line = line.strip()
            match = time_pattern.search(line)
            if match:
                h, m, s = match.group(1).split(':')
                current_time = int(h) * 3600 + int(m) * 60 + float(s)
                progress = min(current_time / duration_sec, 1.0)
                percent = int(progress * 100)
                if percent != last_progress:
                    bar_length = 20
                    filled_length = int(bar_length * progress)
                    bar = '█' * filled_length + '-' * (bar_length - filled_length)
                    sys.stdout.write(f'\r|{bar}| {percent}%')
                    sys.stdout.flush()
                    last_progress = percent
        process.wait()
        sys.stdout.write('\n')
        if process.returncode == 0:
            return compressed_file_path
        else:
            print("Compression failed with error code:", process.returncode)
            return None
    except Exception as e:
        print(f"\nCompression error: {e}")
        retry = input("Compression failed. Retry? (Y/N): ").strip().lower()
        if retry == "y":
            return compress_with_progress(input_path, duration_sec)
        else:
            return None

def split_audio_to_parts(input_path, part_duration_sec=PART_DURATION):
    duration = get_audio_duration(input_path)
    if duration is None:
        return []
    parts = []
    base, ext = os.path.splitext(input_path)
    start = 0
    index = 1
    total_parts = int((duration + part_duration_sec - 1) // part_duration_sec)
    done_flag = [False]
    spinner_thread = threading.Thread(target=spinner_task, args=("Creating parts...", done_flag))
    spinner_thread.start()
    try:
        while start < duration:
            part_path = f"{base}_part{index}{ext}"
            length = min(part_duration_sec, duration - start)
            cmd = ["ffmpeg", "-y", "-i", input_path, "-ss", str(start), "-t", str(length), "-c", "copy", part_path]
            try:
                subprocess.run(cmd, check=True, stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL)
                parts.append(part_path)
            except subprocess.CalledProcessError:
                print(f"\nError splitting part {index}.")
                retry = input("Retry splitting this part? (Y/N): ").strip().lower()
                if retry != 'y':
                    done_flag[0] = True
                    spinner_thread.join()
                    return parts
                else:
                    continue
            index += 1
            start += part_duration_sec
    finally:
        done_flag[0] = True
        spinner_thread.join()
    print(f"Created {len(parts)}/{total_parts} parts.")
    return parts

def relative_output_path(full_path):
    try:
        return os.path.relpath(full_path, BASE_PATH_TO_REMOVE)
    except ValueError:
        return full_path

def seconds_to_hms(seconds):
    h = int(seconds // 3600)
    m = int((seconds % 3600) // 60)
    s = int(seconds % 60)
    return f"{h:02d}:{m:02d}:{s:02d}"

def main():
    api_key = get_api_key()
    while True:
        file_path = input("Path: ").strip()
        if not os.path.isfile(file_path):
            print("File not found.")
            continue
        original_path = file_path
        ext = os.path.splitext(file_path)[1].lower()
        compressed_file = None
        if ext in SUPPORTED_VIDEO_EXTS:
            temp_audio_path = os.path.join(TEMP_DIR, "extracted_audio.m4a")
            cmd = ["ffmpeg", "-y", "-i", file_path, "-vn", "-acodec", "copy", temp_audio_path]
            try:
                subprocess.run(cmd, stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL, check=True)
            except subprocess.CalledProcessError:
                print("Failed to extract audio from video.")
                continue
            file_path = temp_audio_path
            ext = ".m4a"
        if ext not in SUPPORTED_AUDIO_EXTS:
            print("Unsupported format.")
            continue
        duration_sec = get_audio_duration(file_path)
        if duration_sec is None:
            print("Unknown duration.")
            continue
        size_mb = os.path.getsize(file_path) / (1024 * 1024)
        total_parts = int((duration_sec + PART_DURATION - 1) // PART_DURATION)
        part_size_mb = size_mb / total_parts if total_parts > 0 else size_mb
        part_duration_min = PART_DURATION // 60
        print(f"Time:{seconds_to_hms(duration_sec)}  Size:{size_mb:.2f}MB")
        print(f"Parts:({total_parts})  Part:{part_duration_min}min → {part_size_mb:.2f}MB")
        while part_size_mb >= 20:
            ans = input("Compress before splitting? (Y/N): ").strip().lower()
            if ans == "y":
                compressed_file = compress_with_progress(file_path, duration_sec)
                if compressed_file is None or not os.path.isfile(compressed_file):
                    print("Compression failed.")
                    return
                file_path = compressed_file
                duration_sec = get_audio_duration(file_path)
                size_mb = os.path.getsize(file_path) / (1024 * 1024)
                total_parts = int((duration_sec + PART_DURATION - 1) // PART_DURATION)
                part_size_mb = size_mb / total_parts if total_parts > 0 else size_mb
                print(f"Compressed file size: {size_mb:.2f}MB")
                if part_size_mb < 20:
                    break
            elif ans == "n":
                new_path = input("Path: ").strip()
                if os.path.isfile(new_path):
                    ext2 = os.path.splitext(new_path)[1].lower()
                    if ext2 not in SUPPORTED_AUDIO_EXTS:
                        print("Unsupported format.")
                        continue
                    duration_sec2 = get_audio_duration(new_path)
                    if duration_sec2 is None:
                        print("Unknown duration.")
                        continue
                    size_mb2 = os.path.getsize(new_path) / (1024 * 1024)
                    total_parts2 = int((duration_sec2 + PART_DURATION - 1) // PART_DURATION)
                    part_size_mb2 = size_mb2 / total_parts2 if total_parts2 > 0 else size_mb2
                    print(f"Updated file info - Time:{seconds_to_hms(duration_sec2)}  Size:{size_mb2:.2f}MB")
                    if part_size_mb2 < 20:
                        file_path = new_path
                        duration_sec = duration_sec2
                        size_mb = size_mb2
                        total_parts = total_parts2
                        part_size_mb = part_size_mb2
                        compressed_file = None
                        break
                    else:
                        print(f"Parts:({total_parts2})  Part:{part_duration_min}min → {part_size_mb2:.2f}MB")
                else:
                    print("File not found.")
            else:
                print("Invalid input, please enter Y or N.")
        if duration_sec > PART_DURATION:
            parts = split_audio_to_parts(file_path, PART_DURATION)
            if not parts:
                print("Failed to split audio.")
                return
            mime_type = SUPPORTED_AUDIO_EXTS.get(os.path.splitext(parts[0])[1].lower())
            if not mime_type:
                print("Unsupported MIME type for parts.")
                return
            prompt = "Convert audio content to accurate text."
            combined_text = ""
            for idx, part_file in enumerate(parts, 1):
                upload_url = start_upload_session(part_file, mime_type, api_key)
                if not upload_url:
                    print(f"\nFailed to start upload session for part {idx}")
                    return
                file_uri = upload_file_data(upload_url, part_file, mime_type, idx, len(parts))
                if not file_uri:
                    print(f"\nFailed to upload part {idx}")
                    return
                text = send_request_to_gemini(file_uri, mime_type, prompt, api_key, idx, len(parts))
                if text is None:
                    print(f"\nTranscription failed for part {idx}.")
                    return
                combined_text += text + "\n\n"
            output_dir = os.path.dirname(original_path)
            base_name = os.path.splitext(os.path.basename(original_path))[0]
            output_file = os.path.join(output_dir, base_name + "_transcript.txt")
            try:
                with open(output_file, "w", encoding="utf-8") as f:
                    f.write(combined_text.strip())
            except Exception as e:
                print(f"Failed to save transcript file: {e}")
                return
            print(f"saved: {relative_output_path(output_file)}")
            for fpath in parts:
                try:
                    os.remove(fpath)
                except Exception:
                    pass
            if compressed_file is not None:
                try:
                    os.remove(compressed_file)
                except Exception:
                    pass
            temp_audio_path = os.path.join(TEMP_DIR, "extracted_audio.m4a")
            if os.path.exists(temp_audio_path):
                try:
                    os.remove(temp_audio_path)
                except Exception:
                    pass
            return
        else:
            if size_mb > 20:
                ans = input("Compress? (Y/N): ").strip().lower()
                if ans == "y":
                    compressed_file = compress_with_progress(file_path, duration_sec)
                    if compressed_file is None or not os.path.isfile(compressed_file):
                        print("Compression failed.")
                        continue
                    file_path = compressed_file
                    size_mb = os.path.getsize(file_path) / (1024 * 1024)
                    duration_sec = get_audio_duration(file_path)
                    print(f"Done. New size: {size_mb:.2f}MB")
                    if size_mb > 20:
                        print("Warning: still >20MB after compress.")
                else:
                    print("Enter another file path.")
                    continue
            mime_type = SUPPORTED_AUDIO_EXTS[ext]
            prompt = "Convert audio content to accurate text."
            upload_url = start_upload_session(file_path, mime_type, api_key)
            if not upload_url:
                return
            file_uri = upload_file_data(upload_url, file_path, mime_type)
            if not file_uri:
                return
            text = send_request_to_gemini(file_uri, mime_type, prompt, api_key)
            if text is None:
                print("Transcription failed.")
                return
            output_dir = os.path.dirname(original_path)
            base_name = os.path.splitext(os.path.basename(original_path))[0]
            output_file = os.path.join(output_dir, base_name + "_transcript.txt")
            try:
                with open(output_file, "w", encoding="utf-8") as f:
                    f.write(text)
            except Exception as e:
                print(f"Failed to save transcript file: {e}")
                return
            print(f"saved: {relative_output_path(output_file)}")
            if compressed_file is not None:
                try:
                    os.remove(compressed_file)
                except Exception:
                    pass
            return

if __name__ == "__main__":
    main()


```





---

# How This Code Works


| File Size       | Duration         | Action                                   | Speed               |
| --------------- | ---------------- | ---------------------------------------- | ------------------- |
| Less than 20 MB | Less than 30 min | Upload the file directly                 | Fastest             |
| Less than 20 MB | More than 30 min | Split the file, then upload              | Fastest             |
| More than 20 MB | Less than 30 min | Compress the file, then upload           | Good                |
| More than 20 MB | More than 30 min | Compress the file, then split and upload | Best for long clips |

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

### Summary of Main Prompts and Inputs

| Prompt                             | What to Do                               |
|----------------------------------|-----------------------------------------|
| `Gemini API:`                    | Enter your Gemini API key                |
| `Path:`                         | Enter the full file path of your audio or video file |
| `Compress before splitting? (Y/N):` | Type `Y` to compress or `N` to skip    |
| `Retry upload session start? (Y/N):` | Type `Y` to retry upload or `N` to cancel |
| `Retry upload (Y/N)?`            | Type `Y` to retry upload or `N` to cancel |
| `Retry transcription (Y/N)?`     | Type `Y` to retry transcription or `N` to cancel |
| `Compress? (Y/N):`               | Type `Y` to compress or `N` to proceed without compression |

---

Using this process, you can convert your audio or video files into text transcripts by following the interactive prompts step-by-step.

If you found this project helpful, consider supporting me:

[![Donate](https://i.imgur.com/LjjhKAO.png)](https://ko-fi.com/bidjadraft)
