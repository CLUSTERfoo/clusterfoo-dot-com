---
title: Sorting Posts by User Engagement Level
kind: article
created_at: 2014-01-20
category: article
---

*The equations in this post require Javascript and will not 
render if you are on RSS or email.*

At Functional Imperative we're building the new CANLII Connects website
(a social portal for CANLII, a database of Canadian legal cases), and 
this week I was given the task of figuring out a sensible way of sorting posts.

This is a common problem that many social websites face, and researching how 
others (Reddit, Hacker News, Digg) have done it before was quite interesting.

Here's Reddit's [scoring function for 'Best'](http://www.evanmiller.org/how-not-to-sort-by-average-rating.html).
Of course, not all scoring functions are that hairy, 
[here are a few more](http://moz.com/blog/reddit-stumbleupon-delicious-and-hacker-news-algorithms-exposed).

Interestingly enough, Reddit's 'Hot' scoring function:

![Reddit's scoring equation](/assets/images/2014/reddit_hot_algo.png)

is 
[quite flawed](http://technotes.iangreenleaf.com/posts/2013-12-09-reddits-empire-is-built-on-a-flawed-algorithm.html).

Anyway, while all those scoring functions work pretty well, they didn't quite
fit the requirements for CANLII Connects.

In this post, I'll walk through the decision progess in creating such a scoring
function. Hopefully this will be useful if you encounter a similar feature to
implement.

### Requirements:

CANLII Connects links to a databse of legal cases, and users can post opinions
on those cases. Moreover:

1. A user can post an opinion.
2. A user can upvote an opinion.
3. A user can comment on an opinion.
4. A user can comment on a comment on an opinion.

So what's a sensible way of sorting opinions?

Right away, we're dealing with a different problem than Reddit or HN: while
it makes sense to slowly degrade the score of a post on those sites over time,
the same does not make sense for CANLII. Old cases might be cited at any time, 
no matter how old they are, so what matters is not how old a discussion is, but
rather how actively engaged users are within a given discussion.

### Working it out...

Ok, so first we have to give post an initial score. I like Reddit's approach
of taking the base-10 log of its upvotes. This makes sense because, the more 
popular a post already is, the more likely people are to see it, and therefore
upvote it, which gives it an unfair advantage. 

