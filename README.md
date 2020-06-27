# xbox2local

Welcome to **xbox2local**, a command line utility written in Python for downloading all your Xbox Live [screenshots and game clips](https://support.xbox.com/help/friends-social-activity/share-socialize/capture-game-clips-and-screenshots) to your computer.
As of this writing, Microsoft only allows automatic uploads of screenshots and game clips to the Xbox Live service, where [subtle rules](https://support.xbox.com/help/games-apps/my-games-apps/manage-clips-with-upload-studio) dictate how long your content sticks around.
Game clips and screenshots can also be [shared](https://support.xbox.com/help/games-apps/my-games-apps/share-clips-xbox-one) to OneDrive for storage or Twitter for social sharing, but this must be done manually on a clip-by-clip basis.

The goal of **xbox2local** is to make the process of saving your screenshots and game clips much faster: each time you run it, it detects all your media on Xbox Live that it hasn't saved before and copies it to a local folder of your choosing.

TODO: Image showing Xbox storage >(automatic) Xbox Live >(xbox2local) local folder.


## Getting Started

1. You'll need a Unix-based command line and any installation of [Python 3.x](https://www.python.org/downloads/). For users of other command lines, see Issue [#4](https://github.com/jdaymude/xbox2local/issues/4).

2. Have your Xbox Live (Microsoft) account email and password on hand. Both Free and Gold accounts are supported.

3. Sign up for an [X API](https://xapi.us/) account, which is what **xbox2local** uses to interface with the Xbox Live service to download your media. Unless you have a huge number of game clips and screenshots, the free plan (60 requests per hour) will work fine.

4. On your X API [profile page](https://xapi.us/profile), sign into Xbox Live with your Xbox Live account email and password.

5. Clone this repository or download the latest [release](https://github.com/jdaymude/xbox2local/releases). Your directory structure will look like:
```
xbox2local
|---- ids.json
|---- README.md
|---- xbox2local.py
```

6. Open `ids.json` and copy your *API Key* and *Xbox Profile User ID* from your X API [profile page](https://xapi.us/profile) into the corresponding fields:
```
{
    "note": "Copy your X API key and Xbox user ID from https://xapi.us/profile here",
    "xapi_key": "your X API key here",
    "xuid": "your Xbox Profile User ID here"
}
```

7. Run **xbox2local** with `python xbox2local.py`.


## Usage

**xbox2local** provides the following command line arguments:

- `--ids_json <IDS_JSON>` allows you to specify an alternative JSON file to read your X API key and Xbox Profile User ID from, which is potentially useful if you have multiple accounts. The default is `ids.json`.

- `--local_dir <LOCAL_DIR>` allows you to specify the local directory to download screenshots and game clips to (e.g., your OneDrive folder). The default is the `xbox2local` directory.

- `--media_type {screenshots, gameclips, both}` allows you to specify whether to download only screenshots, only game clips, or both. The default is both.

After running **xbox2local** at least once in which it succeeds in downloading your media, a `download_history.json` file will be created.
This file stores the ID of every screenshot and game clip that **xbox2local** has previously downloaded so that, on subsequent runs, it does not download duplicates.
If for whatever reason you want to start fresh and redownload all possible media, simply delete/move this file.


## Troubleshooting and Gotchas

**xbox2local** does its best to notify you if anything goes wrong when communicating with the X API or your Xbox Live account.
Some common errors include:

```
ERROR 401: A fresh login is required to gain a new token from Microsoft
```
This means that the X API key and Xbox Profile User ID you provided are correct, but your Xbox Live login via X API has expired.
This is easily fixed by signing into Xbox Live again on your X API [profile page](https://xapi.us/profile).

```
ERROR 401: No API Key provided or invalid API Key
```
This means that either you did not provide your X API key in `ids.json` or the X API key you provided is incorrect.

```
ERROR: Expected xuid <expected xuid> but got <xuid>
```
This means that either you did not provide your Xbox Profile User ID in `ids.json` or the one you provided does not match the account associated with your X API key.

```
ERROR 403: API Rate Limit Exceeded
```
This means that you have made more calls to X API than your subscription plan allows.
The free tier allows 60 requests per hour.
TL;DR: this limit can be violated if you have a huge number of screenshots and game clips.
This can potentially be circumvented by using the `--media_type` option to only download screenshots, waiting for the limit to reset in the next hour, and then using it again to only download game clips.

In detail, **xbox2local** makes the following calls to X API each time it's run:
- One call to the `/v2/accountxuid` endpoint to verify the given X API key and Xbox Profile User ID.
- One call to the `/v2/{xuid}/screenshots` endpoint per page of screenshot results (X API uses pagination to ensure that no single request is too big).
- One call to the `	/v2/{xuid}/game-clips` endpoint per page of game clip results.

Thus, if the total number of pages of results is greater than 60, this error will be raised and X API will stop serving results.
See Issue [#3](https://github.com/jdaymude/xbox2local/issues/3) for further discussion.

```
No new screenshots or game clips to download
```
This message is expected behavior when there really isn't anything new to download.
However, if you receive this message unexpectedly, check your *Settings > Preferences > Capture & Share > Automatically upload* settings on your Xbox.
This should be set to something other than *Don't upload*; otherwise, all screenshots and game clips you capture stay on your Xbox's local storage and are not uploaded to Xbox Live, leaving them inaccessible to **xbox2local**.


## Contributing

If you'd like to leave feedback, feel free to open a [new issue](https://github.com/jdaymude/xbox2local/issues/new/choose).
If you'd like to contribute, please submit your code via a [pull request](https://github.com/jdaymude/xbox2local/pulls).