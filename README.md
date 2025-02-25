# tv_encoder

This script allows you to easily convert video files to a format suitable for the digital television decoders or other digital video devices.
The script analyzes the specified video file (first video and all audio streams) and then starts recoding only those streams that are not supported by your device. All other streams are copied to the target container without any modification.

You can specify:

- List of supported video codecs
- List of supported audio codecs
- List of supported multimedia containers
- Target video codec
- Target audio codec
- Maximal video frame size
- Desired video stream quality (in BPP)

You can read more information inside the script in the comments.

The script uses the FFmpeg tool.

Tested on Debian 13 (trixie).

## Environment variables

The script uses environment variables to override the default parameters.
You can create your own script and specify other encoding parameters
to it via environment variables:

```
#!/usr/bin/env bash

export SUPP_AC=" mp3 aac "
export MAX_RES=1080
tv_recode "$@"
exit $?
```

Available environment variables:

- SUPP_CN - List of supported containers by you device. Example: SUPP_CN=" avi mkv "
- SUPP_VC - List of supported video codecs by your device. Expample: SUPP_VC=" h264 hevc "
- SUPP_AC - List of supported audio codecs by your device. Expample: SUPP_AC=" mp3 ac3 aac "
- MAX_RES - Maximum horizontal or vertical resolution of the frame in pixels for your device. Example: MAX_RES=1080
- PREF_VC - Video codec to be used when encoding video. Example: PREF_VC=h264
- PREF_AC - Audio codec to be used when encoding audio. Example: PREF_AC=aac
- PREF_CT - Multimedia container to be used as final video file if the current one is not support. Example: PREF_CT=mkv
- BPP     - The number of bits that will be allocated to store one pixel in the final file. The larger this value is, the higher the video quality and the larger the final file size. The final bitrate is calculated from this value.

Note that a leading and trailing spaces are required in the lists above.


## TODO

- Implement two-pass encoding

## What the script doesn't do:

- Does not change subtitles
- Does not change codec parameters
- Does not delete or overwrite existing video files
- Does not recode a stream if the stream's codec is already supported by your device
- Does not detect the codecs supported by your equipment. You must specify them yourself

## Examples

Display information about the actions required:

```sh
$ tv_encoder info video.mp4
Container  : mp4 --> mkv       <--- My TV doesn't support mp4 container
Video frame: 1280x540 --> copy
Video codec: h264 --> copy
Audio codec: aac --> copy

$tv_encoder info other_video.mkv
Container  : mkv --> copy
Video frame: 1280x534 --> copy
Video codec: h264 --> copy
Audio codec: ac3 --> copy
Audio codec: dts --> aac       <--- My TV doesn't support dts codec
Audio codec: aac --> copy
```

Start recoding the video.mp4 file and save the result to the done directory:

```sh
$ tv_encoder start video.mp4 done/
ffmpeg version 7.1-4 Copyright (c) 2000-2024 the FFmpeg developers
  built with gcc 14 (Debian 14.2.0-17)
...

$ ls done
video.mkv
```

Same as the previous example, but instead of executing the ffmpeg command, it will be output to the console with all the parameters:

```sh
$ tv_encoder dry video.mp4 video_for_tv.mkv
ffmpeg -i video.mp4 -map 0:v -map 0:a -map 0:s? -c copy -- done/video.mkv

$ tv_encoder dry other_video.mkv done/
ffmpeg -i other_video.mkv -map 0:v -map 0:a -map 0:s? -c copy -c:a:1 aac -- done/other_video.mkv
```

To recode several .avi files at a time and write the result to the done directory, you can use the following command:

```sh
find . -maxdepth 1 -name "*.avi" -exec tv_encoder start {} done/ \;
```
