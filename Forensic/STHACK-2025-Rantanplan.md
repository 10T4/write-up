# Forensic - Rantanplan's twitch propaganda

<div align="center">
  <img src="https://github.com/10T4/write-up/blob/main/images/Image20.png" alt="sthack">
</div>

So in this challenge we have a file 'capture.pcap', this extension is used to capture network trames and analyse these trames, the first step upload your file in WireShark:

<div align="center">
  <img src="https://github.com/10T4/write-up/blob/main/images/Image21.png" alt="sthack">
</div>

We found lot of UDP packets, we can read the packets now, we must use a tool, to decrypt this we can use ffmpeg:
```
ffmpeg -i capture.pcap -c copy output.mp4
```

after the conversion we can read the flag on the .MP4:
FLAG: STHACK{00f_W@F_tH0ugh7_U_c0UlD_UnD3rs74nD?}
