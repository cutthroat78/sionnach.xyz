---
title: "Pulseaudio"
tags: [
  "computing",
  "FOSS",
  "linux",
  "CLI",
  "audio",
  "sound",
]
---

## Tutorials / How-Tos

### Make A Microphone Mono / Make A Virtual Input

#### Make A Virtual Input Source That Is Mono Using Both Channels Of An (Hardware) Input That Is Stereo

I have tested this and it works for me

1. ```pacmd set-default-source 0```
  - From what I can tell, you can't make a remap a source that is set to default
2. ```pacmd load-module module-remap-source source_name=mono master={INSERT SOURCE NAME HERE} master_channel_map=front-left,front-right channel_map=mono,mono```
  - This will create a virtual mono input that uses both channels for the one channel from the stereo input that we specified

#### Make A Virtual Input Source That Is Mono Using The Left or Right Channel Of An (Hardware) Input That Is Stereo

I have tested this and I couldn't get it working for me

1. ```pacmd set-default-source 0```
  - From what I can tell, you can't make a remap a source that is set to default
2. To map the left channel: ```pacmd load-module module-remap-source master={INSERT SOURCE NAME HERE} channels=1 master_channel_map=front-left channel_map=mono```
  - To map the right channel: ```pacmd load-module module-remap-source master={INSERT SOURCE NAME HERE} channels=1 master_channel_map=front-right channel_map=mono```
  - This will create a virtual mono input that uses the left or right channel (depending on which one you specified) for the one channel from the stereo input that we specified

## Links

- [PulseAudio - ArchWiki](https://wiki.archlinux.org/title/PulseAudio)
  - [PulseAudio/Examples - ArchWiki](https://wiki.archlinux.org/title/PulseAudio/Examples)
