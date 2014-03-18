---
layout: post
title: "MKV SRT Extractor for Samsung Plex users"
date: 2014-03-15 23:51:25 -0600
comments: true
categories: media, script, ruby
---




After I finally built the _Gunpla_ I got a couple years ago (Which I'll write about  later on). I felt more interested about the series, [Gundam 00](http://gundam.wikia.com/wiki/Mobile_Suit_Gundam_00).

Since I watch most of the series and movies with [Plex](https://plex.tv), I set everything up was ready to watch it on the TV. The opening started to play, but there was something missing: The subs! Unless you understand Japanese, subtitles are essential.

Unable to start watching it I tried to figure out whether it was my TV's Plex client or the Plex Media server configuration. Searching on the web, I found out that the problem was that Samsung's Plex App doesn't support embedded subtitles (e.g. subtitles contained in a MKV container file).

But subtitle files on the same root directory were supported, so what I had to do was to use two tools form [MKVtoolsnix](http://www.bunkus.org/videotools/mkvtoolnix/):   

 - ___mkvinfo___ to identify the track inside the MKV containing the subtitles.

 - ___mkvextract___ to extract the subtitles from the identified track.
Since mkvextract is a command line tool, all I needed was to make a small bash script to iterate and extract the subttiles file from each of the MKV files from the current directory:

``` bash
for file in *.mkv
do
	name=`echo $file | cut -f1 -d'.'` 	
	mkvextract tracks "$file" TRACKNUMBER:"$name".ass
done
```


Even though each subtitle file was extracted and named the same as it's corresponding video file, problem persisted, the subtitle files were detected but not shown. 

It turns out that Samsung's Plex App only support [SubRip Text (SRT)](http://www.matroska.org/technical/specs/subtitles/srt.html) files, and the subtitles contained on those MKVs are rich [SubStationAlpha (ASS/SSA)](http://www.matroska.org/technical/specs/subtitles/ssa.html) subtitles. 

Apparently the support for both _SSA_ subtitle files and embedded subtitles  were due to Samsung SDK restrictions.

Having no success finding a proper _SSA_ to _SRT_ batch converter (for Linux or OSX), I made a little script to extract those SSA subtitles from the MKVs, and converting each of them into SRT format. You can get it [here](https://github.com/deivuh/MKV-SRT-Extractor).

Make sure to have both [Ruby](https://www.ruby-lang.org/en/downloads/) and [MKVToolnix](http://www.bunkus.org/videotools/mkvtoolnix/) installed


