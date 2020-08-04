# youtube-dl snippets

## Download best available quality in mp4

```text
youtube-dl -f 'bestvideo[ext=mp4]+bestaudio[ext=m4a]/mp4 https://youtu.be/dQw4w9WgXcQ
```

## Rip mp3 from youtube video

```text
youtube-dl -x --audio-format mp3 https://youtu.be/dQw4w9WgXcQ
```

