
# Extract Audio from Video with Termux and FFMPEG

[![watch video on youtube](https://img.youtube.com/vi/rP7Nzy-No3s/hqdefault.jpg)](https://www.youtube.com/watch?v=rP7Nzy-No3s)
> **Note**  
> Click image to watch video.

# Overview
The script's main function is to extract the audio track from video files using ffmpeg, automating the process with user prompts and progress indication.

# Features

- Supports common Video formats:  
  `MP4, MOV, MKV`

- Handles very long video files efficiently.

- Performs fast, lossless audio extraction without re-encoding.

- User-friendly with no coding required; uses simple interactive prompts.

# Screenshots 

![Screenshot 1](https://i.imgur.com/79O1NYn.png)

![Screenshot 2](https://i.imgur.com/tTXgBa6.png)

---
# Requirements
You will only need the `FFMPEG` tool, but all of this will happen automatically upon running the code. Therefore, follow these steps:

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
  mp4|mov|mkv)
    break
    ;;
  *)
    echo "support only: mp4, mov, mkv"
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
  >`Ctrl` + `X` + `Y` + `Enter`

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
1. Upon running the script, you will be prompted to download the `FFMPEG` tool, choose `Y` and wait for the installation to complete.
2. Next, you will be prompted to enter the video file `PATH`.
3. Use a file manager that supports copying paths, such as [Mixplorer](https://drive.google.com/drive/u/0/mobile/folders/1BfeK39boriHy-9q76eXLLqbCwfV17-Gv?usp=share_link&pli=1) or [Material Files](https://f-droid.org/packages/me.zhanghai.android.files/)
4. After entering the correct path, the script will begin the audio
extraction process at a fast speed.
5. The extracted file will be `SAVED` in the same folder as the video with a different name.

---

If you found this project helpful, consider supporting me:

[![Donate](https://i.imgur.com/LjjhKAO.png)](https://ko-fi.com/bidjadraft)
