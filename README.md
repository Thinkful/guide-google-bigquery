# Google BigQuery


## What is it?

Google BigQuery is a tool that allows you to execute SQL-like queries on large amounts of data at outstanding speeds. You can easily load your own datasets or, for sandbox play, you can use one of the datasets they provide. Results can be stored on Google Cloud or downloaded as a CSV.


## Is it free?

Almost! It's free to upload datasets up to 10MB; and you can do as many queries as you want. Other operations — such as saving a table to the cloud — requires you to set up billing. Setting up billing can be a bit of a hassel but it doesn't cost much ($0.026 per GB, per month) and, for small queries, you won't need it.


## BigQuery Browser Tool

In this tutorial we'll be doing a code-along using the [BigQuery Browser Tool](https://developers.google.com/bigquery/bigquery-browser-tool), which allows you to load data, run queries, and download the results all in your [web browser](https://bigquery.cloud.google.com/). If you're interested in using BigQuery without this tool, check out Google's page on [synchronous and asynchronous queries](https://developers.google.com/bigquery/querying-data). 

### Set Up

In order to follow along you'll need to start a sample project on Google BigQuery by going to the [Google API Console](https://console.developers.google.com/project). All you need to do is sign in to your Google Account (or create one), and select `Create Project`.

![select project on google big query](http://i.imgur.com/eq8xOLy.png)

The project name can be anything, but the project ID has to be unique across all of BigQuery (it's like picking a Twitter username all over again!). In a few moments, your project should pop up.

_Select a project name_
![creating your google big query project](http://i.imgur.com/7Jiy59I.png)

_Your project is being created_
![project creation in progress](http://i.imgur.com/VAay00c.png)

_Your project was created_
![project was created](http://i.imgur.com/WJ3206t.png)

To access the Browser Tool, click on your newly created project, and, on your Project Dashboard, click `Try BigQuery` in the bottom left corner. This should bring you to the tool where we will be running our queries.

![try bigquery](http://i.imgur.com/4c5xLlZ.png)


### Getting Data

Before we can run any queries, we need some data! There are a couple of options here: 

* load your own data
* query some of their sample data

To make sure we're all on the same wavelength, let's use their sample data. You can find the dropdown on the left side under the tab `publicdata:samples`. So many choices!

![view public bigquery data](http://i.imgur.com/bDy6uD4.png)

### Querying Data

_BigQuery uses a slight variant of SQL expressions. For a refresher, check out [this SQL query guide](http://technet.microsoft.com/en-us/library/bb264565)._

To create your first query, click `Compose Query` on the top left. A query box should show up at the top of your screen and whichever dataset you have selected should be below it. To start, let's work with the `shakespeare` data.

_Selecting the data_
![selecting the data](http://i.imgur.com/LVmEuql.png)


This dataset contains the __words__ in Shakespeare's works, the __word_count__ for each word, in which __corpus__ the word appears, and the __date__ the corpus was written. 

#### SELECT and FROM

Select is the most basic clause and specifies _what_ it is that you want to be returned by the query.From specifies what dataset we are using.

So, if we want to see a list of all of Shakespear's works (or "corpuses") we could try: 

```
SELECT corpus
FROM (publicdata:samples.shakespeare)
```

_Note: SQL is not case sensitive, but it is standard to write keywords such as SELECT in capital letters for clarity_

![](http://i.imgur.com/tQgl9Zi.png)

Ah! That query returned a corpus each time it was listed in the dataset - over 100,000 rows of data! It would be much better if we could __group by__ name, right?

#### GROUP BY

The GROUP BY clause tells BigQuery to group the data by some parameter. In our case, we want to group by corpus. Let's try:

```
SELECT corpus
FROM (publicdata:samples.shakespeare)
GROUP BY corpus
```

Now we have a list of all of Shakespear's works, each listed only once.

![](http://f.cl.ly/items/2H0F2q211p2d1l1z323G/Screen%20Shot%202014-08-29%20at%203.30.48%20PM.png)

_Aside: we can see that the query returned 42 rows, but isn't is generally accepted that he only had 37 works? Even if you include his lost plays, it's still only 40. We still have 2 rogue rows! It's because this list includes "various" and "sonnets" as corpus names, which accounts for the last two rows and solves our mystery._

#### SUM and COUNT

Let's switch gears. Say we want to count something — say, the number of words in Shakespeare's works. Luckily, we have `word_count` which represents how many word that particular word appeared in a particular corpus. We can just sum all of these values, and we are left with the total number of words that he wrote. 

```
SELECT SUM(word_count) AS count
FROM (publicdata:samples.shakespeare)
```

_Notice: we have to use the keyword `AS` to specify what we want the count of contributors to be called. This is a variable name and could be anything you want._

945,845 words! Pretty good - but there must surely be some duplicates. How would we query the number of unique words that he used?

```
SELECT COUNT(word) AS count, word
FROM publicdata:samples.shakespeare 
GROUP BY word
```

Here we use the `COUNT` function to count how many words they are, and group them by word so as not to show duplicates. 32,786 unique words! But what is the count value actually showing? The "count" column represents in how many of the 42 works that word appears. So, the word "brave" is used in 35 of his 42 works, while "That" is used in all 42 and "left'st" is only used in 1. 

#### ORDER BY

Let's check out the most popular words are and how many times they were used: 

```
SELECT word, SUM(word_count) as count
FROM (publicdata:samples.shakespeare)
GROUP BY word
ORDER BY count DESC
```

We use `ORDER BY` to do just that - order the data by some parameter, in our case, the sum of word counts. `DESC` orders the counts in descending order, and the opposite is `ASC`, for ascending.  
Surprise surprise, the words "the", "I", and "and" are the big winners. 

![](http://i.imgur.com/JPOmpLP.png)

#### WHERE

Hmm, well those top words aren't that interesting. I wonder how many times the word "swagger" or "bedazzled" come up? We can use the `WHERE` clause to filter data to meet our speficications.

```
SELECT word, SUM(word_count) as count
FROM (publicdata:samples.shakespeare)
WHERE word = "swagger"
OR word = "bedazzled"
GROUP BY word
ORDER BY count DESC
```

_Note: You can also use "AND" instead of "OR" — this would show 0 results since no text has bedazzled AND swagger — what a shame. Another option is "CONTAINS" (or "NOT" word "CONTAINS" for does not contain)._

Looks like Shakespeare was pretty hip for his time. Play around with some other words - try your name.

## Exploring GitHub Data

### GitHub Archive Data

Have you heard of [Google's Ngram Viewer](https://books.google.com/ngrams/)? Basically, "it displays a graph showing how those phrases have occurred in a corpus of books (e.g., "British English", "English Fiction", "French") over the selected years." 

Why don't we try and do the same with programming languages over all repositories in GitHub. Here, we're going to make a graph that compares two or more github languages over time. It'll look something like this:

![](http://i.imgur.com/xURukGw.png)

#### Gather GitHub Data

For this graph, we're going to need access to the GitHub Archive data, which they have conveniently provided for us through the BigQuery browser tool. To get the data, press the little arrow next to your project name, click "Switch to project > Display project..." and type in "githubarchive".

_Display a new project_
![display a new project](http://i.imgur.com/JNqCXJk.png)

_Type `githubarchive`_
![](http://i.imgur.com/Ce0gCNj.png)

A new dataset should appear in the sidebar with three tables. We'll be using the "timeline" table which has data for every repository created since around 2008! Click on the "timeline" dataset to see the table shema and available query fields.

![](http://i.imgur.com/BmToZJh.png)

#### What to Query

For this graph, we're going compare any number of programming languages against each other based on how many repositories use each given language.

* each word
* how many times that word occured in a given year
* what year it was

Here is a query we could run:

```
SELECT LEFT(repository_created_at, 4) as year, repository_language AS language, count(DISTINCT repository_name) AS count
FROM [githubarchive:github.timeline]
WHERE (repository_language = "Python" OR repository_language = "Ruby")
and PARSE_UTC_USEC(repository_created_at) >= PARSE_UTC_USEC('2012-01-01 00:00:00')
GROUP BY year, language
ORDER BY year, count DESC
```

![](http://i.imgur.com/9aGqElA.png)

WOAH! What in the world is going on up there? Let me break it down for you. When we select `repository_created_at`, it returns a full UTC date, which has a lot of junk that we don't care about for this graph. We just want the year, so the `LEFT` function allows us to select only the 4 leftmost digits of this date -- the year. Then we select `repository_language` and count all of the __distinct__ `repository_name`'s so that each repository only gets counted once. We have to do this because the table records every event on GitHub; it doesn't just provide us with a list of repositories and their statistics. We don't want to count every event as a new repository, so we only count the distinct names as to limit each repositories count to 1. 

The `WHERE` clause has two parts. The first filters our results to the two (or however many you want!) languages that we want to compare. The second uses `PARSE_UTC_USEC` to put the `repository_created_at` value into a date formate. This allows us to change our timescale. In this example, we are only looking at repositories created in 2012 or later.

#### Download the Data

Now that you have the data, simply press the "Download CSV" button. If you open it with Excel, it should look like a nicely organized 3 column chart.

![](http://i.imgur.com/SlVfcAV.png)

### Graphing the Ngram

Now's the fun part! We've included a d3 template that will turn your data into a pretty graph. The only thing you need to be sure of is that you used the same column names as the example above and that the CSV file is saved in the same folder as the template.

- Download the template [here](https://github.com/Thinkful/guide-google-bigquery)
- Save your data in the save folder under the name of `github_data.csv`
- Open terminal, cd into that folder and run `pytnon -m SimpleHTTPServer`
- In your browser, navigate to `http://localhost:8000/` and your graph should pop up!

Try graphing lots of different combinations and let us know what you find :)

## Uploading your own data

Uploading your own data to the BigQuery is free, _but_ in order to be able to use this feature, Google requires that you enable billing.

You can see more detailed quotas and pricing [here](https://developers.google.com/bigquery/pricing), but it should be free for reasonable sized uploads and queries. Also, loading data into BigQuery is __free__! Querying data cost $5 per TB, but the first 1 TB of data processed per month is also __free__. Exporting data is also a __free__ service. So, if you just want to play around, don't worry about accumulating a large receipt (we haven't been charged a penny, and we've been using BigQuery for a few weeks). 

To upload your own data, simply press the little arrow next to your project name. Then choose the file you want to upload, click through the instructions, and voila! Once uploaded you'll need to create a schema for the table. Basically, you'll need to type out the name and data value for each column you uploaded. As an example, the Shakespeare dataset we just worked with would have the schema: `word:string, word_count:integer, corpus:string, corpus_date:integer`.

You now know how to navigate the wonderful world of Google BigQuery!
