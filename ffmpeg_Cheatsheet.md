# ffmpeg cheat sheet

[Download ffmpeg](https://ffmpeg.org/)

[ffmpeg Documentation](https://ffmpeg.org/ffmpeg-all.html)

[ffmpeg wiki](https://trac.ffmpeg.org/wiki)

[ffmprovisr](https://amiaopensource.github.io/ffmprovisr/#abitscope)

[ffmpeg-commands](https://catswhocode.com/ffmpeg-commands/)

[Video Size Syntax Overview](https://ffmpeg.org/ffmpeg-utils.html#video-size-syntax)

[Audio Filter Documentation](https://ffmpeg.org/ffmpeg-filters.html#Audio-Filters)


### Get File Information

```
ffmpeg -i video.avi
```

```
ffmpeg -i audio.mp3
```


### Wav to Mp3

```
ffmpeg -i input_file.wav -write_id3v1 1 -id3v2_version 3 -dither_method rectangular -out_sample_rate 48k -qscale:a 1 output_file.mp3


ffmpeg -i input.mp3 output.wav
```

- ffmpeg
  starts the command
- -i *input_file*
  path and name of the input file
- -write_id3v1 1
  This will write metadata to an ID3v1 tag at the head of the file, assuming you’ve embedded metadata into the WAV file.
- -id3v2_version 3
  This will write metadata to an ID3v2.3 tag at the tail of the file, assuming you’ve embedded metadata into the WAV file.
- -dither_method rectangular
  Dither makes sure you don’t unnecessarily truncate the dynamic range of your audio.
- -out_sample_rate 48k
  Sets the audio sampling frequency to 48 kHz. This can be omitted to use the same sampling frequency as the input.
- -qscale:a 1
  This sets the encoder to use a constant quality with a variable bitrate of between 190-250kbit/s. If you would prefer to use a constant bitrate, this could be replaced with `-b:a 320k` to set to the maximum bitrate allowed by the MP3 format. For more detailed discussion on variable vs constant bitrates see [here.](https://trac.ffmpeg.org/wiki/Encode/MP3)
- *output_file*
  path and name of the output file



### WAV to AAC/MP4
```
ffmpeg -i input_file.wav -c:a aac -b:a 128k -dither_method rectangular -ar 44100 output_file.mp4
```

- ffmpeg
starts the command
- -i *input_file*
path and name of the input file
- -c:a aac
sets the audio codec to AAC
- -b:a 128k
sets the bitrate of the audio to 128k
- -dither_method rectangular
Dither makes sure you don’t unnecessarily truncate the dynamic range of your audio.
- -ar 44100
sets the audio sampling frequency to 44100 Hz, or 44.1 kHz, or “CD quality”
- *output_file*
path and name of the output file


Mix a Video With a Sound File

```
ffmpeg -i sound.wav -i original_video.avi final_video.mpg
```



### Extract audio from an AV file

```
ffmpeg -i *input_file* -c:a copy -vn *output_file*
```

- ffmpeg
starts the command
- -i *input_file*
path, name and extension of the input file
- -c:a copy
re-encodes using the same audio codec
- -vn
no video stream
- *output_file*
path, name and extension of the output file



### Create an allround Visualizing Video from Audio File

```
ffmpeg -i input.mp3 -filter_complex \
"[0:a]avectorscope=s=640x518[left]; \
 [0:a]showspectrum=mode=separate:color=intensity:scale=cbrt:s=640x518[right]; \
 [0:a]showwaves=s=1280x202:mode=line[bottom]; \
 [left][right]hstack[top]; \
 [top][bottom]vstack,drawtext=fontfile=/Library/Fonts/Arial.ttf:fontcolor=white:x=10:y=10:text='\"Song Title\" by Artist'[out]" \
-map "[out]" -map 0:a -c:v libx264 -preset fast -crf 18 -c:a copy output.mkv
```

- input.mp3 can also be .wav or maybe other formats as well
- output must be .mkv
- `-preset fast` can be exchanged in:
- `ultrafast`
- `superfast`
- `veryfast`
- `faster`
- `fast`
- `medium` – default preset
- `slow`
- `slower`
- `veryslow`





### Audio Waveform Video from Audio File

```
ffmpeg -i input.wav -filter_complex \
"[0:a]showwaves=s=1280x720:mode=line:rate=25,format=yuv420p[v]" \
-map "[v]" -map 0:a output.mp4
```



```
ffplay -f lavfi "amovie=input.m4a, asplit [a][out1]; [a] showwaves [out0]"
```



### Audio Spectrum Video from Audio File

```
ffmpeg -i input.wav -filter_complex \
"[0:a]showspectrum=s=1280x720:color=rainbow:start=0:stop=0:legend=enabled,format=yuv420p[v]" \
-map "[v]" -map 0:a output.mp4

```

some Options for ```showspectrum```

- `slide=replace` or `=scroll` `=fullframe` `=rscroll` (How spectrum slides along the window)
- `mode=combined` or `=separate`

- `color=channel` or `=rainbow` `=magma` `=green` (different color schemes)
- `scale=sqrt` or `=lin` `=log` 
- `orientation=vertical` or `=horizontal`
- `start=0 ` set start Frequency for Display
- `stop=0` set stop frequency for display
- `legend=disabled` or `=enabled`  (show legend in video)

```
ffplay -f lavfi "amovie=input.wav, asplit [a][out1]; \
[a] showspectrum=mode=separate:color=intensity:slide=1:scale=cbrt [out0]"
```



### Audio-Spectrum Picture from Audio File

```
ffmpeg -i input.wav -lavfi showspectrumpic=s=hd720 output.jpg
```

- input can also be other files







### Audio Vector Scope Video from Audio File

```
ffmpeg -i input.wav -filter_complex \
"[0:a]avectorscope=s=1280x720:r=25:draw=dot:mode=lissajous,format=yuv420p[v]" \
-map "[v]" -map 0:a output.mp4
```

some Options for ```vectorscope```

- ```s=1024X768``` any size for video output
- ```draw=dot``` or  ```line```
- ```mode=lissajous``` or ```polar```
- ```scale=lin``` or ```log``` (amplitude scale of audio samples)
- ```rc=40:gc=160:bc=80:ac=255``` (red, green, blue, alpha contrast DEFAULTS. Range is [0, 255])
- ```rf=15:gf=10:bf=5:af=5``` (red, green, blue, alpha fade DEFAULTS. Range is [0,255])





```
ffplay -f lavfi "amovie=input.wav, asplit [a][out1]; \
[a] avectorscope=zoom=1.3:rc=2:gc=200:bc=10:rf=1:gf=8:bf=7 [out0]"
```





### Audio Frequency-Spectrum Video from Audio File

```
ffmpeg -i input.wav -filter_complex \
"[0:a]showfreqs=mode=line:fscale=log:win_size=1024:win_func=hanning,format=yuv420p[v]" \
-map "[v]" -map 0:a output.mp4
```

some Options for ```showfreqs``` 

- ```mode=bar``` Options are `=line` `=dot`
- `ascale=log` or `=lin` (amplitude scale)
- `fscale=lin`or `=log` (frequency scale)
- `win_size=2048` Range is [16, 65536] 

- `win_func=hanning` or  `=rect` `=sine` `=gauss``
- `cmode=combined` or `=seperated` (channel display mode)
- `colors=Azure|Lime` (set colors. Check `ffmpeg -colors` for available color names)


```
ffplay -f lavfi "amovie=input.mp4, asplit [a][out1]; [a]  showfreqs=mode=line:fscale=log [out0]"
```

### Video from Spatial Relationship between two Audio Channels

```
ffmpeg -i input.wav -filter_complex \
"[0:a]showspatial=s=1024x768:win_size=2048,format=yuv420p[v]" \
-map "[v]" -map 0:a output.mp4
```

### Audio Volume Video from Audio File

```
ffmpeg -i input.wav -filter_complex \
"[0:a]showvolume=f=1:b=4:w=720:h=68,format=yuv420p[vid]" \
-map "[vid]" -map 0:a output.mp4
```


```
ffplay -f lavfi "amovie=input.mka, asplit [a][out1]; [a] showvolume=f=255:b=4:w=720:h=68 [out0]"
```


### Video to Analyse Musical Notes from Audio File

```
ffmpeg -i input.wav -filter_complex \
"[0:a]showcqt,format=yuv420p[v]" \
-map "[v]" -map 0:a output.mp4
```


```
ffplay -f lavfi "amovie=input.mp4, asplit [a][out1]; [a] showcqt [out0]"
```


### Volume Histogram Video from Audio File

```
ffmpeg -i input.wav -filter_complex \
"[0:a]ahistogram,format=yuv420p[v]" \
-map "[v]" -map 0:a output.mp4
```

```
ffplay -f lavfi "amovie=input.flac, asplit [a][out1]; [a] ahistogram [out0]"
```


### VR 180 Format für Google Video 

Verwendeter Code 

```
ffmpeg -i /SOURCE-FILELOCATION/SOURCE-FILENAME -c:v libx264 -preset fast -crf 18 -x264-params mvrange=511 -maxrate 120M -bufsize 25M -vf "scale=5120x2560:in_range=full:out_range=full:in_color_matrix=bt709:out_color_matrix=bt709" -pix_fmt yuv420p -c:a aac -b:a 160k -r 29.97 -movflags faststart /TARGET-FILELOCATION/TARGET-FILENAME 
```

von google vorgeschlagenes Script:

```
ffmpeg -i /SOURCE-FILELOCATION/SOURCE-FILENAME -c:v libx264 -preset fast -crf 18 -x264-params mvrange=511 -maxrate 120M -bufsize 25M -vf "scale=5120x2560:out_range=full:out_color_matrix=bt709" -pix_fmt yuv420p -c:a aac -b:a 160k -r 30 -movflags faststart /TARGET-FILELOCATION/TARGET-FILENAME 
```


### Convert Video with Audio for youtube Upload

Video conversion with audio stream copying:

```
ffmpeg -i input.avi -c:v libx264 -preset slow -crf 18 -c:a copy -pix_fmt yuv420p output.mkv
```

- add for youtube faststart: `-movflags +faststart` 


Video conversion with Audio reencoding (AAC):

```
ffmpeg -i input.mov -c:v libx264 -preset slow -crf 18 -c:a aac -b:a 192k -pix_fmt yuv420p output.mkv
```


Create Video from still image and audio file:

```
ffmpeg -loop 1 -framerate 2 -i input.png -i audio.m4a -c:v libx264 -preset medium -tune stillimage -crf 18 -c:a copy -shortest -pix_fmt yuv420p output.mkv
```



