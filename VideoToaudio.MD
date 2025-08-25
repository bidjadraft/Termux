
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
1. create new file
```bash
nano FILENAME.sh
```
---
2. Paste this script into the file
```bash
#!/bin/sh

spinner() {
  while :
  do
    for c in '-' '\' '|' '/'
    do
      printf "\r%s %s" "$1" "$c"
      sleep 0.1
    done
  done
}

if ! command -v ffmpeg > /dev/null 2>&1; then
  while true; do
    echo -n "ffmpeg not found. install now (Y/N)? "
    read answer
    case $answer in
      [Yy]* )
        spinner "download running" &
        spinner_pid=$!
        yes | pkg install ffmpeg > /dev/null 2>&1
        kill $spinner_pid
        wait $spinner_pid 2>/dev/null
        echo -e "\rDownload completed.            "
        break
        ;;
      [Nn]* )
        echo "You can't without ffmpeg."
        exit 1
        ;;
      * )
        echo ""
        ;;
    esac
  done
fi

while true; do
  echo -n "path: "
  read video_path
  if [ -z "$video_path" ]; then
    continue
  fi
  ext="${video_path##*.}"
  ext_lower=$(echo "$ext" | tr '[:upper:]' '[:lower:]')
  case $ext_lower in
    mp4|mov|avi|mkv|flv|webm)
      break
      ;;
    *)
      echo "support only: mp4,mov,avi,mkv,flv,webm"
      ;;
  esac
done

filename=$(basename -- "$video_path")
filename="${filename%.*}"
folder_path=$(dirname "$video_path")
audio_ext="m4a"

output_audio="$folder_path/${filename}_audio.$audio_ext"
count=1
while [ -f "$output_audio" ]; do
  output_audio="$folder_path/${filename}_audio($count).$audio_ext"
  count=$((count + 1))
done

while true; do
  spinner "extracting audio is running" &
  spinner_pid=$!

  ffmpeg -i "$video_path" -vn -acodec copy "$output_audio" > /dev/null 2>&1
  ret=$?

  kill $spinner_pid
  wait $spinner_pid 2>/dev/null
  if [ $ret -eq 0 ]; then
    printf "\rsaved: %s\n" "$output_audio"
  else
    echo "Error extracting audio."
  fi

  # إعادة الطلب للمسار لتنفيذ عملية جديدة
  echo -n "path: "
  read video_path
  if [ -z "$video_path" ]; then
    break
  fi
  ext="${video_path##*.}"
  ext_lower=$(echo "$ext" | tr '[:upper:]' '[:lower:]')
  case $ext_lower in
    mp4|mov|avi|mkv|flv|webm)
      filename=$(basename -- "$video_path")
      filename="${filename%.*}"
      folder_path=$(dirname "$video_path")
      output_audio="$folder_path/${filename}_audio.$audio_ext"
      count=1
      while [ -f "$output_audio" ]; do
        output_audio="$folder_path/${filename}_audio($count).$audio_ext"
        count=$((count + 1))
      done
      ;;
    *)
      echo "support only: mp4,mov,avi,mkv,flv,webm"
      break
      ;;
  esac
done
```
---
3. save the file by click:
**cntrl + x + y + enter**
---
4. Giving file permissions
```bash
chmod +x FILENAME.sh
```
---
5. Running the file
```bash
./FILENAME.sh
```


# How to Use This Script
1. Upon running the script, you will be prompted to download the ffmpeg tool; choose "y" and wait for the installation to complete.
2. Next, you will be prompted to enter the video file path.
3. After entering the correct path, the script will begin the audio
extraction process at a fast speed.
4. The extracted file will be saved in the same folder as the video with a different name.


If you found this project helpful, consider supporting me:

[![Donate](https://i.imgur.com/LjjhKAO.png)](https://ko-fi.com/bidjadraft)
