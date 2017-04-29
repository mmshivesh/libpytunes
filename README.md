# pyItunes

Created by Liam Kaufman (liamkaufman.com)

Contributions by Liam Kaufman (liamkaufman.com), Steven Miller (copart), dpchu, selftext, z4r, pschorf, Mathew Bramson (mbramson), Roger Filmyer (rfilmyer), cktse

**Before using pyItunes it is recommended that you backup your Itunes Library XML file. Use pyItunes at your own risk - there is no guarantee that it works or will not blow-up your computer!**

## Usage:

```
from pyItunes import Library

l = Library("iTunes Library.xml")

for id, song in l.songs.items():
    if song and song.rating:
        if song.rating > 80:
            print(song.name, song.rating)

playlists=l.getPlaylistNames()

for song in l.getPlaylist(playlists[0]).tracks:
	print("[{t}] {a} - {n}".format(t=song.track_number, a=song.artist, n=song.name))
```

See below for available song attributes.

There is also a deprecated legacy method, which still works for now:

```
from pyItunes import XMLLibraryParser, Library

pl = XMLLibraryParser("iTunes Library.xml")
l = Library(pl.dictionary)

for song in l.songs:
	if song and song.rating:
		if song.rating > 80:
			print(song.name)
```

If your library is very large, reading the XML into memory could be quite slow. If you need to access the library repeatedly, Python's "pickle" can save a binary representation of the XML object to disk for much faster access (up to 10x faster). To use a pickled version, do something like this:

```
import os.path
import pickle
import time
from pyItunes import Library

lib_path = "/Users/[username]/Music/iTunes/iTunes Library.xml"
pickle_file = "itl.p"
expiry = 60 * 60  # Refresh pickled file if older than
epoch_time = int(time.time())  # Now

# Generate pickled version of database if stale or doesn't exist
if not os.path.isfile(pickle_file) or os.path.getmtime(pickle_file) + expiry < epoch_time:
    itl_source = Library(lib_path)
    pickle.dump(itl_source, open(pickle_file, "wb"))

itl = pickle.load(open(pickle_file, "rb"))

for id, song in itl.songs.items():
    if song and song.rating:
        if song.rating > 80:
            print("{n}, {r}".format(n=song.name, r=song.rating))
```

## Notes

Track counts may not match those shown in iTunes. e.g.:

```
l = Library("iTunes Library.xml")
len(l.songs)
```

May report a higher number than the song count shown in iTunes itself. This is because
iTunes does not count things like Podcasts and Voice Memos as "Music," whereas
pyitunes counts **all** tracks.

Version 0.2 adds the ability to get playlists. However, the songs dictionary is keyed on TrackID (as coded in iTunes xml).
Playlists are lists of Song objects, with their order noted as a `playlist_order` attribute.
(note that previously, playlists were lists of PlTrack objects, with their order noted as a `number` attribute.
PlTracks have been removed in favor of modified Songs.)

### Attributes of the Song class:

```
name (String)
artist (String)
album_artist (String)
composer = None (String)
album = None (String)
genre = None (String)
kind = None (String)
size = None (Integer)
total_time = None (Integer)
track_number = None (Integer)
track_count = None (Integer)
disc_number = None (Integer)
disc_count = None (Integer)
year = None (Integer)
date_modified = None (Time)
date_added = None (Time)
bit_rate = None (Integer)
sample_rate = None (Integer)
comments = None (String)
rating = None (Integer)
album_rating = None (Integer)
play_count = None (Integer)
location = None (String)
location_escaped = None (String)
compilation = False (Boolean)
grouping = None (String)
lastplayed = None (Time)
skip_count = None (Integer)
skip_date = None(Time)
length = None (Integer)
work = None (String)
movement_name = None (String)
movement_number = None (Integer)
movement_count = None (Integer)
loved = False (Boolean)
album_loved = False (Boolean)

```

Songs retrieved as part of a playlist have an additional attribute:
```
playlist_order = None (Integer)
```


Song object attributes can be iterated through like this:
```
for key,value in SongItem:
	<interact with specific key,value pair>.
```

You can also convert songs directly to Dictionaries with the ToDict() Method.
```
SongDictionary = SongItem.ToDict()
```

### Attributes of the Playlist class:
```
name (String)
tracks (List[Song])
is_folder = False (Boolean)
playlist_persistent_id = None (String)
parent_persistent_id = None (String)
```
