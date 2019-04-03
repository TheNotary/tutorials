

#### Overview

- DL the `.deb` to install Unity on a debian based system
- Install the .deb
- Run the newly creaty `Unity` app from your application menu
- As you open Unity, you'll need to create a unity account
- 


Find the download for Unity's Linux beta-test .deb package here 

http://forum.unity3d.com/threads/unity-on-linux-release-notes-and-known-issues.350256/

You'll need to install monodevelop too

ref:  http://www.mono-project.com/docs/getting-started/install/linux/#usage

```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 \
  --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF
echo "deb http://download.mono-project.com/repo/debian wheezy main" | \
  sudo tee /etc/apt/sources.list.d/mono-xamarin.list
sudo apt-get update
sudo apt-get install mono-devel
sudo apt-get install mono-complete
sudo apt-get install monodevelop
```

It was picky about me starting with `mono-devel` so be warned...

There's one last hack.  To get unity to start monodevelop properly, you need to modify the top of one of your files to be as below.

(/opt/Unity/MonoDevelop/bin/monodevelop)
```
#!/usr/bin/env bash

/usr/bin/monodevelop "$@"
exit
.
.
.
```



