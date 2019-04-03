# FFMpeg, m3u8, html5 screencasting
In this tutorial, I walk through how to stream a screencast to people's browsers in a low latency manor.  

###### Easy Steps
  
  - Stream your desktop with ffmpeg
  - Write an html file pointing to the stream file
  - Server the html file via a web server
  - Connect to the stream from a browser
  

## Stream your desktop with ffmpeg

In this step we will be running the ffmpeg program which will create a .m3u8 file and also serve HLS video to browsers that request it (upon follow the link to the .m3u8).  I won't cover audio, though that's easy to do, reason being, the audio is unidirectional.  A class shouldn't be tought unidirectionally so a bidirectional audio system such as Mumble will be used to cover my audio needs.  

For linux, just do

```
ffmpeg -video_size 1600x900 -framerate 25 -f x11grab -i :0.0+0,0 output.mp4
```

The key takeaway is the `-f x11grab` which tells linux to hand over the screen

ref: https://trac.ffmpeg.org/wiki/Capture/Desktop


