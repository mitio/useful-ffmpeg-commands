# Useful FFmpeg commands

This repo contains examples for useful FFmpeg commands extracted from practical needs.

## Intro

If you don't know what [FFmpeg](https://www.ffmpeg.org/) is, this page will probably be of little use to you.

FFmpeg is a free, open-source software and it's the Swiss Army knife of video- and audio-related processing. It can be used to do an unbelievable range of things and it's being utilized by virtually anyone who's doing any form of video processing. It comes as a command-line tool, but many programs ship with a built-in version of FFmpeg in them to be able to process multimedia files. FFmpeg will run on Linux, Windows, OS X and other platforms.

This is my personal collection of FFmpeg commands, recorded here for myself to come back to and check when I need something. I'm making it public because someone else might find some pieces of information here useful.

**License:** Feel free to use the information here in any way you wish, no
attribution is needed, but I take no responsibility for the results of your
actions.

Some of the commands have been taken verbatim from other sources, others have been heavily modified since. You can find some more commands here:

- <https://trac.ffmpeg.org/wiki/> – the official FFmpeg wiki contains some very good examples.
- <http://www.labnol.org/internet/useful-ffmpeg-commands/28490/>

Contributions are welcome.

## How to read this

Just search the page for the example you need.

The examples below have little to no context and usually do not explain all the different options used. That's what the documentation is for – you can [read it online here in this huge one-page HTML document](https://www.ffmpeg.org/ffmpeg-all.html) or just do `man ffmpeg-all` in your terminal.

The order of the options in an FFmpeg command has some significance, but generally you can use any order you want.

## Extract audio from a video file (e.g. an YouTube clip)

    ffmpeg -i skrillex.mp4 -ab 256k skrillex.mp3

## Slice a 1-minute portion of an audio file starting at 2m30s, convert it to MP3 and add audio fade in and fade out effects

    ffmpeg -i INPUT.mp4 -c:a libmp3lame -b:a 128k -ss 00:02:30 -to 00:03:30 -af 'afade=t=in:st=150:d=3,afade=t=out:st=207:d=3' OUTPUT.mp3

## Slice the first 5 min of a video

    ffmpeg -i 038-Speed-Limit-260.avi -c copy -t 00:05:00 Simone_Origone_Speed_Limit_260.avi

## Slice 5 min starting from 12:30 from a video

    ffmpeg -i 038-Speed-Limit-260.avi -c copy -ss 00:12:30 -t 00:05:00 Simone_Origone_Speed_Limit_260.avi

## Create x246 mp4 videos suitable for online streaming

The x264 codec provides a very good compression ratio and a good output quality. The options below are suitable for online embedding/streaming of a video. You will probably need an `webm` version as well as some browsers can't play the mp4 one (see below). Adjust the size and bitrates accordingly.

    ffmpeg -y -i in.suffix -filter:v scale=640:360,setsar=1/1 -pix_fmt yuv420p -c:v libx264 -preset:v slow -profile:v baseline -x264opts level=3.0:ref=1 -b:v 700k -r:v 25/1 -force_fps -movflags +faststart -c:a libfaac -b:a 80k -pass x out.mp4

## Create webm videos suitable for online streaming (no audio)

Suitable for online streaming. You will probably need an mp4 version as well (see above.) Adjust the bitrates and resolution accordingly.

NB: This example assumes the input has no sound.

    ffmpeg -i bff-teaser-part-1-1000kbps-no-sound.mp4 -filter:v scale=640:360,setsar=1/1 -pix_fmt yuv420p -vpre libvpx-720p -b:v 800k -r:v 25/1 -force_fps -c:a none -pass 2 bff-teaser-part-1-1000kbps-no-sound.webm

## Remove sound from webm videos

    ffmpeg -i bff-teaser-part-1-1000kbps-no-sound.mp4 -pix_fmt yuv420p -vpre libvpx-720p -b:v 800k -r:v 25/1 -force_fps -c:a none -pass 2 bff-teaser-part-1-1000kbps-no-sound.webm

