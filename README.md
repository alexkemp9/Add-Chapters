# Add-Chapters
A BASH-Script to Add Chapters to a MP4 Video File

## *Basics*
This script file is designed for use by any-user (no root access required) within a Terminal. It’s principal purpose is to make it easy & reliable to add Chapters to video files. It has been used under *Devuan* (a systemD-free version of *Debian*). In addition to BASH, the main utilities required are common items such as FFMPEG, PYTHON3 & TPUT (latter comes with NCURSES). Each required app is tested for before use.

## *Extra*
The script advises you to export all current MetaData from the video-file into a text file called (by default) *‘FFMETADATAFILE’*. You are required to created a plain-text file called *“chapters.txt”* which contains the chapter headings. The script can then use a Python script to convert your chapter headings and export them into *FFMETADATAFILE*. Global metadata such as *‘title’*, *‘artist’*, etc. is contained at the top of that file. It is easy to edit and/or add extra global metadata by hand if you wish.

The Python script (called *“helper.py”* by default) is embedded within `add_chapters` and will be exported into the same dir as the MP4 file if not already existing.

## *Begin*
The following will assume that you have created a dir `~/.local/sbin` within which you store the bash-file; this is to set the script as executable for current user only:

```bash
$ chmod 0700 ~/.local/sbin/add_chapters
$ la ~/.local/sbin/add_chapters
-rwx------ 1 user user 17461 Nov 20  2021 /home/alexk/.local/sbin/add_chapters
```
## *Help*
Attempting to run the bare script with zero parameters shows an extended Help message:

```bash
$ ~/.local/sbin/add_chapters
!Fatal Error! No MP4NAME in command.

usage: /home/user/.local/sbin/add_chapters MP4NAME -ACTION

Adds a set of Chapters to a MP4 video.

   MP4NAME     mp4 filename
ACTIONs:
   -a ADD      Add chapter metadata from “chapters.txt” to “FFMETADATAFILE”
               note 1: if “helper.py” is missing it will be auto-created
               note 2: ‘python 3’ is required to be installed
   -c CREATE   Create python script “helper.py” in mp4 directory
               note 2: ‘python 3’ is required to be installed
   -e EXPORT   Export “FFMETADATAFILE” (text file) from MP4NAME into same directory
   (default)   note 3: ‘ffmpeg’ is required to be installed
   -h HELP     Show this Help
   -i IMPORT   Import “FFMETADATAFILE” (text file) into MP4NAME
               Note 3: ‘ffmpeg’ is required to be installed

WORKFLOW:
# https://ffmpeg.org/ffmpeg-formats.html#Metadata-1
# https://ikyle.me/blog/2020/add-mp4-chapters-ffmpeg
In brief:
  a) Put a chapter-free MP4 into a directory
  b) -e EXPORT the metadata from that mp4
  c) Create “chapters.txt” (lines of chapter start-time + title)
  d) -a ADD the chapters from (c) into (b)
  e) -i IMPORT the chapters (new metadata) from (d) into the mp4

At dreadful length:
  1) Place a chapter-free MP4 into a directory
  2) -e EXPORT “FFMETADATAFILE” (metadata) text-file from MP4
  3) Check that there are no Chapters in “FFMETADATAFILE”
  4) (optional) Add/edit Global metadata in “FFMETADATAFILE”
eg:
title=Groundhog Day
artist=Harold Ramis
date=1993

note:
     • In all Metadata, empty lines and lines starting with ‘;’ or ‘#’ are ignored
     • Metadata keys or values containing special characters (‘=’, ‘;’, ‘#’, ‘\’
       and a newline) must be escaped with a backslash ‘\’.
     • Whitespace in metadata (e.g. ‘foo = bar’) is considered to be a part of the tag
       (in example key is ‘foo ’, value is ‘ bar’)

  5) (manual) Create file “chapters.txt” containing only each chapter start-time +
              title, ended by a dummy “END” chapter
              (maintain a rigid format: “<hh:mm:ss><single-space><Chapter-Title>”)
eg:
00:00:14 Splash-Screen
00:00:27 Warning
…
00:08:05 Sleepless in Seattle: MOVIE BEGIN
…
01:45:11 End Titles
01:48:47 END

  6) -a ADD “chapters.txt” to bottom of “FFMETADATAFILE”
note:
     • the python script “helper.py” is used to transform the single-line start-time +
       title entries in “chapters.txt” into mp4 METADATA
     • “helper.py” will be auto-created if it does not already exist in the mp4 directory

After -a ADD the above example becomes (note that last 'chapter' has gone):
[CHAPTER]
TIMEBASE=1/1000
START=14000
END=26999
title=Splash-Screen

[CHAPTER]
TIMEBASE=1/1000
START=27000
END=77999
title=Warning
…
[CHAPTER]
TIMEBASE=1/1000
START=485000
END=695999
title=Sleepless in Seattle: MOVIE BEGIN
…
[CHAPTER]
TIMEBASE=1/1000
START=6311000
END=6526999
title=End Titles

  7) -i IMPORT “FFMETADATAFILE” to original mp4

note:
     • “-c copy” means it takes seconds to process hours-long videos
     • Although -i IMPORT contains “-force_key_frames chapters-0.1”
       within the FFMPEG line to add KeyFrames at each Chapter,
       it is ineffective due to presence of “-c copy”. Remove
       “-c copy” if you want KF at each Chapter (but realise that
       it will then take hours to process every frame in the file).
```
