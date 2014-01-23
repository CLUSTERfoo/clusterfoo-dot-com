---
title: Sorting Posts by User Engagement Level (With Elasticsearch Implementation)
kind: article
created_at: 2014-01-23
category: article
---

At Functional Imperative we're building the new *CanLII Connects* website
(a social portal for Canada's largest database of legal cases), and 
this week I was given the task of figuring out a sensible way of sorting posts.

Figuring out how to sort user-generated content is
a common problem that many social websites face.

Here's Reddit's scoring equation for 'Best' [(source and explanation)](http://www.evanmiller.org/how-not-to-sort-by-average-rating.html):


![Reddit's 'best' scoring equation](http://clusterfoo.com/assets/images/2014/reddit_best.png)

Not all scoring equations are that hairy, 
[here are a few more](http://moz.com/blog/reddit-stumbleupon-delicious-and-hacker-news-algorithms-exposed).

Interestingly enough, Reddit's 'Hot' scoring function (explained in link above):

![Reddit's scoring equation](http://clusterfoo.com/assets/images/2014/reddit_hot_algo.png)

is [quite flawed](http://technotes.iangreenleaf.com/posts/2013-12-09-reddits-empire-is-built-on-a-flawed-algorithm.html).

> **Sidenote**: One observation not mentioned in that first article 
is that, while all other equations use some form of 
`time_now - time_posted` to calculate how old a post is, the clever guys at 
Reddit use `time_posted - some_old_date`.
> 
> The advantage of this is that the post's score need only be calculated once,
whereas the value of scores calculated with `time_now` will change on every 
request.

Anyway, while all those scoring functions work pretty well, they didn't quite
fit the requirements for *CanLII Connects*.

In this post, I'll walk through the decision process of creating a scoring
function. Hopefully this will be useful if you encounter a similar feature to
implement.



### Requirements:

*CanLII Connects* links to a database of legal cases, and users can post opinions
on those cases. Moreover:

1. A user can post.
2. A user can upvote an post.
3. A user can comment on an post.
4. A user can comment on a comment on an post.

So what's a sensible way of sorting posts?

Right away, we're dealing with a different problem than Reddit or HN: while
it makes sense to slowly degrade the score of a post on those sites over time,
the same does not make sense for CanLII. Old cases might be cited at any time, 
no matter how old they are, so what matters is not how old a discussion is, but
rather how actively engaged users are within a given discussion.



### Initial Score

Ok, so first we have to give each post an initial score. I like Reddit's approach
of taking the base-10 log of its upvotes. This makes sense because, the more 
popular a post already is, the more likely people are to see it, and therefore
upvote it, which gives it an unfair advantage. 

So <span class="mj">`log_10(u)`</span>$$\log_{10} u$$ is a good start. 
In our case, we're not only trying to
measure how much people "like" a post, but rather how engaged they are with it.
It makes sense that, while an upvote is a fine indicator of "engagedness", a
user actually bothering to comment on a post is even more of an indicator. I'll
count that as equivalent to two upvotes. And if a user bothers to not only
comment on a post, but read the other comments and then comment on one of
those? Well, clearly that user is very engaged. 
I'd say that's worth three upvotes:


<span class="mj">`log_10(u + 2c + 3cc)`</span>
<script type="math/tex; mode=display">
    log_{10}(u + 2c + 3cc)
</script>



### Frequency

Next, we need the post's position to degrade as it becomes less active. It makes
sense to divide the intial score by some factor of time:



<span class="mj">`log_10(score)/t_ave`</span>
<script type="math/tex; mode=display">
    \cfrac{ log_{10} u+2c+3cc }{ \bar{t} }
</script>



Now we need a reasonable value for <span class="mj">`t_ave`</span>$$ \bar{ t } $$.
A good start is the average time, in seconds, between the three most
recent user interactions with  a post. 

We define a user interaction to be: a user creates a post, a user comments on
a post, or a user upvotes a post.

Also, we want the most recent interactions to weigh more than older interactions
(if a post is slowing down, or speeding up, we want the value of `t_1` to be 
higher than that of `t_3`). So let's say each `t` weighs twice as much
as the previous. What we get is <span class="mj">`t_ave`</span>$$ \bar{ t }$$
equal to the weighed average of the previous three interaction times:



<span class="mj">
`t_ave = sum(0.5^i-1 * (t_i - t_i-1)) / sum(o.5^i-1) for i = 1..3 
-- ENABLE JAVASCRIPT TO VIEW RENDERED EQUATION`
</span>
<script type="math/tex; mode=display">
    \bar{ t } = \cfrac{\sum_{i=1}^3 \left(\frac{1}{2}\right)^{i-1} * (t_i - t_{i-1}) }{ \sum_{i=1}^3  \left(\frac{1}{2}\right)^{i-1}}
</script>



Where

<span class="mj">`t_0`</span>$$ t_0 $$ = [UNIX timestamp](http://en.wikipedia.org/wiki/Unix_time), 
at now, in seconds.

<span class="mj">`t_n`</span>$$ t_n $$ = [UNIX timestamp](http://en.wikipedia.org/wiki/Unix_time) 
of n<sup>th</sup> interaction.

### One Final Detail

There is one last property I wanted this function to have, which is the following:
if interactions are very frequent right now (within a timeframe of, say, 10 days), 
then clearly the post is "hot", and its score should be boosted. But as more
and more time passes, it really doesn't matter as much how much distance there 
is between interactions. If a post has already gone a full year without anyone
commenting on it, does it 
really make that much difference if it goes another month without a comment?

To accomplish the first property, all we do is divide <span class="mj">`t_ave`</span>$$ \bar{ t }$$
by the number of seconds in 10 days: `60*60*24*10`.

To accomplish the second property, what we are looking for is some sort of
always-increasing, concave function (positive derivative, negative second derivative).
The first thing that comes to mind is the square-root function, which is good enough.

### Result

And thus we have our final scoring function:



<span class="mj">
`log(u + 2c + 3cc)/(sqrt(t_ave/60*60*24*10)); t_ave defined above -- ENABLE JAVASCRIPT FOR PRETTY EQUATIONS`
</span>
<script type="math/tex; mode=display">
    \cfrac{ log_{10} (u + 2c + 3cc) }{ \sqrt{ \bar{t}/60*60*24*10 }}
</script>

$$ \bar{ t } = \cfrac{\sum_{i=1}^3 \left(\frac{1}{2}\right)^{i-1} * (t_i - t_{i-1}) }{ \sum_{i=1}^3  \left(\frac{1}{2}\right)^{i-1}} $$




If we plot this equation for `x = number of points` and `y = time`, we can see the
shape of this function and check for different values if they make sense:

![Scoring function 3D plot](http://clusterfoo.com/assets/images/2014/scoring_function_shape.jpg)


As expected, there is a steep 10-day "boost" period, followed by an increasingly 
slower decline in the value as more and more time passes.



### Bonus Round: Implementing in ElasticSearch

Implementing a custom scoring function in Elasticsearch, though easy once it's
all set up, was rather frustrating because of the poor documentation.

For our implementation, we're using the `tire` gem (a wrapper around the 
Elasticsearch API). This is where we call the custom scoring script:

    query do
      #custom_score script: "parseInt(doc['upvote_count'].value)", lang: "javascript" do
      custom_score script: script, lang: 'javascript' do
        string query.join(" OR ")
      end
    end
{: class="language-ruby" }

Where `script` is simply a variable holding the contents of a javascript file
as a string. Note the option `lang: 'javascript'`. This lets us use javascript
as our language of choice, as opposed to [mvel](http://mvel.codehaus.org/), the
most poorly documented scripting language on the face of the earth. To enable 
this option, we'll also require the [elasticsearch-lang-javascript](https://github.com/elasticsearch/elasticsearch-lang-javascript) 
plugin.


Here is our script:

> **Sidenote:** Notice the logger function. This enables us to implement a sort of "console.log"
which we can read using the following shell command `tail -f /var/log/elasticsearch/elasticsearch.log`: 

    // Logger function:
    var logger = org.elasticsearch.common.logging.Loggers.getLogger("rails_logger");
    // Example usage:
    logger.info("========= NEW CALC ===========");

    var points_log = parseFloat(doc.points_log.value);
    var now = Math.round(new Date().getTime() / 1000);

    /**
    * NOTE: doc.ts.values is not actually an array,
    * here I create an array out of it:
    **/
    var ts = [];
    for (var i = 0; i < doc.ts.values.length; i++) ts[i] = doc.ts.values[i];
    ts.push(now);
    // Newest first
    ts.reverse();

    /**
    * Boost period.
    **/
    var ten_days = 60*60*24*10;

    /**
    * The scoring function
    **/
    function score() {
      /**
      * Weighed average numerator
      **/
      var times_num = (function() {
        var val = 0;
        for (var i = 1; i < ts.length; i++) {
          var exp = i - 1;
          val += Math.pow(0.5, exp) * 
                 (parseFloat(ts[i]) -
                 parseFloat(ts[i - 1]));
        }
        return val;
      })();

      /**
      * Weighed average denominator
      **/
      var times_denom = (function() {
        var val = 0;
        for (var i = 1; i < ts.length; i++) {
          var exp = i - 1;
          val += Math.pow(0.5, exp);
        }
        return val;
      })();

      var t_ave = (times_num/times_denom);

      return points_log/Math.sqrt(t_ave/ten_days);
    };

    score();
{: class="language-javascript" }
