# vtscheduler
A backend scheduler that will track VTuber live stream (and archive) for Youtube, ~~Bilibili~~, Twitch, Twitcasting<br>
Written in Typescript, and using Mongoose.

## Implementation
- [x] YouTube Implementation
- [ ] Bilibili Implementation
- [x] Twitch Implementation
- [x] Twitcasting Implementation
- [x] Mildom Implementation
- [x] Twitter Implementation

BiliBili Implementation is a little bit hindered because rate limiting, currently working around the limitation :smile:

## Breaking Changes
**0.3.0**
Please run the database script `npm run database` and run the Migrations 2021-02-05<br>
Run this if you already setup this program before this update

## Installation
1. Install Node.js and Prepare a MongoDB Server
2. Run `npm install`
3. Run `npm install -g ts-node`

## Preparation

You need to have MongoDB Server up and running at localhost or Mongo Atlas

### Youtube streams
You need:
- Youtube Data API v3

There's a limit of 10k request per day, so you might want to ask google
or try to get another API key that will be rotated

### Twitch Streams
You need: Twitch API Key, register a new application on your Developer Console

That will create a Client ID and Client Secret for you to use.

### Twitter Spaces
You need: Twitter Developer API Token (Bearer Token).

You need to apply for developer access at https://developer.twitter.com/en and then make sure you're application has access to the v2 API.

## Configuration
Configure the scheduler in [src/config.json](src/config.json.example)<br>
Rename the config.json.example to config.json<br>

```json
{
    "mongodb": {
        "uri": "mongodb://127.0.0.1:27017",
        "dbname": "vtapi"
    },
    "youtube": {
        "api_keys": [],
        "rotation_rate": 60
    },
    "twitch": {
        "client_id": null,
        "client_secret": null
    },
    "twitter": {
        "token": null
    },
    "workers": {
        "youtube": true,
        "bilibili": false,
        "twitch": false,
        "twitcasting": false,
        "mildom": false,
        "twitter": false
    },
    "intervals": {
        "bilibili": {
            "channels": "*/60 */2 * * *",
            "upcoming": "*/4 * * * *",
            "live": "*/2 * * * *"
        },
        "youtube": {
            "channels": "*/60 */2 * * *",
            "feeds": "*/2 * * * *",
            "live": "*/1 * * * *",
            "missing_check": "*/5 * * * *"
        },
        "twitcasting": {
            "channels": "*/60 */2 * * *",
            "live": "*/1 * * * *"
        },
        "twitch": {
            "channels": "*/60 */2 * * *",
            "feeds": "*/15 * * * *",
            "live": "*/1 * * * *"
        },
        "mildom": {
            "channels": "*/60 */2 * * *",
            "live": "*/1 * * * *"
        },
        "twitter": {
            "channels": "*/60 */2 * * *",
            "live": "*/1 * * * *",
            "feeds": "*/3 * * * *"
        }
    },
    "filters": {
        "exclude": {
            "channel_ids": [],
            "groups": []
        },
        "include": {
            "channel_ids": [],
            "groups": []
        }
    }
}
```

**Explanation**:
- mongodb
  - **url**: the MongoDB Server URL without trailing slash at the end
  - **dbname**: the database name that will be used
- youtube
  - **api_keys**: collection of Youtube Data API v3 Keys you want to use
  - **rotation_rate**: the rate of the API will be rotated in minutes
- twitch
  - **client_id**: Twitch Application Client ID
  - **client_secret**: Twitch Application Client Secret
- twitter
  - **token**: The breater token of your Twitter application
- workers:
  - **youtube**: enable Youtube scheduler (ensure that your API keys is enough)
  - **bilibili**: enable bilibili scheduler
  - **twitch**: enable Twitch scheduler (ensure you have Client ID and Secret put)
  - **twitcasting**: enable Twitcasting scheduler
  - **mildom**: enable Mildom scheduler
  - **twitter**: enable Twitter Spaces scheduler
- **intervals**: self-explanatory, all of those are in cron-style time<br>
  You can also disable the workers by putting `null` instead of the crontab styles
- filters:
  - **exclude**: Exclude a `channel_ids` and `groups` from being processed
  - **include**: Include a `channel_ids` and `groups` to be processed<br>
    Warning: Using this will only fetch anything in the `include` filters.<br>
    It's also take precedence from `exclude`

## Running
Make sure you've already configured the config.json and the MongoDB server is up and running

The next thing you need to do is filter what you want and what do you not want for the database.<br>
You can do that by adding the word `.mute` to file in `database/dataset`

Example, you dont want to scrape Hololive data, then rename it from `hololive.json` to `hololive.json.mute`<br>

### Database setup and Channel Models
1. Run `npm run database`, this will run the database creation handler
2. After the manager part came out, press `2`, this will start the initial scrapping process

If you have something changed, you could run that again to update the Channel Models<br>
If you just removed something, you want to run `3` then `2` to reset it.

### Deployment
Its recommended to split the worker into separate server or process to avoid rate-limiting.

After that you need to rename `skip_run.json.example` to `skip_run.json` and add anything you dont need on that server.<br>
Do that on every other server.

## Improving Query Performance
It's recommended to create a `id` and `group` indexes for every collection. It's not made by default so you need to open Mongo Shell or use MongoDB Compass to create the Indexes for it.

For `viewersdatas`, `id` Indexes is enough

**Recommended Indexes**:
- Name: `Identifier`<br>
  Field: `id`: `1`
- Name: `Group`<br>
  Field: `group`: `1`
- Name: `Identifier and Group`<br>
  Field:
    - `id`: `1`
    - `group`: `1`
