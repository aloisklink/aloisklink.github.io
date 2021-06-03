---
layout: single
title: "Geotagging Photos using Google Location History"
excerpt: ""
author_profile: true
---

**Remember to rename `elementalfrog` to `my-server`.**


Firstly, since my photos were on my NAS, and I wanted to view the photos and
videos locally, I used SSHFS to mount all the photos locally.

```bash
mkdir --parents /tmp/geotag-photos
sshfs elementalfrog:/zfspool/geotag-photos /tmp/geotag-photos -o Compression=no -o Ciphers=aes128-ctr
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

Since I wanted to upload the images from my server, I needed to create
a VNC connection to that server.

To do this, I first installed VNC:

```bash
sudo apt install tightvncserver
```