## Replace video's audio with audio from an external audio file

Based on [this StackOverflow answer](http://superuser.com/questions/800234/how-to-replace-an-audio-stream-in-a-video-file-with-multiple-audio-streams).

    ffmpeg -i VIDEO -i AUDIO -map 0:v -map 1:a -codec copy -shortest RESULT

## Concatenate files with different encodings (but the same resolution)

A more detailed explanation can be found in [this FFmpeg wiki page](https://trac.ffmpeg.org/wiki/Concatenate).

    ffmpeg -threads 2 -i INPUT1 -i INPUT2 -filter_complex '[0:0] [0:1] [1:0] [1:1] concat=n=2:v=1:a=1 [v] [a]' -map '[v]' -map '[a]' -c:v mpeg4 -preset:v slow -filter:v scale=768:576,setsar=1/1 -pix_fmt yuv420p -b:v 1900 -r:v 25 -force_fps -c:a mp3 -b:a 256 -s:a 48000 flying2.avi

## Create an mpeg4 video with mp3 sound

This is suitable for playback on a wide range of slower devices. The quality options here are relatively good.

    ffmpeg -threads 2 -i input.avi -qmin 1 -qmax 3 -c:v mpeg4 -filter:v scale=768:576,setsar=1/1 -pix_fmt yuv420p -b:v 2400k -r:v 25 -force_fps -c:a mp3 -b:a 256k -s:a 48000 encoded.avi

## Create an mpeg4 video with mp3 sound and resize to 720p

Used for processing videos for [Bansko Film Fest](http://banskofilmfest.com/en/).

    ffmpeg -i original.mp4 -threads 8 -c:v mpeg4 -c:a mp3 -vf scale=-2:720 -qmin 1 -qmax 4 encoded.avi

## Create a 5-second 720p video from an image

    ffmpeg -loop 1 -i INPUT.png -t 5 -c:v mpeg4 -vf scale=-2:720 -qmin 1 -qmax 3 OUTPUT-5s.avi

The `-2` tells FFMpeg to calculate the output width so that the original aspect ratio is preserved but also to make sure the output width is divisible by 2 as some codecs require even numbers for video width and height. [Read more about scaling here](https://ffmpeg.org/ffmpeg-filters.html#Options-1).

## Rip a DVD to an mpeg4+mp3 AVI

This assumes a PAL DVD.

To rip a DVD with FFmpeg, you just need the `*.VOB` files in the `VIDEO_TS` folder. Read more about [ripping DVDs with FFmpeg here](http://www.commandlinefu.com/commands/view/6585/dvd-ripping-with-ffmpeg).

    cat VIDEO_TS/VTS_01_*VOB | ffmpeg -i - -s 720x576 -c:v mpeg4 -c:a mp3 -qmin 1 -qmax 3 -b:a 256k -b:v 4000k -y encoded.avi

## Rendering videos with subtitles (burn-in subtitles)

This is tricky. A more detailed explanation can be found on [the FFmpeg wiki](https://trac.ffmpeg.org/wiki/HowToBurnSubtitlesIntoVideo).

The examples below assume you have subtitles in a SRT format, encoded in UTF-8.

### Add (render) subtitles to an already encoded video

    ffmpeg -i INPUT.avi -vf subtitles=SUBTITLES.srt -qmin 1 -qmax 3 OUTPUT-WITH-SUBS.avi

### Add (render) subtitles with different font, colors, etc.

Customizing the font, color, size, background color, etc. of the subtitles can be hard. The options for customizations are actually option names from the ASS subtitles spec ("Advanced SubStation Alpha"). This is what FFmpeg uses internally (via `libass`).

To see the available subtitle styling options, download this [Word doc file](http://moodub.free.fr/video/ass-specs.doc) (yes, indeed). I've also included [a PDF version of the Advanced SubStation Alpha (ASS) standard here](ass-specs.pdf).

The example below prepares a video for burning on a video DVD. The subtitles have a semi-opaque rectangular back background. The `original_size` differs from the resize option because the input is assumed to be 16:9, even though it's stored in a physical resolution of 720x576 (if this confuses you, you're not alone; Google for "SAR" and "DAR").

    ffmpeg -i INPUT.avi -target pal-dvd -s 720x576 -vf subtitles="f=SUBTITLES.srt:force_style='FontName=Arial,FontSize=22,OutlineColour=&H66000000,BorderStyle=3':original_size=720x405" -y ENCODED-WITH-SUBS.mpg

### Rip a DVD, burn-in subtitles and prepare for burning back to a DVD

The subtitle styling is as in the previous example. See more about ripping DVDs above. More info [here](http://ffmpeg.gusari.org/viewtopic.php?f=25&t=1235).

This is for a widescreen (16:9) PAL DVD input and prepares the video for burning it back to a DVD.

    cat VIDEO_TS/VTS_01_*.VOB | ffmpeg -i - -target pal-dvd -s 720x576 -vf subtitles="f=SUBTITLES.srt:force_style='FontName=Arial,FontSize=22,OutlineColour=&H66000000,BorderStyle=3':original_size=720x405" -y ENCODED-WITH-SUBS.mpg

### Render .ASS subtitles in a DVD video

Steps: **Rip the DVD** → **add subs** → **mpg file for ripping back to a DVD**

    cat VIDEO_TS/VTS_01_[123456789].VOB | ffmpeg -i - -target pal-dvd -s 720x576 -vf ass=SUBTITLES.ass -y ENCODED-WITH-SUBS.mpg

### Convert SRT subtitles to ASS

    ffmpeg -i subtitles.srt subtitles.ass

You can now edit the plaintext `.ass` file and change the styling accordingly. For documentation on the meaning of options and possible values, read above.

Here is an excerpt of an `.ass` file for subtitles with a semi-transparent rectangular black background and a white Arial text.

    [V4+ Styles]
    Format: Name, Fontname, Fontsize, PrimaryColour, SecondaryColour, OutlineColour, BackColour, Bold, Italic, Underline, StrikeOut, ScaleX, ScaleY, Spacing, Angle, BorderStyle, Outline, Shadow, Alignment, MarginL, MarginR, MarginV, Encoding
    Style: Default,Arial,28,&Hffffff,&Hffffff,&H44000000,&H0,0,0,0,0,100,100,0,0,3,2,0,2,10,10,10,0

### Render .ASS subtitles in a video

    ffmpeg -i INPUT.avi -vf ass=SUBTITLES.ass -qmin 1 -qmax 3 OUTPUT-WITH-SUBS.avi

## Output to a PAL DVD with top and bottom black bars padding and subtitles

This is adjusted for a `720:304` video.

    ffmpeg -i INPUT.avi -target pal-dvd -vf 'scale=720:304, pad=720:576:0:136, ass=subtitles.ass' -y OUTPUT-WITH-SUBS.mpg

## Prepare a video for burning on a PAL DVD

    ffmpeg -i input.avi -f:v scale=720:576 -target pal-dvd output.mpg

To burn the DVD itself, on Linux you will have to use `dvdauthor` and `genisoimage`:

    export FOLDER_NAME="YOUR_MOVIE"
    export VIDEO_FORMAT=PAL
    dvdauthor --title -o $FOLDER_NAME -f out.mpg && dvdauthor -T -o $FOLDER_NAME
    genisoimage -dvd-video -o MOVIE.iso $FOLDER_NAME

On OS X you can use [Burn](http://burn-osx.sourceforge.net/Pages/English/home.html) - it does the DVD video authoring and burning for you. It also has an embedded ffmpeg in it and can convert videos to the proper format if needed.
