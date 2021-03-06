![Rockets](header.gif) [![Author](http://img.shields.io/badge/author-@rudi_theunissen-00aced.svg?style=flat-square)](https://twitter.com/rudi_theunissen) [![Support](https://img.shields.io/badge/support-donate-399c99.svg?style=flat-square)](https://plasso.co/rudolf.theunissen@gmail.com) [![Status](https://img.shields.io/badge/status-stopped-cc6666.svg?style=flat-square)]()
---

**Mission**
>Provide a way to stream new content on reddit.com without using the API.

Many reddit bots rely on monitoring new content, constantly sending requests to keep up. Unfortunately this means that bots can't use their precious rate-limit tokens to then *do something* with that content.

*Rockets* allows you to subscribe to a `channel`, with the ability to specify content `filters`. All you need to do is open a web socket connection to the command center at `ws://rockets.cc:3210` and transmit your subscription.

You will receive JSON models exactly as they appear in reddit listings, ie. with `kind` and `data` keys. These will be sent one at a time, but are not guaranteed to be in perfect chronological order due to the level of concurrency on the server.

#### Demo

See [rockets/demo](https://github.com/rockets/demo).

#### Client

See [rockets/client](https://github.com/rockets/client).

## Usage

Subscriptions are sent to `ws://rockets.cc:3210` as JSON in the following format:

```js
{
    "channel": "",  // Model channel, eg. 'posts' or 'comments'
    "include": {},  // Rules for which models to include.
    "exclude": {},  // Rules for which models to exclude.
}
```

### Channels

- `comments`
- `posts`

### Rules

All rules can be provided as either a single value or an array of values.
A rule is considered a match if any of the values match the corresponding value in the model.

*Note: all string and regular expression rules are case-insensitive unless explicitly stated otherwise.*

#### Comments

| Key       | Type            | Description                                                      |
|-----------|-----------------|------------------------------------------------------------------|
| contains  | string (regex)  | Comment body (markdown)                  |
| subreddit | string          | Subreddit in which the comment was made, eg. "subreddit"         |
| author    | string          | The user who made the comment, eg. "username"                    |
| post      | string (exact)          | Comments that are replies to a specific post, eg. "t3_abcd"      |
| root      | boolean         | Comments that are not replies to other comments                  |

#### Posts

| Key       | Type            | Description                                                      |
|-----------|-----------------|------------------------------------------------------------------|
| contains  | string (regex)  | Link title and selftext (markdown)
| subreddit | string          | Subreddit in which the post was made, eg. "subreddit"            |
| author    | string          | The user who made the post, eg. "username"                       |
| domain    | string          | Link's domain, or "self.subreddit" if it's a selfpost            |
| url       | string          | Link's URL, or the post's permalink if it's a selfpost           |
| nsfw      | boolean         | Flagged NSFW at the time of creation                             |

### Example

```js
{
    "channel": "comments",
    "include": {

        "subreddit": [
          "space",
          "spacex",
          "science"
        ],

        "contains": [
          "mars",
          "rockets",
          "\\bion\\b"
        ]
    }
}
```

## Limitations

### Dropped connections

The server will drop all connections when being updated or restarted, which can't be avoided. Make sure that your client
will attempt to reconnect automatically when a connection is dropped. The official [rockets-client](https://github.com/rtheunissen/rockets-client) does this automatically.

Updates will be posted to /r/redditdev in the event of unexpected downtime.

### Latency

Even though the command center is in low reddit orbit, there will always be some delay between a model's creation and its broadcast. Most of the time this delay will only be a few seconds, but could be several minutes in some cases. This occurs when reddit.com goes down, or during very busy periods. The command center will catch up again when things are back to normal or when busy periods subside. Data will not be lost during this time.

### Bandwidth

A single model is roughly 1kb of JSON. The command center receives an average of 30 models per second. This works out to about 2.6GB per day, per unfiltered connection. With a limit of 8TB downstream traffic per month, this equates to a maximum of roughly 100 concurrent unfiltered connections.

### Subscriptions

You are only allowed one subscription per connection per channel, where new subscriptions replace previous ones.
It's possible that some unwanted models may still be received after a subscription has been replaced, so you should
open a new connection when you need a strict boundary between subscriptions.

## Support

There are a few ways to resupply and assist the command centre:

- Use rules to avoid receiving unwanted data.
- [Donate](https://plasso.co/rudolf.theunissen@gmail.com) a few dollars to help cover server costs.

## Credits

Illustrations by Ken Samonte.
