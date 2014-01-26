---
title: Eigengoogle: How the Google PageRank Algorithm Works
kind: article
created_at: 2014-01-25
category: article
---


<span class="mj">-- IF THIS MESSAGE IS SHOWING, EQUATIONS HAVE NOT RENDERED. 
IF YOU ARE ON RSS, VISIT THE ORIGINAL POST. JAVASCRIPT REQUIRED. --</span>
<script type="math/tex">
</script>

While we're on the topic of [sorting things online](http://clusterfoo.com/articles/sorting/), 
we might as well talk about Google: 
the 93-billion dollar company whose main export is taking all
the things ever and putting them in the right order. If there's one thing Google
knows best, it's sorting stuff.

And it all started with an algorithm called PageRank. 
[According to Wikipedia](http://en.wikipedia.org/wiki/PageRank),

> Pagerank uses a model of a random surfer who gets bored after several 
clicks and switches to a random page. It can be understood as a Markov chain in 
which the states are pages, and the transitions are the links between pages. 
When calculating PageRank, pages with no outbound links are assumed to link 
out to all other pages in the collection (the random surfer chooses another 
page at random).
>
> The PageRank values are the entries of the dominant eigenvector of the 
modified adjacency matrix. 

![](/assets/images/2014/eigenvectors.png)

In this post I'll try to break that down and provide some of the background
necessary to understand Google PageRank.



### Graphs as Matrices

A graph is a connection of nodes joined by edges. If the edges are arrows
that flow in one direction, we call that a *directed graph*. A graph whose edges
have each been assigned a "weight" (usually some real number) is a *weighted
graph*.

![](/assets/images/2014/weighted_graph01.png)


A graph of `n` nodes can be represented in the form of an 
`n x n` *adjacency matrix*,
<span class="mj">`M = [m_ij]`</span>$$ M = [m_{ij}]$$ such that 
<span class="mj">m_ij</span>$$m_{ij}$$ is equal to the weight of the edge going
from node <span class="mj">j</span>$$j$$ to node
<span class="mj">i</span>$$i$$:

            
    [0, 1, 0, 0]
    [1, 0, 2, 0]
    [2, 1, 0, 1]
    [0, 0, 4, 0]
            



### Stochastic Matrices

The
term "stochastic" is used to describe systems whose state can only be described
in probabilistic terms (i.e: the likelihood of some event happening at any given
time).

> **Scenario**: 
Consider two competing websites. Every month, the first website loses 30% of its 
audience to
the second website, while the second website loses 60% of its audience to the first. 
>
> If the two websites start out with 50% of the global audience, how
many users will each website have after a month? After a year?

This scenario can be represented as the following system:

    P = [0.7, 0.6],    x_0 = [0.5, 0.5]    
        [0.3, 0.4]


![](/assets/images/2014/competing_stores_graph.png)

This is a *Markov chain* with *transition matrix* <span class="mj">`P`</span>$$P$$
and a *state vector* <span class="mj">`x_0`</span>$$\mathbf{ x^{(0)} }$$.

The transition matrix is called a *stochastic matrix*; it represents the
likelihood that some individual in a system will transition from one state
to another. The columns on a stochastic matrix are always non-negative numbers
that add up to 1 (i.e: the probability of *at least one*
of the events occurring is always 1 -- the likelihood of a user either staying
on the same website, or leaving, is always 100%. He must choose one of the two).

The state after the first month is 
<span class="mj">`x_1 = P*x_0 = [0.65, 0.35]`</span>
$$ \mathbf{ x^{ (1) } } = P \mathbf{ x^{ (0) } } = [0.65, 0.35]$$.

So, after the first month, the second website will have only 35% of the
global audience.

To get the state of the system after two months, we simply
apply the transition matrix again, and so on. That is, the current state of
a Markov chain depends only on its previous state.
Thus, the state vector at month $$k$$ can be defined recursively:

<span class="mj">`-- EQUATION NOT RENDERED IN RSS OR WITH JAVASCRIPT DISABLED --`</span>
<script type="math/tex; mode=display">
    \mathbf{ x^{(k)} } = P\mathbf{ x^{ (k - 1) } }
</script>

From which, through substitution, we can derive the following equation:

<span class="mj">`-- EQUATION NOT RENDERED IN RSS OR WITH JAVASCRIPT DISABLED --`</span>
<script type="math/tex; mode=display">
    \mathbf{ x^{(k)} } = P^k \mathbf{ x^{(0)} }
</script>

Using this information, we can figure out the state of the system
after a year, and then again after two years 
(using the [Sage](http://www.sagemath.org/) mathematical library for python):

    P = Matrix([[0.70, 0.60],
                [0.30, 0.40]])
    x = vector([0.5,0.5])
    P^12*x
    # -> (0.666666666666500, 0.333333333333500)
    P^24*x
    # -> (0.666666666666666, 0.333333333333333)
{: class="language-python" }

So it seems like the state vector is "settling" around those values. 
It would appear that, as <span class="mj">EQ</span>$$n \to \infty$$,
<span class="mj">EQ</span>$$P^n\mathbf{ x^{ (0) } }$$ is converging to some
$$\mathbf{ x }$$
such that <span class="mj">EQ</span>$$P\mathbf{ x } = \mathbf{ x }$$. 
As we'll see below, this is indeed the case.

We'll call this $$\mathbf{ x }$$ the *steady state vector*.


### Eigenvectors!

Recall from linear algebra that an eigenvector of a matrix $$A$$
is a vector $$\mathbf{x}$$ such that:


<span class="mj">`-- EQUATION NOT RENDERED IN RSS OR WITH JAVASCRIPT DISABLED --`</span>
<script type="math/tex; mode=display">
    A\mathbf{ x } = \lambda \mathbf{ x }    
</script>

for some scalar <span class="mj">EQ</span>$$\lambda$$ (the *eigenvalue*). A *leading eigenvalue* is an
eigenvalue <span class="mj">EQ</span>$$\lambda_{ 1 }$$ such that its absolute value is greater than 
any other eigenvalue of a given matrix. 

One method of finding the leading eigenvector of a matrix is through 
a [power iteration](http://en.wikipedia.org/wiki/Power_iteration) sequence, defined
recursively like so:


<span class="mj">`-- EQUATION NOT RENDERED IN RSS OR WITH JAVASCRIPT DISABLED --`</span>
<script type="math/tex; mode=display">
    \mathbf{ x_k } = \cfrac{ A\mathbf{ x_{ k-1 } } }{ \| A\mathbf{ x_{ k-1 } } \| }
</script>

Again, by noting that we can substitute 
<span class="mj">EQ</span>$$ A\mathbf{ x_{ k-1 } } =  A(A\mathbf{ x_{ k-2 } }) = A^2\mathbf{ x_{ k-2 } } $$, 
and so on, it follows that:

<span class="mj">`-- EQUATION NOT RENDERED IN RSS OR WITH JAVASCRIPT DISABLED --`</span>
<script type="math/tex; mode=display">
    \mathbf{ x_k } =  \cfrac{ A^k \mathbf{ x_0 } }{ \| A^k \mathbf{ x_0 } \| }
</script>

This sequence converges to the leading eigenvector of <span class="mj">EQ</span>$$A$$.

Thus we see that the steady state vector is just an eigenvector with the
special case <span class="mj">EQ</span>$$\lambda = 1$$.

### Stochastic Matrices that Don't Play Nice

Before we can finally get to Google PageRank, we need to make a few more observations.

First, it should be noted that power iteration has its limitations: 
not all stochastic matrices converge. Take as an example:

    P = Matrix([ [0, 1, 0],
                 [1, 0, 0],
                 [0, 0, 1]])

    x = vector([0.2, 0.3, 0.5])

    P * x
    # -> (0.3, 0.2, 0.5)
    P^2 * x
    # -> (0.2, 0.3, 0.5)
    P^3 * x
    # -> (0.3, 0.2, 0.5)
{: class="language-python" }

The state vectors of this matrix will oscillate in such a way forever. This 
matrix can be thought of
as the transformation matrix for reflection about a line in the x,y axis... this
system will never converge (indeed, it has no leading eigenvalue: 
<span class="mj">EQ</span>$$ |\lambda_1| = |\lambda_2| = |\lambda_3| = 1 $$).

Another way of looking at $$P$$ is by drawing its graph:

![](/assets/images/2014/oscillating_chain.png)

Using our example of competing websites, this matrix describes a system such that,
every month, *all* of the first website's users leave and join the seconds website,
only to abandon the second website again a month later and return to the first, 
and so on, forever.

It would be absurd to hope for this system to converge to a steady state.

States 1 and 2 are examples of *recurrent states*. These are states that,
once reached, there is a probability of 1 (absolute certainty) 
that the Markov chain will return to them infinitely many times. 

A *transient state* is such that the probability is <span class="mj">EQ</span>$$ > 0$$ that they will
never be reached again. (If the probability *is* 0, we call such a state 
*ephemeral* -- in terms of Google PageRank, this would be a page that no
other page links to):

![](/assets/images/2014/diffrent_states.png)

There are two conditions a transition matrix must meet if we want to ensure that
it converges to a steady state:

It must be *irreducible*: an irreducible transition matrix is a
matrix whose graph has no closed subsets. (A closed subset is such that no state
within it can reach a state outside of it. 1, 2 and 3 above are closed from
4 and 5.)

It must be *primitive*: 
A primitive matrix $$P$$ is such that, for some positive
integer <span class="mj">EQ</span>$$n$$, <span class="mj">EQ</span>$$P^n$$ is such 
that <span class="mj">EQ</span>$$p_{ ij } > 0$$ for all <span class="mj">EQ</span>$$p_{ ij } \in P$$
(that is: all of its entries are positive numbers).

> More generally, it must be *positive recurrent* and *aperiodic*. 
>
> Positive recurrence means that it takes, on average,
a finite number of steps to return to any given state. Periodicity means the
number of steps it takes to return to a particular state is always divisible
by some natural number $$n$$ (its period). 
>
> Since we're dealing
> with finite Markov chains, irreducibility implies positive recurrence, and 
primitiveness ensures aperiodicity.

![](/assets/images/2014/periodic.png)

### Google PageRank

We are now finally ready to understand how the PageRank algorithm works. Recall
from Wikipedia:

> The formula uses a model of a random surfer who gets bored after several clicks 
and switches to a random page. The PageRank value of a page reflects the chance 
that the random surfer will land on that page by clicking on a link. It can be 
understood as a Markov chain in which the states are pages, and the transitions, 
which are all equally probable, are the links between pages.

So, for example, if we wanted to represent our graph above, we would start 
with the following adjacency matrix:

    [0, 0, 0.5, 0,   0],
    [0, 0, 0.5, 0.5, 0],
    [1, 1, 0,   0,   0],
    [0, 0, 0,   0,   0],
    [0, 0, 0,   0.5, 0]

For the algorithm to work, we must transform this original matrix in such a way
that we end up with an irreducible, primitive matrix. First,

> If a page has no links to other pages, it becomes a sink and therefore 
terminates the random surfing process. If the random surfer arrives at a sink 
page, it picks another URL at random and continues surfing again.
>
> When calculating PageRank, pages with no outbound links are assumed to link 
out to all other pages in the collection.


            [0, 0, 0.5, 0,   0.2],
            [0, 0, 0.5, 0.5, 0.2],
    S =     [1, 1, 0,   0,   0.2],
            [0, 0, 0,   0,   0.2],
            [0, 0, 0,   0.5, 0.2]

We are now ready to produce $$G$$, the Google Matrix, which is both irreducible
and primitive. Its steady state vector gives us the final PageRank score for
each page. 



### The Google Matrix

The [Google Matrix](http://en.wikipedia.org/wiki/Google_matrix) 
for an $$n \times n$$ matrix $$S$$ is derived from the equation

<span class="mj">`-- EQUATION NOT RENDERED IN RSS OR WITH JAVASCRIPT DISABLED --`</span>
<script type="math/tex; mode=display">
    G = \alpha S + (1 - \alpha) \frac{1}{n} E
</script>

Where $$E = \mathbf{ e }\mathbf{ e }^T$$ is an $$n \times n$$ matrix whose entries are all 1, and
$$0 \le \alpha \le 1$$ is referred to as the *damping factor*. 

If $$\alpha = 1$$, then $$G = S$$. Meanwhile, if $$\alpha = 0$$ all of the entries
in $$G$$ are the same (hence, the original structure of the network is
"dampened" by $$\alpha$$, until we lose it altogether).

So the matrix $$(1 - \alpha) \frac{1}{n} E$$ is a matrix that
represents a "flat" network in which all pages link to all pages, and the user is
equally likely to click any given link (with likelihood $$\frac{ (1-\alpha) }{ n }$$),
while $$S$$ is dampened by a factor of $$\alpha$$.

> Google uses a damping factor of around 0.85. If you would like to research
further, [this paper](http://ilpubs.stanford.edu:8090/582/1/2003-20.pdf) 
is a good place to start and is quite readable. 
> 
> **tl;dr:** the second eigenvalue
of a Google matrix is $$|\lambda_2| = \alpha \le |\lambda_1| = 1$$ , and the rate of convergence
of the power iteration is given by $$\frac{ |\lambda_2| }{ |\lambda_1| } = \alpha$$.
So higher values of $$\alpha$$ imply better accuracy but worse performance. 0.85
is a good compromise between the two.

With some elementary algebra we can see that

<span class="mj">`-- EQUATION NOT RENDERED IN RSS OR WITH JAVASCRIPT DISABLED --`</span>
<script type="math/tex; mode=display">
    \left(\alpha s_{ 1j } + \frac{1-\alpha}{ n }\right) + \left(\alpha s_{ 2j } 
    + \frac{1-\alpha}{ n }\right) + ... + \left(\alpha s_{ nj } + \frac{1-\alpha}{ n }\right) = 1
</script>

For all $$j$$ up to $$n$$, which means that $$G$$ is indeed stochastic, 
irreducible, and primitive.

In conclusion,

![Imgur](/assets/images/2014/eigensnotsicles.png)
