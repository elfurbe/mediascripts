# mediascripts
My assorted media manipulation scripts

  - addstats
    - You'd think remembering "--add-track-statistics-tags" wouldn't be that hard but I got so tired of forgetting that I made a wrapper
  - audiovise
    - Just transcodes audio, does not touch video
    - It's a 30 Rock joke
  - av1meup
    - A poorly maintained mod of hevcmeup2 for av1 instead of hevc
  - cropfinder
    - Tells you what ffmpeg thinks the crop should be for an input file
  - framedumper
    - Dumps specific frames as PNGs. Uses math and magnets to dump one frame per even time interval based on the number of frames you asked for.
    - Useful for doing comparisons between two versions of a file if you don't want to cherry pick specific frames because you're lazy
  - gifify
    - Makes a gif from a video file based on timecodes
    - It's simple and dumb, you have to edit the framerate and the output width by hand because I hate you and myself
  - hevcmeup2
    - This is the franchise. An extremely overcomplicated script to cajole ffmpeg into transcoding media files to HEVC via libx265
    - More on this below
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
    - This _very specifically_ keeps _only_ english language tagged audio and subtitle tracks
    - It can also optionally change the title tag of the primary audio track
      - I specifically transcode most audio to opus cause it's the best so that's mostly what it's good at making names for
  - toolchecker
    - So. You're probably asking yourself why everything requires the toolchecker.
      1. So tired of trying to use these places and getting rando errors cause the fuckin' tools aren't installed, now we get intentional errors for that
      2. So tired of running these scripts inside a screen session or some kind of background subshell situation and it not having my god damned environment getting rando errors, now I get very intentional errors for that
      3. Literally _any excuse_ *EVER* to make use of variable variables for confusion value
  - tracktitleeditor
    - You can do this with mkvpropedit by hand like a big bad matroska gangster but I can never remember the damn syntax
    - It only works on mkv files, natch.
    - I did my very best to take whatever text you type and stuff it into the metadata tag but if you try to paste some emoji nonsense in here it's probably not gonna work and I do not give a single shit.
