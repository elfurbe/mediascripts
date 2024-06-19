# mediascripts
My assorted media manipulation scripts

## What's in here

  - addstats
    - You'd think remembering "--add-track-statistics-tags" wouldn't be that hard but I got so tired of forgetting that I made a wrapper
  - audiovise
    - Just transcodes audio, does not touch video
    - It's a 30 Rock joke
  - av1meup2
    - ~~A poorly maintained mod of hevcmeup2 for av1 instead of hevc~~ A very well maintained analog to hevcmeup2 for SVT-AV1 and the script I now use most often by far
    - Protip: AV1 is ~~a lie concocted by big youtube to trick you into not using HEVC. Open your eyes, sheeple.~~ good, actually.
  - cropfinder
    - Tells you what ffmpeg thinks the crop should be for an input file which is more useful than you think, probably.
  - framedumper
    - Dumps specific frames as PNGs. Uses math and magnets to dump one frame per even time interval based on the number of frames you asked for.
    - Useful for doing comparisons between two versions of a file if you don't want to cherry pick specific frames because you're lazy
  - gifify
    - Makes a gif from a video file based on timecodes
    - It's simple and dumb, you have to edit the framerate and the output width by hand because I hate you and myself
  - hevcmeup2
    - This ~~is~~ was the franchise. An extremely overcomplicated script to cajole ffmpeg into transcoding media files to HEVC via libx265
    - More on this [below](#hevcmeup2)
  - stripcovers
    - Sometimes there are covers in mkv files and while some people think that's cool, they're wrong, it's dumb
    - They're specifically dumb because it's difficult to reliably keep ffmpeg from trying to encode them as video which really chaps my ass
    - This removes all covers with extreme prejudice
  - striptags
    - Sometimes there are tags in mkv files with data you'd rather not have in them
    - Specifically global file title metadata confuses the Kodi media scanner
    - Some tools/people like to stuff notes into the video track title, they can die in a fire
  - thecleaner
    - I'm a 'Murican, a high percentage of my media is from 'Murica in native 'Murican and I don't need other language audio tracks or subtitles
    - This _very specifically_ by _default_ keeps _only_ english language tagged audio and subtitle tracks
    - It can also optionally change the title tag of the primary audio track
      - I specifically transcode most audio to opus cause it's the best so that's mostly what it's good at making names for
    - If you want specific tracks or different languages to be kept, look at the script help, it can do that
  - toolchecker
    - So. You're probably asking yourself why everything requires the toolchecker.
      1. So tired of trying to use these scripts in places and getting rando errors cause the fuckin' tools aren't installed, now I get intentional errors
      2. So tired of running these scripts inside a screen session or some kind of background subshell situation and it not having my god damned environment and thus missing my path and then getting rando errors, now I get very intentional errors
      3. Literally _any excuse_ *EVER* to make use of variable variables for confusion value
  - tracktitleeditor
    - You can do this with mkvpropedit by hand like a big bad matroska gangster but I can never remember the damn syntax
    - It only works on mkv files, natch.
    - I did my very best to take whatever text you type and stuff it into the metadata tag but if you try to paste some emoji nonsense in here it's probably not gonna work and I do not give a single shit.
  - propedit
    - This is the evolution of tracktitleeditor
    - I hate mkvpropedit syntax so much, I never want to remember it again, so this tool does the things I need to do most often in a TUI manner
    - It only works on mkv files, natch.
  - visionary 
    - This is for dealing with Dolby Vision (and HDR10+) files
    - See [below](#visionary)
  - vmafer
    - I wanted to be able to run [vmaf](https://github.com/Netflix/vmaf) comparisons for "am I doing this right" reasons, but remembering and typing the command was super tedious. This used to be a dumb script but now it's less dumb, it has help and options and everything.

## av1meup2/hevcmeup2
### What even is this
av1meup2/hevcmeup2 is the shiniest turd jewel in this shiny turd crown. I transcode a lot of video files for space saving reasons, specifically into yuv420p10le (yes even SDR files, do not at me). I got very, very tired of hand-massaging ffmpeg command lines so I built my own wrapper to solve my specific problems. It started out quite simple, back before it had a `2` in the name. Now it's a complete bloody nightmare of bash nonsense, just absolute fucking chaos of bashisms. If you can read this whole script and not need to look up a single convoluted bash tricknique, your service to the secret fire has clearly been both lengthy and arduous.
### How do I use it
Look, and I'm being real here. Probably don't. But if you decide to use it, it has help, specifically this
```bash
Usage: ${0##*/} -i <INPUT FILE> -o <OUTPUT FILE>

-h|--help:     show this help message
-i|--input:    file name for input
-o|--output:   file name for output
--start:       start time for encode (default: unset)
--duration:    duration of encode (default: unset)
--crf:         crf value (default varies with input res, 1080:18, 720: 22, 480: 26)
--melton:      use Don Melton's trickeration for bitrate control
--mbitrate:    if using --melton, use this bitrate cap
--preset:      x265 preset value (default: medium)
--tune:        x265 tune value (default: unset)
--crop:        enable crop detection (default: no)
--debug:       output commands to be run, run nothing
--oprofile:    output profile [1080, 720, 480]
--vf:          custom videofilter input
--sdr:         best effort 2020->709 color fix
--yadif:       deinterlace with yadif
--yadifmode:   set yadif to specific mode (default: 1, send frame per field)
--mcdeint:     deinterlace with yadif and mcdeint (DEPRECATED)
--nnedi:       use the power of ai to deinterlace
--ivtc:        Inverse telecine, aka pullup (assumes NTSC DVD 29.97->23.976)
--pullup:      Alias for --ivtc
--fieldmatch:  Inverse telecine a different way using fieldmatch filter
--bitdepth:    Bitdepth for pixel format (8, 10 or 12, default: 10)
--audio:       transcode audio tracks to selected codec [flac, opus, aac]
--square:      convert anamorphic to square pixels which make sense
```

(seriously, if `${0##*/}` makes sense to you, honest to god you can lay your burden down and go into the west and remain `<your name here>`, you've done enough)

When I run it, which is literally constantly, this is my CPU time, this is typically what it looks like:
```bash
$ hevcmeup2 -i input_file.mkv -o output_file.mkv --crop --audio opus
$ av1meup2 -i input_file.mkv -o output_file.mkv --crop --audio opus
```
See how easy it is? What, you're thinking "Gosh there must be a container ship full of defaults and nonsense to make that work"? Well you, friend, win the prize, that is exactly right, it's for me and it does what I want with minimal effort. Most every default is listed above in the usage, they're pretty self-explanatory.

### Confusing options explained
  - `--melton` and `--mbitrate`
    - is an implementation of a tricksy rate control hack as described here: ![video_transcode](https://github.com/donmelton/video_transcoding#how-my-simple-and-special-ratecontrol-systems-work) If you need to hit an ABR target for specific reasons, it's probably better than ABR, but it's still worse than surfing CRFs to find one that makes an output file the right size. I'm not sayin' it's easy or fast to surf crfs but for two files of the same average bitrate, one using actual crf rate control will look better every single time. But I wanted to try it and see, so I tried it and saw and now it's here forever.
  - `--oprofile`
    - this was an idea I had that was ostensibly going to let me dial in defaults for specific output profiles. I started with profiles based on video dimensions, that's all I ever made, and all they actually do is set a scale filter and change the CRF to be appropriate for that video dimension. So, if you've got a 1080p file and you want it to be a 720p output, `--oprofile 720` will set up the scale and lower the CRF automatically.
  - `--square`
    - Boy howdy do anamorphic pixels blow. I'm not gonna explain them, but they make everything worse, they're a tired holdover from the analog era and I wish they were never made part of the DVD standard. But sadly they were, and sometimes you're trying to take content from a DVD that has dumbass anamorphic pixels and you'd really just like it to have a normal square pixel resolution instead, great news! This does that.
  - `--mcdeint`
    - Prior to ffmpeg5, mcdeint (motion compensating deinterlacer) was, in my opinion, the hands-down best quality de-interlacer in ffmpeg. It was unbelievably, unbearably slow, but the output was superb. It's been deprecated in ffmpeg5, so now, when I'm feeling very picky about deinterlacing, I use `--nnedi` which is also unbelievably, unbearably slow, but it uses the power of _machine learning_ so you know it's good. Also it actually is good, it makes very good output, maybe even better than mcdeint. Worth noting, `--nnedi` requires a model file that it does not detect through sorcery, so if you want to use it you will need to find the model file and modify ![this line](https://github.com/elfurbe/mediascripts/blob/main/hevcmeup2#L475) to point to the model file.
  - `--audio`
    - This will also transcode the audio tracks. I like opus, I think it's the least doofy lossy codec, it also has the benefit of being unencumbered, so I transcode a lot of lossless audio to opus for space reasons. I _do not_ transcode _lossy_ audio, if I can help it. Also I try not to transcode between lossless formats cause why fucking bother. This section of the script tries its best to follow those rules. It chooses the opus bitrate based on the channel layout presented, if it understands the channel layout, or by the channel count if it doesn't. Based on the opus guidelines for bitrate I'm wildly exceeding the recommended bitrates at every channel count but we're talking about kilobits here and who fucking cares, it's still smaller than OG DTS. I would not call what this option does entirely intuitive, so if it does something to your audio you didn't expect, rest assured I probably did expect it so that's why it did that.

## visionary
### What even is this
Dolby Vision, and Samsung's HDR10+, are _dynamic_ methods of describing the dymanic range of a video file. Normal HDR (HDR10, HLG, etc) uses a static HDR profile that is constant through the entire stream. Dolby Vision and HDR10+ attach metadata elements (via different methods, obviously) at a frame level to the video stream to allow a mastering engineer to have more complete control over how the dynamic range is applied. This _ostensibly_ allows for a superior experience, in practice, who could say, but the formats exist and the video streams using them exist and I want to be able to compress them, so I made a script that does it for me, this is that script.

I entertained simply throwing this dynamic HDR info away. Virtually all Dolby Vision files (excepting profile 5, more on this later) have a base video layer that includes bog standard normal HDR10 or HLG info which is, you know, pretty good. AV1 can encode static HDR no problem, so I could just throw away the dynamic info, cook the base layer into a surprisingly tiny AV1 stream and robert would be my proverbial mother's brother. But, me being me, I could not commit myself to doing that, so I wrote 400 lines of bash code instead like the brain-poisoned sysadmin I am.

By default this script transcodes to HEVC which may seem counterintuitive since most Dolby Vision/HDR10+/HDR-period content is already in HEVC. In the case of UHD BluRay discs, nothing about them is oriented around efficiency, they're oriented around providing a bitstream with sufficiently controlled parameters that UHD BluRay players less powerful than a Casio calculator can leverage their ten year old fixed-function ICs to make a moving picture happen. That's perfectly reasonable and all but we do not need to store all them dumb ass bits on a hard drive to play with a whole-ass computer, so we can use all the good tricks to make the bitrate much lower without giving up any practical amount of visual fidelity. In the case of streaming video, the constraints are entirely different. They need to build streams that will play on old mid-range Android phones over LTE, so while they are often much more bitrate efficient, they are also often still prevented from leveraging the full power of optimization available with an encoder like x265 to ensure older fixed-function decoders can produce the talkies, so there is often still value to be found in recompressing those streams and allowing x265 to Do What It Do.

"What about AV1?" I hear you asking, "Why not use the new hot shit?". Well. Funny you should mention that. Today, if and only if you build the [SVT-AV1-PSY](https://github.com/gianni-rosato/svt-av1-psy) research fork of SVT-AV1, there is a method for injecting RPU data into an AV1 bitstream. ffmpeg doesn't pass the option, and may never based on things I've read, so I've implemented AV1 support with a pipe to `SvtAv1EncApp` like an animal but it _does_ appear to produce a file that is recognized by MediaInfo as a 'dav1.10' Dolby Vision stream, so I've got that going for me. I've also been reading about some kind of method leveraging libplacebo in ffmpeg to do the RPU injection but the level of gizmosity appears, at this time, to be untenable. Anyway just set `--av1` and Things will Happen, but don't expect it to be a media file any player you currently possess in TYOOL 2024 will play back correctly cause Dolby Vision in AV1 does not really exist in wide use by any commercial provider for consumer playback.

### How do I use it

It's actually very easy to use, and much like all the other scripts, unless you're pretty sure you've thought about this harder than me, probably just use the defaults. The only _required_ option is an input file, specified with `-i` or `--input`. The script will use that input name to gin up an output name and clean up all the temp files when it's done and you'll have a shiny new transocded version of your input right there in the folder.

### Extremely nerd nonsense, turn back while you still can

So, if you're not familiar with Dolby Vision at a technical level (why would you be, I do not recommend it), there is some vocabulary you'll need to accept if you want to understand what's happening. You _do not_ need to understand what's happening, this script works, you can stop reading anytime, but if you want to, you should know some things. This is the tl;dr version of a PDF document you can find on [this page](https://professionalsupport.dolby.com/s/article/What-is-Dolby-Vision-Profile?language=en_US) called "Dolby Vision Profiles and Levels". Beware: that which is learned cannot be un-learned.
 - Layers:  There are three elements used in two different combinations to represent a video stream masterd with Dolby Vision.
   - Base Layer or "BL": Primary video data is stored in a standard video stream called the "Base Layer" or "BL". This video stream will usually (profile 7, 8, 10) have static HDR metadata characteristics, e.g. HDR10 or HLG. 
   - Enhancement Layer or "EL": An optional secondary video stream used as a further masking element to extend the dynamc range to an effective 12 bits. This is only used by profile 7 and only on physical media, e.g. UHD BluRays.
   - Reference Picture Units or "RPU": RPUs are the per-frame HDR metadata attached throughout the bitstream. Each frame can have its own RPU packet.
 - Profile: Defined by Dolby and identified by a number (4,5,7,8,9,10,20) which specifies formats and features for the stream. The most common profiles you will see are 5, 7 and 8.
   - Profile 5 ,"dvhe.05": HEVC IPT-PQ-C colorspace BL+RPU format used exclusively by streaming video services. Not actually but _practically_ deprecated by Profile 8. For profile 5, RPU data is attached to the BL as there is no EL. Profile 5 is slowly disappearing from the earth so I have not bothered to support it at all.
   - Profile 7, "dvhe.07": HEVC standard colorspace BL+EL+RPU format. Used exclusively on UHD BluRay streams, not supported for streaming applications. Not _all_ UHD BluRay streams are profile 7, but all profile 7 streams came from UHD BluRays, if you're a consumer. For profile 7, RPU data is attached to the EL and not to the BL.
   - Profile 8, "dvhe.08": HEVC standard colorspace BL+RPU format used primarily by streaming video providers and in some cases on UHD BluRay or downloadable media. For profile 8, RPU data is attached to the BL as there is no EL.
   - Profile 10, "dav1.10": AV1 standard colorspace BL+RPU format used primarily by literally no one, it doesn't exist in the real world. For profile 10, RPU data is attached to the BL as there is no EL.
 - Level: Defined by Dolby, _also_ identified by a number (thanks Dolby), specifies video dimensions, frame rates and bitrates. Why wouldn't you simply determine those things by inspecting the video stream, who could say. For UHD/4K streams, you will generally speaking only see a few levels
   - 06: 4K 24p film content
   - 07: 4K 30p television content
   - 09: 4k 60p television content
 - You will most commonly see the profiles and levels annotated like this: "dvhe.08.06". This means: Dolby Vision, HEVC, Profile 08 (BL+RPU) Level 06 (24p).

So let's talk about the EL. Dolby, in their wish to find new and exciting rent-seeking opportunities, had what I assume is a meeting where some version of this conversation happend.
 - Dolby Marketing and/or Finance person: "We must find a new way to extort pennies from device makers. We've hung our hats on audio for a long time but we're running out of new ways to pretend it's better than before. Ideas, eggheads!"
 - Dolby Engineer with No Soul 1: "What if we could make "regular HDR" seem bad in a math-y way? That might enable us to sell licenses to use a proprietary HDR format!"
 - Dolby Marketing: "Hey, that DOES sound lucrative. But what could we say about it, these sheep literally just got 10-bit HDR, it's the biggest revolution in video since HD."
 - Dolby Engineer with No Soul 2: "What if we suggested that 10 bits of static dynamic range, while maybe good enough for plebian blind cave trolls, was in no way adequate for the discerning, smooth-brained videophile? What if we made up a new format that...I don't know, provided the HDR metadata _dynamically_?"
 - Dolby Marketing: "Hmm, that sounds pretty good but it's too technical to sell. What numbers can we make bigger? They love bigger numbers, they'll always pay for bigger numbers."
 - Dolby Engineer 2: "We could....say it has more precision for HDR? Like, I don't know, like 12 bits! And...uh...and we'll say it can be mastered to some absurd brightness level...I don't know, like 10000 nits peak! That's a _huge_ number!"
 - Dolby Engineer 1: "I like where your head's at, friend, those are good numbers to make up. But, 10 bits of pixel precision is all a 10-bit colorspace can carry! Going to more bits per pixel requires support in fixed-function decode hardware that does not currently, and may never in practical terms, exist. We can't ship a format that no one can play!"
 - Dolby Engineer 2: "Well, we've got good support for 10-bit native decoding in the marketplace, right? So, what if instead of decoding _one_ 10-bit video stream during playback, you decoded _two_ video streams during playback and that _second_ stream could be used to "enhance" the rendering of the first stream?"
 - Dolby Engineer 1: "You fiend, you madman, you devious genius! And since it's basically just a masking layer, we can make it a fractional scale version of the primary stream and still get enough useful value out of it for mapping pixel light levels! We can limit that layer to 1920x1080 as a maximum resolution and cap the bitrate at 15Mbit since it's 1080p and HEVC, that'll keep the size under control and still leave us most of 85Mbit to use for the primary 4k base layer."
 - Dolby Engineer 2: "What briliant fucking boffins we are! But how will we deliver that _not_ on a physical disc?"
 - Dolby Engineer 1: "I say we just don't! I say we make up a brand name for it and use that brand name everywhere, regardless of whether they provide the same thing. Like...Dolby....VISION, yes. Dolby Vision, that's great. And I don't know, just add some extra numbers to streaming video instead, just like some hints about how to use the light per scene or per frame or something. As long as we have a way to make some indicator light on some electronic box turn on, we're good to go."
 - Dolby Engineer 2: "Oh that's awful, we will make so much money from that. But isn't authoring these files going to be kind of a nightmare?"
 - Dolby Engineer 1: "You're thinking about this all wrong, buddy. We just made it impossible to master this format in any editing suite that exists on Earth! We can _also_ start a program to validate hardware and editing facilities so they have to get our stamp of approval to even know they're doing it right!"
 - Dolby Marketing: "_tearing up_ I've heard enough, you're both examples of the finest that Dolby engineering has to offer. Now get out of here and go make papa a new cash cow!"

Samsung, being Samsung, did not want to pay Dolby for their license, so they unilaterally extended HDR10 themselves and made up HDR10+ which sounds like a standard but is not, really, a standard. It is, functionally speaking, analogous to a MEL or BL+RPU Dolby Vision stream. The data packets are not called RPUs, they're not stored the same way, but the functional value is the same and there's no EL concept, so they're much less annoying to deal with. As such I'm not going to explain it at all.

Since native support for dynamic metadata barely exists in practical terms in ffmpeg, you need some other tools to deal with it in a useful way. The current gold standard tools are [dovi_tool](https://github.com/quietvoid/dovi_tool) and [hdr10plus_tool](https://github.com/quietvoid/hdr10plus_tool), both written and maintained by the same (I presume) video wizard wreathed in rgb flames. They do the hard work of demuxing and remuxing the dynamic data into the base hevc video stream. In the case of dovi_tool, it also handles converting RPU data between different profiles (07->08, for our use case here).

Someone may now be asking themselves, and by proxy me, "Wait, but. Why are you converting the RPU profile data from the FEL-compatibile format to the MEL-compatible format....wait are you _throwing away_ the _ENHANCEMENT LAYER_?!" Yes. I am. Look, here's the scoop and I'm gonna tell ya, the EL is nonsense. It is. I mean I'm sure someone has maths that make it seem important, but there's no practical value for 12 bits of precision for _brightness_. With a mere 10-bit unsigned integer you can represent 1024 different states. No human eye can in real-time discern the difference between even that many states of brightness. 12 bits of precision makes that number 4096, which is categorically absurd. 10-bit _color spaces_ are good, they help us eliminate banding and other unfortunate sorts of artifacts. Maybe someday 12-bit color spaces will be a real thing and that will also potentially be even better, but for the purposes of _dynamic range_ 10 bits is more than adequate. Arguably 8 bits was probably adequate, but whatever, 10. Fine. We have 10 bits, we can do that without _two video streams_ fine. We'll do it. Making a decoder decode two videos to play back one video so you can have ~4x the number of possible states for how eye-searing specular highlights are is marketing hoodwinkery and you're wrong if you disagree with me.

Ok, but also ranting aside: I tried to preserve it. I don't throw away data, as a rule, I did not go into working up a Dolby Vision transcoding scheme to throw away data, but keeping the EL is, in my opinion, so difficult to accomplish with freely available tools that I gave up trying. Since you're 5000 words deep on this, the tl;dr of why is that you essentially need to co-encode the BL and the EL so every frame is of the same frame type, meaning, as an example, if there's a keyframe in the BL there has to be a keyframe in the EL at the same spot, otherwise the decode sauce doesn't work. Figuring out how to make ffmpeg do that was too complicated for me, I'm just a little guy. Am I saying it's not _possible_ to do in ffmpeg, I learned a long time ago that very few media-related tasks are _impossible_ with ffmpeg, but I am saying it's sufficiently prohibitively difficult that even I, that made all these dumb tools, will not spend any more time on it. If YOU, dear reader, know how to make ffmpeg do this thing, take in two unique video streams at different resolutions, and match the frame cadence of the output streams precisely while passing through an encoder, file an issue or something, I'll read it.

One thing you may notice is that this script can _optionally_ use the Dolby Vision Professional Verification Toolkit to "verify" the output file as a valid Dolby Vision file (BL+RPU, usually 08.06). It is _exceedingly rare_ to have zero errors or warnings, even on the original source files, so I would not necessarily recommend you get this toolkit and use this feature regularly. I baked it in while I was developing the script to make sure I was producing output that passed some basic checks, but you have to learn a great deal about what you can ignore to find the output meaningful and I would not recommend anyone look at enough output from it to learn that. It is truly useless esoterica, raw bitstreams straight off UHD blurays _do not pass without errors_ so just let that wash over you.

A weird thing worth noting is that this script leverages a fifo/named pipe to avoid having a _third_ yes third copy of the video bitstream lying around on disk while it works. 4K bluray video streams in particular are obscenely large, so only needing ~2x the size of the original file for working set space instead of ~3x was worth a little faffery. It is also worth noting if mkvtoolnix would just let me send an extract output to stdout I wouldn't have to work like this, but no no, has to have a file descriptor.
