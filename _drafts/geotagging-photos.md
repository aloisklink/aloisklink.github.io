---
layout: single
title: "Geotagging Photos using Google Location History and Exiftool"
author_profile: true
---

A few years back, I went on a hiking trip, where I used a Campark ACT74
Action Camera, an extremely cheap 4K camera.

One of the features was that it could take a 4K picture 5 times a second,
or a 4K 60fps video in 5 minute chunks. Although the low battery life made
the camera difficult to use (and the fact that when the battery died), I made
almost 100 GB of photos/videos with this camera.

The camera didn't have a GPS unit installed. Additionally, the camera had
an internal clock that constantly reset itself to 2017-01-01 whenever
the battery went out, so I also had loads of photographs incorrectly dated.

However, I realised that I could geotag these photos by using my Google Location
History and `exiftool`, so it was off to do that,
before [Google Photos ended their unlimited storage on June 1st 2021](https://www.theverge.com/2020/11/11/21560810/google-photos-unlimited-cap-free-uploads-15gb-ending).

## Mounting NAS folder locally

Firstly, since my photos were on my NAS, and I wanted to view the photos and
videos locally, I used SSHFS to mount all the photos locally.

SSHFS is slower than other methods of mounting a folder, but it's definitely
the easiest, so it's what I used.

I disabled compression and used the fastest cipher still supported by my
OpenSSH server/client, since as my NAS was on my local LAN, I was not worried
about security or network speed, I was more worried about my slow NAS CPU!

```bash
mkdir --parents /tmp/geotag-photos
sshfs server:/zfspool/geotag-photos /tmp/geotag-photos -o Compression=no -o Ciphers=aes128-ctr
```

## Getting Geographical Information from Google

To geotag my images, I used data from Google Takeout, both my Google Fit
workout history in Garmin TCX format, and my Google Location history, in JSON
format.

I exported both of these as a gzipped tar (`.tgz` file).

Then, to extract the files I wanted, I ran the following:

```bash
mkdir --parents ../extracted/google-fit
tar -xvf takeout-20201212T143859Z-001.tgz --directory=../extracted/google-fit --strip-components=3 'Takeout/Fit/Activities/'
tar -xvf takeout-20201212T143859Z-001.tgz --directory=../extracted --strip-components=2 'Takeout/Location History/Location History.json'
```

## Converting Google Location Histroy JSON to KML

Next, I wanted to use [exiftool to geotag my images](https://exiftool.org/geotag.html).

However, unfortunately, although exiftool currently supports the Garmin TCX format
produced by Google Fit, it doesn't support the Google Histroy GeoJSON file.

Because of that, I used
[Scarygami/location-history-json-converter](https://github.com/Scarygami/location-history-json-converter)
to convert the Google History GeoJSON file to a KML file that `exiftool` supports.

```bash
cd ../extracted
git clone https://github.com/Scarygami/location-history-json-converter.git
python3 location-history-json-converter/location_history_json_converter.py 'Location History.json' location-history.kml
```

Then, finally I copied over all the location history tracks to my NAS:

```bash
mkdir --parents /tmp/geotag-photos/geoinfo
cp -r location-history.kml google-fit/ /tmp/geotag-photos/geoinfo
```

## Geotagging images

Firstly, I found that some of my images were correctly dated, but some of
my images were set to `2017-01-01`, probably due to the internal clock of the
camera getting accidentally reset.

### Correctly dated images

Firstly, I copied over all the correctly dated images and videos into a new
folder:

```bash
mkdir --parents accurate-time/ungeotagged
cp ../camera/{photo,video}/201806* accurate-time/ungeotagged
```

First, I checked what the timezone for `DateTimeOriginal` was on my images:

```bash
me@server:/zfspool/geotag-photos $ exiftool -DateTimeOriginal accurate-time/ungeotagged/20180613_191228A.jpg
Date/Time Original              : 2018:06:13 19:12:28
```

However, `DateTimeOriginal` did not exist on my videos,
and the one in my photos was missing the correct timezone, so I had to add that in
using their filename:

```bash
exiftool '-datetimeoriginal<${filename}+01:00' -overwrite_original accurate-time/ungeotagged/
```

Looks like it's in UTC+00:00 (the photos were taken in +01:00, aka British Summer Time).

```bash
mkdir --parents 'accurate-time/geotagged' # output directory
exiftool -geotag 'geoinfo/google-fit/2018-06-1*.tcx' -geotag 'geoinfo/location-history.kml' '-geotime<${DateTimeOriginal}+01:00' -o 'accurate-time/geotagged/' 'accurate-time/ungeotagged'
```

```bash
exiftool -coordFormat '%.8f' '-keys:GPSCoordinates<$GPSLatitude, $GPSLongitude' -overwrite_original accurate-time/geotagged/2018061{0,1}_*.mp4
exiftool -geotag 'geoinfo/google-fit/2018-06-1*.tcx' -geotag 'geoinfo/location-history.kml' '-geotime<${DateTimeOriginal}+01:00' -o 'accurate-time/geotagged/' accurate-time/ungeotagged/2018061{0,1}_*.mp4
```

For Google Photos to understand the geotagged location of the videos, we
[additionally need to add the `Keys:GPSCoordinates` tag](https://exiftool.org/forum/index.php?topic=11040.0).

We can do this via:

```bash
exiftool -coordFormat '%.8f' '-keys:GPSCoordinates<$GPSLatitude, $GPSLongitude' -overwrite_original accurate-time/geotagged/*.mp4
```

### Incorrectly dated images

#### 2018-06-12 Afternoon

Firstly, I copied over the first batch of photos that were incorrectly dated
starting from: `2017-01-01`.

I manually looked through the images and found that the one called
`20170101_014459A.jpg` was roughly at `20180612-203800`.

This meant that the time offset was (running this in python):

```python
from datetime import datetime

labelled_date = datetime.fromisoformat('2017-01-01T01:44:59')
actual_date = datetime.fromisoformat('2018-06-12T20:38:00')

print(actual_date - labelled_date)
# 527 days, 18:53:01
```

To fix these, we can then run:

```bash
mkdir --parents 2018-06-12-afternoon/incorrect-time
exiftool '-AllDates<${filename}+01:00' ./2018-06-12-time-fucked/afternoon/ -o ./2018-06-12-afternoon/incorrect-time/
exiftool '-AllDates+=1:05:11 18:53:01' ./2018-06-12-afternoon/incorrect-time/ -o ./2018-06-12-afternoon/ungeotagged/
exiftool -geotag 'geoinfo/google-fit/2018-06-1*.tcx' -geotag 'geoinfo/location-history.kml' '-geotime<${DateTimeOriginal}+01:00' -o '2018-06-12-afternoon/geotagged/' './2018-06-12-afternoon/ungeotagged/'
exiftool -coordFormat '%.8f' '-keys:GPSCoordinates<$GPSLatitude, $GPSLongitude' -overwrite_original ./2018-06-12-afternoon/geotagged/*.mp4
```

#### 2018-06-12 Morning

I did the same thing with the next batch, taken on the same day, but in the
morning.

Luckily, I had a frame in the video where I took a picture of my phone,
and the time on it, so that I knew that `2017-01-01 03:00:00` was equal to
roughly `20180612-161100`.

```python
from datetime import datetime

labelled_date = datetime.fromisoformat('2017-01-01T03:00:00')
actual_date = datetime.fromisoformat('2018-06-12T16:11:00')

print(actual_date - labelled_date)
# 527 days, 13:11:00
```

To fix these, we can then run:

```bash
mkdir --parents 2018-06-12-morning/incorrect-time
exiftool '-AllDates<${filename}+01:00' ./2018-06-12-time-fucked/morning/ -o ./2018-06-12-morning/incorrect-time/
exiftool '-AllDates+=1:05:11 13:11:00' ./2018-06-12-morning/incorrect-time/ -o ./2018-06-12-morning/ungeotagged/
exiftool -geotag 'geoinfo/google-fit/2018-06-1*.tcx' -geotag 'geoinfo/location-history.kml' '-geotime<${DateTimeOriginal}+01:00' -o '2018-06-12-morning/geotagged/' './2018-06-12-morning/ungeotagged/'
exiftool -coordFormat '%.8f' '-keys:GPSCoordinates<$GPSLatitude, $GPSLongitude' -overwrite_original ./2018-06-12-morning/geotagged/*.mp4
```

#### 2018-06-13

Luckily, one of my pictures was on my phone, so I had an exact time match:
`2017-01-01T15:41:32` was roughly equal to `2018-06-13T16:14:30`

```python
from datetime import datetime

labelled_date = datetime.fromisoformat('2017-01-01T15:41:32')
actual_date = datetime.fromisoformat('2018-06-13T16:14:30')

print(actual_date - labelled_date)
# 528 days, 0:32:58
```

```bash
mkdir --parents 2018-06-13-afternoon/incorrect-time
exiftool '-AllDates<${filename}+01:00' ./unknown/20170101_1* -o ./2018-06-13-afternoon/incorrect-time/
exiftool '-AllDates+=1:05:12 0:32:58' ./2018-06-13-afternoon/incorrect-time/ -o ./2018-06-13-afternoon/ungeotagged/
exiftool -geotag 'geoinfo/google-fit/2018-06-1*.tcx' -geotag 'geoinfo/location-history.kml' '-geotime<${DateTimeOriginal}+01:00' -o '2018-06-13-afternoon/geotagged/' './2018-06-13-afternoon/ungeotagged/'
exiftool -coordFormat '%.8f' '-keys:GPSCoordinates<$GPSLatitude, $GPSLongitude' -overwrite_original ./2018-06-13-afternoon/geotagged/*.mp4
```

## Uploading Images to Google Photos

Finally, in order to upload the photos/videos,
I just opened up <https://photos.google.com> in my local computer's web-browser,
and dragged the `sshfs`-mounted NAS folder to the web-page.

Although my intranet uses very slow Ethernet-over-Powerline (<100 Mbps),
which makes `sshfs` very inefficient, my internet upload speed is very limited,
and is less than 20Mbps, which remains the bottle-neck.

## Manually correcting video times on Google Photos

Unfortunately, once everything was uploaded, my videos still had the incorrect
time. Their UTC time was shown as the local time, which in Britian during the summer,
is +01:00 hour behind.

I didn't bother to use `exiftool` to fix this, as I was too close to the deadline
to reupload many gigabytes of videos. So instead, I just manually selected all
the videos in the Google Photos browser, pressed **Edit date & time**,
then **Shift dates & times**, and manually gave all the videos a 01:00 hour offset.

It's not the best way of doing this, I'd have to find the correct flag to label
videos for Google Photos using `exiftool` in the future, but with the lack
of unlimited storage for Google Photos nowadays, it's unlikely I will upload
many more vidoe files.
