---
layout: post
title: "Getting Started with Twitter's Academic Track API"
date: 2021-03-25 18:18:00 +0000
categories: [python, scraping, tutorial]
---

_I originally published this article on 5 March on Medium. I've decided that I don't need their paywall._

I'm teaching graduate students how to use the Twitter API, so I figured I would give the new Academic Track API a spin and see what's changed in the couple of years since I used Python to work with Twitter data.

My goal is provide a brief guide on the following:

- How and where to apply for access
- Setting up your environment
- Writing your first query
- A little bit about the additional query parameters

Prerequisites:

- None! This should all work on colab (but see this stackoverflow answer on how to download files saved in a colab notebook)

## Step 1: Applying for Academic Track

Twitter released added a new account type to its API specifically for academic researchers last month. What's more, this includes full archive search. If you've worked with Twitter data as an academic before, you'll probably be dancing like I am! If not, then count yourself lucky you now have the possibility to access most tweets published since 2006. (I assume most and not all, but I may be corrected).

In order to apply for this, you will have to be a graduate student or academic researcher working on a project that requires Twitter data. Details on application and the link to apply can be found here.

Twitter will ask you to write a short application, including your name, affiliation, proposed project, and so on. A few things to note:

- You will need some proof that you're a student or a scholar. A profile on your department's website, or Google scholar, etc. will be sufficient.
- I don't think they give blanket approval, so do have a project in mind that could at the very least be supplemented by access to Twitter data.

I received my approval within less than a week of applying, but obviously I cannot guarantee how long they will take with yours.

## Step 2: Setting Up Credentials

In order for the API to accept your requests, you will need a valid set of credentials. Otherwise anybody could (ab)use the API, and the previous step would be completely pointless! In this step we will retrieve our newly gained credentials and store them in such a way that our Python script can automatically read them.

Once you've received your approval, you'll get access to a dashboard. From here you'll see your Project App (with whatever name you decided to give it—mine is a largely nonsensical Hello World reference). Click on the key icon to see your credentials.

You now need to create a file to store your credentials. The default location is `~/.twitter_keys.yaml`. I stored mine in a different location along with the various access keys I have for different APIs—you will need to point the script to the credentials file later on anyways.

_A quick explanation of the previous bit: `~/` is your home directory. `.` before a filename makes it "hidden" to commands like `ls` without `-a` or your file explorer. `.yaml` is a human-readable data format that you might've come across in the header block of an RMarkdown document._

If you want to use the historical API (i.e. gain access to all Tweets), then your credentials file should look like this:

~~~ yaml
search_tweets_v2:
  endpoint:  https://api.twitter.com/2/tweets/search/all
  consumer_key: <CONSUMER_KEY>
  consumer_secret: <CONSUMER_SECRET>
  bearer_token: <BEARER_TOKEN>
~~~

_Note that if you're only interested in Tweets from the past 7 days, you can use the "Recent search" endpoint, which allows you to request slightly more tweets per 15 minutes (450 vs 300) in which case the second line of your credentials file should read:_

~~~ yaml
  endpoint:  https://api.twitter.com/2/tweets/search/recent
~~~

Save this file, and if you're on Linux maybe also `chmod 600` it. If you're using git, make sure not to accidentally upload this file! Anyone who gains your credentials can abuse your API access, which may result in your application (credentials) getting revoked.

## Step 3: Getting Tweets

For this part, you'll need to install the following packages:

~~~ yaml
pip install searchtweets-v2
pip install requests
pip install pyyaml
~~~

In the past I'd used `tweepy`, this time I am using the official package provided by the Twitter developers. As far as I can tell it's a bunch of convenience functions and wrappers for the API. Remember that at the end of the day, the API is just an endpoint for HTTP GET requests that sends back data in `json` format, so nothing about this is Python-specific.

In less technical terms, the API is kind of like a specialized "website". It has a URL, like other websites, but instead of showing a webpage it sends back data files in `json` format. In order to get a response from this webpage, you need to a) provide proper authentication and b) provide a properly formed request. Because it's a GET request, the parameters of your query are actually passed in the URL, after the `?`. The purpose of the API is to provide a more efficient interface or developers (and researchers), who need the data contained in the website rather than viewing the website itself.

### Step 3a: Authenticate Your Token

First things first, you need to read in your credentials from the file. The library provides a convenience function for this:

~~~ python
import searchtweets as tw
search_args = tw.load_credentials("~/.twitter_keys.yaml",
                                     yaml_key="search_tweets_v2",
                                     env_overwrite=False)
~~~

### Step 3b: Prepare Your Query

They also provide a convenience function for preparing your query, `gen_request_parameters`. This is where we will narrow our search, but I'll start with a simple example. In this case I will look at the 100 most recent tweets that contain the phrase "stonks":

~~~ python
query = tw.gen_request_parameters(
    query='stonks',
    results_per_call=100
)
print(query)
~~~

If you want to search for tweets containing a different phrase, then substitute "stonks" for some other phrase. I'll get to customizing parameters a bit further down.

When you execute that code, you should get back the following:

~~~ python
{"query": "stonks", "max_results": 100}
~~~

