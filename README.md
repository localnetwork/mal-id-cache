# mal-id-cache

This is a cache of IDs on [MAL](https://myanimelist.net)

[cache](./cache) includes anime, manga, person and character IDs.

The JSON files for anime/manga are structured like:

```
{
    "sfw": [
        1,
        5,
        ....

    ],
    "nsfw": [
        188,
        203,
        ...    
    ]
}
```

For person/character:

```
{
    "ids": [
        1,
        2,
        ...
    ]
```

### Raison d'être

The reason this exists is because there's currently no easy way to get a list of all the approved entries on MAL. 

Whenever an entry requested to be added (by a user), it gets an ID and it "on the website" - unlisted; at this point no one can add it to their list.
 
If a moderator approves the entry, it keeps the same ID and becomes an entry that people can view publicly.

If its denied, the ID disappears and becomes a 404, leaving a 'gap' in the IDs for MAL. At the time of writing this, the most recently approved anime ID is 40134, but theres less than 17000 anime entries on MAL.

There have been a few cases where IDs are re-used for others, but that is not the common case.

This uses the [Just Added](https://myanimelist.net/anime.php?o=9&c%5B0%5D=a&c%5B1%5D=d&cv=2&w=1) page on MAL to find new entries, but all that page is is a search on the entire database reverse sorted by IDs. New entries may appear on the 2nd, 3rd, or even 20th page, if a moderator took a long time to get around to it.

The most obvious application for this cache is to use the cache to choose an entry at random.

This will be updated whenever a new entry is added.

Since Github doesn't allow you to serve large files without an authenticated token, the easiest way to download this and keep it updated is to `git clone https://github.com/seanbreckenridge/mal-id-cache` and have a script that `git pull`s periodically.

Check [the config file](./default_config.toml) for how often this checks different ranges of IDs.

### Installation

If you'd like to set up your own instance:

To setup the cache, we have to use a local [Jikan](https://github.com/jikan-me/jikan) instance since the remote one would cache requests for too long. See [here](https://github.com/jikan-me/jikan-rest) for how to set that up.

After Finishing Step 1 on that page (Installing Dependencies):

```
# Clone Repo
mkdir ~/.mal-id-cache
git clone https://github.com/seanbreckenridge/mal-id-cache ~/.mal-id-cache/repo
cd ~/.mal-id-cache/repo
# Install python dependencies (see below for alternatives)
pipenv install -r requirements.txt
pipenv shell
# Install mal_id_cache script
python3 setup.py install
# Clone jikan-rest (into this directory is fine, though you can have it somewhere else)
git clone https://github.com/jikan-me/jikan-rest
cp env.dist jikan-rest/.env  # Copy environment settings (so requests are cached for too long)
cd jikan-rest
composer install
php artisan key:generate  # Generate APP_KEY
cd ~/.mal-id-cache
mal_id_cache --init-dir  # Setup directory structure at ~/.mal-id-cache
```


How you install python requirements is up to you, you can import it into a [`pipenv`](https://realpython.com/pipenv-guide/) using the command above, create a virtualenv and install it there, or just install them directly into the global modules. 

Start the jikan-rest server: `php -S localhost:8000 -t jikan-rest/public >> ~/.mal-id-cache/logs/jikan-requests.log 2>&1`

Run the script:

```
Usage: mal_id_cache [OPTIONS]

  Caches IDs for MyAnimeList

Options:
  --config-file PATH        Override the default .toml config file
  --dry-run / --no-dry-run  Don't affect local files or make requests, log
                            actions instead
  --loop / --no-loop        Run the process till stopped, checking for new
                            entries periodically
  --server / --no-server    --loop, and open a socket (default port: 32287) to
                            listen for requests from other processes
  --init-dir                Make sure directories at ~/.mal-id-cache are setup
                            properly and exit
  --initialize              Initialize each cache -- deletes and re-requests
                            everything.
  --force-state INTEGER     Update all 'last checked' times for the state
                            files to 'n' seconds ago
  --delete                  Delete the cache and state files if they exist and
                            exit
  --help                    Show this message and exit.

```

#### Thanks

Thanks to [lynn root/mayhem mandril](https://github.com/econchick/mayhem), for some asyncio best practices.
