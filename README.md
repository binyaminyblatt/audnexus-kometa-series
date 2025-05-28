# Audnexus + Kometa: Audiobook Series Collections

This is a guide on how to set up
[Kometa](https://github.com/Kometa-Team/Kometa) to
automatically generate collections from your audiobook series' that are
populated from the
[Audnexus metadata agent]([https://github.com/djdembeck/Audnexus.bundle](https://github.com/binyaminyblatt/Audnexus.bundle/tree/backup-api)) for
Plex. This is done by generating collections from the "Mood" tags that Audnexus
adds to each book's metadata from each Audible series they are a part of.

> Kometa (formerly known as Plex Meta Manager) is a powerful tool designed
> to give you complete control over your media libraries. With Kometa, you
> can take your customization to the next level, with granular control over
> metadata, collections, overlays, and much more.

There are other tools made with a similar purpose such as
[Plex-Auto-Collections](https://github.com/mza921/Plex-Auto-Collections) but
I've had the most success with Kometa.

## 1. Setting Up Audnexus

First, you'll need to set the Audnexus metadata agent up using
[their official guide]([https://github.com/djdembeck/Audnexus.bundle](https://github.com/binyaminyblatt/Audnexus.bundle/tree/backup-api)#getting_started).
During this process, you should uncheck the "Append authors as Mood tags"
option, unless you'd like a collection to be created for each author in addition
to each series.

If you already have this agent set up and had previously checked
this option, you can update the settings to stop appending authors as mood tags,
and then refresh all book metadata in your audiobook library. If this doesn't
remove the authors as mood tags, update to the latest version of the agent and try
again, as this was recently fixed.

## 2. Setting up Kometa

Next, you'll need to set up Kometa, using
[their official guide](https://kometa.wiki/en/latest/kometa/install/overview/).
Personally, I've only set up this tool using
[Docker](https://kometa.wiki/en/latest/kometa/install/docker/)
(specifically
[unRAID](https://kometa.wiki/en/latest/kometa/install/unraid/)), but they have a
variety of other options. Their guide is pretty comprehensive, so I'll leave it
up to you to decide which one you'd like to use.

Regardless of which setup you choose, you'll eventually get to a point where
you'll need to set up your config files. First, I recommend copying the two YML
files included in this repo into your root config directory.

#### `config/config.yml`

```yml
libraries:
  REPLACE_WITH_AUDIOBOOK_LIBRARY_NAME:
    collection_files:
      - file: config/audiobooks.yml
    operations:
      delete_collections:
        managed: true
        configured: false
        less: 1

plex:
  url: REPLACE_WITH_LOCAL_PLEX_URL
  token: REPLACE_WITH_PLEX_TOKEN

tmdb:
  apikey: REPLACE_WITH_TMDB_API_KEY
```

#### `config/audiobooks.yml`

```yml
templates:
  audiobooks:
    builder_level: album  # ensure each collection is made from an entire book's album, not individual tracks on it
    smart_filter:
      limit: 1000  # without a custom limit, the max books in a series will be cut off at 10.
      sort_by: title.asc  # sort each series by sort title ascending
      all:
        album_mood: <<value>>
      # Exclude all album moods beginning with these prefixes.
      # These are tags that Audnexus adds to albums so third-party apps can use the data,
      # but they should not be included in user-facing collections.
      not:
        album_mood.begins: 
          - PMM_exclude
          - KOMETA_exclude
          - PMM_ignore
          - KOMETA_ignore

dynamic_collections:
  Audiobook Series:
    type: album_mood
    title_format: <<key_name>>
    remove_prefix: 'Series: '  # remove the "Series: " prefix added by Audnexus
    remove_suffix: ' Series'   # optionally remove " Series" suffix used in some Audible naming
    template: audiobooks
```

Then, you'll need to replace the variables in `config.yml` I left in `ALL_CAPS`
with their respective values:

1. `REPLACE_WITH_AUDIOBOOK_LIBRARY_NAME` - Replace this value with human
   readable name your audiobook library in Plex was created with. In my case
   it's simply "Audiobooks".
2. `REPLACE_WITH_LOCAL_PLEX_URL` - This should be replaced with the local URL
   your plex server can be found at, often just an IP address. In my case this
   is `http://192.168.1.2:32400`.
3. `REPLACE_WITH_PLEX_TOKEN` - For this part, you'll need to find your Plex
   authentication token following the official guide on the Plex website: https://support.plex.tv/articles/204059436-finding-an-authentication-token-x-plex-token/
4. `REPLACE_WITH_TMDB_API_KEY` - While this setup will not require
   [TMDB](https://www.themoviedb.org/) in any way, it is still required for
   Kometa to run. It is, however, very easy to obtain one following
   Kometa's guide for it: https://kometa.wiki/en/latest/kometa/install/docker/#getting-a-tmdb-api-key.

## Finishing Up

And that should be all you need! Once both config files are in the right place
with all the proper variables replaced, you can start up Kometa to see if
everything is working properly. You will know it is if you see collections
being created and populated for each `Series: Series Name` mood tags each of
your books is tagged with.

The next step for me would be to set up Kometa on a schedule so it always
generates new collections as new books are added to the server. You can do that
by checking out their
[Scheduling Guide](https://kometa.wiki/en/latest/kometa/guides/scheduling/).

If you have any questions, run into any issues getting this set up, or have any
suggestions to improve my config, feel free to leave an issue on this repo.