This is valid `json` to be passed to the query function. Note that we have restricted the number of results as to avoid hitting the query cap.

### Step 3c: Submit Your Query

Again, they provide a convenience function for this: `collect_results`. Here's the code:

~~~ python
tweets = tw.collect_results(
    query,
    max_tweets=100,
    result_stream_args=search_args
)
~~~

You'll notice that you pass your credentials in the `result_stream_args` argument of this function.

## Step 4: Analysing the Output

Let's have a look at what we have:

~~~ python
print(
 f"The function returns a {type(tweets)},",
 f"where each tweet is held in a {type(tweets[0])}.",
 f"The total number of objects is: {len(tweets)}",
 f"The first 100 have the following keys (attributes): {tweets[0].keys()}",
 f"The last is a token for the next query, with the following keys:\n {tweets[-1].keys()}",
 sep="\n"
)
~~~

From which we get the output:

~~~ python
The function returns a <class 'list'>,
where each tweet is held in a <class 'dict'>.
The total number of objects is: 101
The first 100 have the following keys (attributes): dict_keys(['id', 'text'])
The last is a token for the next query, with the following keys:
 dict_keys(['newest_id', 'oldest_id', 'result_count', 'next_token'])
~~~

So we requested 100 tweets, and got back a `list` of 101 `dict`s, where the first 100 are the tweets and the last one is a token for getting the next "page" of results (to be discussed in a future blog post).

Let's also look at the contents of the tweets themselves!

~~~ python
[
    print(tweet['text'], end='\n') for tweet in tweets
    if 'text' in tweet.keys()
]
~~~

I won't bother posting the output of that command here (in three emojis: 🚀💎🙌), but I will explain the above command briefly. I'm using list comprehension, which allows me to "unpack" an iterable object (such as a list). I know everything is a dictionary, so I am telling it to print the "text" entry of the dict, as long as "text" is in the keys.

## Step 5: Customizing Your Queries

Obviously we want to be able to search for a lot more than just the word "stonks". Fortunately the developers provide [extensive documentation](https://developer.twitter.com/en/docs/twitter-api/tweets/search/integrate/build-a-query) (which took me a bit of time to find). I'll summarize it here in Python terms:

Let's look at a few of the key parameters in the `gen_request_parameters` function. Below I have an example of another query:

~~~ python
query = tw.gen_request_parameters(
    query = "#metoo (place_country:MX OR place_country:IN) -is:retweet -is:nullcast",
    results_per_call = 100,
    start_time = "2021-01-01", 
    end_time = "2021-01-31",
    tweet_fields = "id,created_at,text,author_id,context_annotations,entities"
)
~~~

To the query argument I've passed the following string:

~~~ python
"#metoo (place_country:MX OR place_country:IN) -is:retweet -is:nullcast"
~~~

Let's break this down:

- `#metoo` the query begins with the special hashtag operator. This will match tweets that contain the hashtag: #metoo. Note that this will not match the text "metoo", or a longer hashtag "#metookutsu"
- ` ` Spaces act as boolean AND operators.
- `( )` brackets (parentheses) group operators together.
- `place_country:MX` filters for tweets geo-tagged in Mexico. Note that the country names are written as [two-letter ISO codes](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2), and also that the vast majority of tweets will not have a country code, so adding this filter will also end up filtering out of tweets that were in fact tweeted in your country of interest.
- `OR` a boolean OR operator.
- `-is:retweet` removes all retweets; in other words, you only get "primary" tweets. The minus sign negates the following argument, so we are saying IS NOT RETWEET. As a researcher, this is an incredibly useful one.
- `-is:nullcast` removes tweets created purely as promotions/ads.

The next three arguments are fairly self-explanatory, I think. I should only note that the start and end dates should be formatted as `yyyy-mm-dd HH:MM:SS`. Also note that with the new historical API, you can go back as far as you like, and that results will be returned in reverse chronological order.

The final argument is `tweet_fields`, which is a new feature (at least since I last used the Twitter API). Previously, when you requested a tweet via the API, you would get dozens of fields of metadata, some more useful than others. The API now only returns the id of the tweet and its text by default.

In order to retrieve further fields, you need to pass additional parameters to the `tweet_fields` argument. Note that this takes a single string, delineated by commas. I've included the following fields:

- `id,created_at,text`—the tweet id, the date of its creation, and the actual text of the tweet.
- `author_id`—the id string of the author. This is distinct to the `@username`; this id is permanent for each account.
- `context_annotations`—a new feature. It looks like some NER (named entity recognition), recognizing famous people/places etc., as well as some topic analysis.
- `entities`—things like `#hashtags`, `$cashtags`, and so on.

For a full list, see [this page](https://developer.twitter.com/en/docs/twitter-api/data-dictionary/object-model/tweet).

## Next Step: Users, Dealing with Rate Limiting, and More

That's all I have time for now, but this should be enough to get your tweet collection up and running. If you run into HTTP Error 429, it means you've requested too many tweets in a given period of time. You can find more about rate limits [here](https://developer.twitter.com/en/docs/twitter-api/rate-limits).

The documentation and examples provided by Twitter are also great, so do go check those out! I've linked some key pages in this article. Thanks for reading!
