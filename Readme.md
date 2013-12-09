# Twitter Broadcast

[![NPM version](https://badge.fury.io/js/twtcst.png)](http://badge.fury.io/js/twtcst) [![Dependency Status](https://david-dm.org/serenity/twtcst.png)](https://david-dm.org/serenity/twtcst)

It’s a nodejs module provide you easy interface to get the stream of tweets.

[Article about this in Russian in my blog.](http://simonenko.su/53381781858/pulse-of-web-developments)

## Installation

```
$ npm install twtcst
```

## Examples

```js
var twtcst = require("twtcst");

// array of words you want to track
var words = ["javascript", "coffeescript"];

// object contains your oauth tokens
var oauth = {
  "consumer_key": "",
  "consumer_secret": "",
  "token": "",
  "token_secret": ""
};

var twitter = twtcst(words, oauth);

var validate = twitter.validate();
var beautify = twitter.beautify();

// twitter stream api
twitter.filter(validate, beautify, function(error, tweet) {
  console.log(tweet);
});
```

## API

`twtcst` has two parameters. The first is `words` you want to search and the second is your `oauth` tokens.

```js
var twitter = twtcst(words, oauth);
```

The `twitter` object has two methods: `search` and `filter`.

**search** implement [Twitter Search API](http://dev.twitter.com/docs/api/1.1/get/search/tweets). It takes a three arguments: validate function, beautify function and a callback. Functions `validate` and `beautify` will be described below. The callback will be caused when all tweets are found. The first argument of callback is error (if it has occured) and the second is array of tweets.

```js
twitter.search(validate, beautify, function(error, tweets) {
  if (tweets) {
    tweets.forEach(function(tweet) {
      message(tweet);
    });
  }
});
```

**filter** implement [Twitter Streaming API](http://dev.twitter.com/docs/api/1.1/post/statuses/filter). It has three arguments: `validate`, `beautify` and callback. Filter cause callback and passed new tweet to it every time new tweet appears in Twitter Stream. The filter pass to callback two options: `error` and `tweet`.

> **Warning**: you can open only one stream per account! If you open the second
> stream, the first stream disconnects.

```js
twitter.filter(validate, beautify, function(error, tweet) {
  if (tweet) {
    message(tweet);
  }
});
```

**Format of tweets** depends on the `beautify` function.

### Words

`words` is array contains words you want to search. It also can contains hashtags started with hash, e.g.:

```js
var words = [
  "#javascript",
  "#coffeescript"
];
```

### OAuth tokens

`oauth` is an object in format required by Twitter API. You can get tokens by [creating new app](http://dev.twitter.com/apps/new) or [from existing app](http://dev.twitter.com/apps).

```js
var oauth = {
  "consumer_key": "",
  "consumer_secret": "",
  "token": "",
  "token_secret": ""
};
```

### Validate

**validate** is a function to filter your tweets. Pass it an array of functions you want to filter your tweets.

```js
var validate = twitter.validate([
  twitter.allowLangs(['en', 'ru']),
  twitter.blockUsers(['simonenko', 'isquariel']),
  twitter.blockWords(['test', 'word', 'array', '#php']),
  twitter.noRetweets(),
  twitter.noMentions(),
  twitter.noDefaults(),
  twitter.maxHashtags(5),
  yourOwnFilter
]);
```

If tweet doesn’t match any of checks you define, it won’t pass on.
Each of these functions takes a tweet as an argument and return true if tweet is valid and false otherwise. E.g.:

```js
noRetweets = function() {
  return function(tweet) {
    if (tweet.text.indexOf('RT ') === 0) {
      return false;
    }
    return true;
  };
};
```

All tweets is in format served by Twitter: [description](https://dev.twitter.com/docs/platform-objects/tweets)

There are some built-in functions to filter:

**blockUsers** filter tweets posted by users you pass to the function. To don’t show tweets from users @simonenko and @isquariel just exec

```js
validate = twitter.validate([
  twitter.blockUsers(['simonenko', 'isquariel'])
]);
```

**blockWords** filter tweets that contains specified words. To hide tweets that contains word ruby and hashtag #php write

```js
validate = twitter.validate([
  twitter.blockWords(['#php', 'ruby'])
]);
```

**maxHashtags** do not skip tweets that contains more hashtags than you specify.

```js
validate = twitter.validate([
  twitter.maxHashtags(5)
]);
```

**allowLangs** show tweets written in specified languages only. E.g.:

```js
validate = twitter.validate([
  twitter.allowLangs(['en', 'ru'])
]);
```

**noRetweets** do not skip old-format retweets (RT @username ...). To use it exec:

```js
validate = twitter.validate([
  twitter.noRetweets()
]);
```

**noMentions** do not skip tweets start with @username and .@username. To use it exec:

```js
validate = twitter.validate([
  twitter.noMentions()
]);
```

**noDefaults** do not skip tweets posted by users with default userpic. To use it exec:

```js
validate = twitter.validate([
  twitter.noDefaults()
]);
```

### Beautify

**beautify** is a function to format tweets you get. Pass it an array of functions you want to use for format

```js
beautify = twitter.beautify([
  twitter.autoLink(false),
  twitter.expandEntities({
    "urls": true,
    "media": {
      "width": 500,
      "height": 500,
      "class": 'tweet_image'
    }
  }),
  twitter.humanDate(),
  twitter.twtcstFormat(),
  yourAwesomeFunction
]);
```

The first function takes as an argument tweet in tweet format subscribed above. The next functions takes as an argument result of previous function. So you have to write functions in the order you want to work on tweets. There are some built-in functions for beautify.

**autoLink** modify `tweet.text`. The function automatically wrap links, hashtags and usernames with links. To wrap, use:

```js
beautify = twitter.beautify([
  twitter.autoLinks()
]);
```

to wrap links only (without usernames and hashtags) use

```js
beautify = twitter.beautify([
  twitter.autoLinks(false)
]);
```

**expandEntities** modify `tweet.text`. It will expand links and images

```js
beautify = twitter.beautify([
  twitter.expandEntities({
    "urls": true,
    "media": {
      "width": 500,
      "height": 500,
      "class": 'tweet_image'
    }
  })
]);
```

set `"urls": true` to expand urls if you want to expand them and set `"media": { ... }` to expand images. Images will be wrapped in an `a` tag, class of this tag will be class you specified in `media.class`. If you specified `media.width` image width will be set to minimal value of media.width and width of the image. `media.height` property works analogously.

**humanDate** adds two fields to tweet: `tweet.human_date` and `tweet.iso_date`. Human date is in format *YYYY-MM-DD HH:MM* and ISO date is Date.toISOString()

```js
beautify = twitter.beautify([
  twitter.humanDate()
]);
```

**twtcstFormat** is a function return tweet converted to format we find convenient for later use

```js
beautify = twitter.beautify([
  twitter.twtcstFormat()
]);
```

The output format is:

```json
{
  "id": "Tweet id"
  "link": "Link to user page on Twitter"
  "avatar": "Link to user profile image"
  "login": "User login (@username without @)"
  "name": "User name or login"
  "text": "Improved text of the tweet"
  "date": "YYYY-MM-DD HH:MM"
  "iso": "Date in ISO"
}

```

## Development

To get the source form Github execute

```
$ git clone git@github.com:serenity/twtcst.git
$ cd twtcst

$ npm link
$ cake build
```

Then you should specify your access token in `examples/staff/oauth.js`. Now you have a working example.

To try `twtcst.filter` execute

```
$ node examples/filter.js
```

The script puts new tweet to console.

To try `twtcst.search` execute

```
$ node examples/search.js
```

First the scripts puts an array of tweets get from search to console and then it will output tweets from stream.

Finally, you can view the working html page with stream of tweets. Just execute

```
$ node examples/socket.js
```

and open the examples/index.html in your browser.

## Authors

* [Alexey Simonenko](//github.com/meritt), [alexey@simonenko.su](mailto:alexey@simonenko.su), [simonenko.su](http://simonenko.su)
* [Sophia Ilinova](//github.com/isquariel), [tavsophi@gmail.com](mailto:tavsophi@gmail.com)

## License

The MIT License, see the included `License.md` file.