+++
title = "Basics of Probability and Random Variable"
date = 2019-05-13
tags = ["probability distribution", "random variable"]
categories = ["statistics", "probability"]
draft = false
author = "Peter Lo"
+++

Roughly speaking, probability describes how likely a certain event
takes place, and probability distribution refers to how likely a
random variable takes different values. In this post, we look at some
examples to get an intuitive idea of probability, sample space,
events, random variable and probability space.


## Probability {#probability}

_Probability_ is used to quantitatively describe how _likely_ an
_event_ would occur. A _probability_ is a number between (inclusive) 0
and 1, where a larger value means the event is more _likely_.


### Frequentist and Bayesian interpretation {#frequentist-and-bayesian-interpretation}

There are two slightly different interpretations of probability:
_frequentist_ and _Bayesian_ interpretation.

> Frequentist interpretation:
>
> The probability of an event is the limiting _relative frequency_ that
> the event occurs if the number of trials approaches infinity.

For example, under the frequentist interpretation, if we say the
probability of coin toss giving a tail is 0.6, what is meant is that
_if_ we flip the coin a _huge_ number of times, 60% of the flips would
result in a tail.

Note that this interpretation requires that it is at least conceivable
to repeat the trial to calculate the relative frequency that the event
occurs. For example, if we were to make sens of the "probability that
the moon is made of cheese", we might need to imagine the moon gets
repeatedly created, and calculate the proportion where the moon is
made of cheese.

On the other hand, the Bayesian interpretation of probability is more
liberal.

> Bayesian interpretation:
>
> The probability of an event is the _degree of belief_ that it occurs.

Under the Bayesian interpretation, we can speak of _subjective_
probability, in addition to /objective probability, and there is no
difficulty of making sense of the "probability that the moon is made
of cheese". Also, by the _Law of large number_, the relative frequency
of an event will tend to the objective probability if the number of
trials tends to infinity.

In practice, it does not matter much which interpretation you
subscribe to, because the formulas related to probability are mostly
the same (sometimes frequentists refuse to give interpretation to some
formulas in some cases). The difference is more philosophical than
practical.


## Sample space and events {#sample-space-and-events}

Whether we use frequentist or Bayesian interpretation of probability,
one key ingredient is _event_. For an event to make sense, it suffices
to have a clear criteria to determine whether an event has
occurred. For example, if we throw a dice, we may consider the
following events:

-   event A: the number is 2
-   event B: the number is even
-   event C: the number is odd
-   event D: the number is less than 3
-   event E: the number is both even and odd

We note that the above events are related:

-   Since 2 is even and is less than 3, event A implies both event B and event D.
-   Event C and D have some overlap (e.g. the number 1 is both odd and less than 3), but neither one implies the other.
-   Also, an integer is either even or odd, but not both, so event B and event C are _exclusive_, and together they are _exhaustive_.
-   Moreover, event E is an impossible event.

When modeling a situation, we can formally define a _sample space_,
which could be regarded as the set of atomic events, and subsets of
the sample space are the events. Note that we may model the same
situation with different sample spaces, but some sample space may be
more intuitive than others.

Events can give rise to other events, for example:

-   event E above is "event B _and_ event C";
-   event B is "_not_ event C" since an integer is either even or odd;
-   event D is "the number is less than 3", which is the same as "the number is 1 _or_ the number is 2" in this example.

In general:

-   the whole sample space (the set of all atomic events) is always an event, this is an event that is sure to happen
-   if A is an event, then "not A" is also an event
-   if A1, A2, ... are (finite or _countably infinite_ many) events, then "A1 _or_ A2 _or_ ..." and "A1 _and_ A2 _and_ ..." are also events


### Example: Dice throwing {#example-dice-throwing}

For instance, in the dice throwing example, we could define the sample
space as {1, 2, 3, 4, 5, 6}, then the 4 events above are respectively:

-   event A (the number is 2): {2}
-   event B (the number is even): {2, 4, 6}
-   event C (the number is odd): {1, 3, 5}
-   event D (the number is less than 3): {1, 2}
-   event E (the number is both even and odd): {}


### Example: Coin tossing {#example-coin-tossing}

As another example, for tossing a coin once, we may define the sample
space as {H, T} for "Head" and "Tail". Note that the sample space can
consist of any object, not just numbers. Also note that by defining
the sample space as {H, T}, we are implicitly excluding the
possibility that the coin may land on its edge, and therefore giving
neither a "Head" nor a "Tail".


### Example: Multiple coin tossings {#example-multiple-coin-tossings}

We may also model multiple coin tossings. E.g. if we model 3 coin
tossings, we may define the sample space as {(H,H,H), (H,H,T),
(H,T,H), (H,T,T), (T,H,H), (T,H,T), (T,T,H), (T,T,T)}, then we may
consider events such as:

-   "two heads out of 3 tosses": {(H,H,T), (H,T,H), (T,H,H)}
-   "the first two tossings give the same results": {(H,H,H), (H,H,T), (T,T,H), (T,T,T)}

We can even model infinitely many coin tossings, e.g. by defining the
sample space as \\(\\{(x\_1, x\_2, \ldots): x\_i \in \\{H,T\\}\\}\\). Then we may
consider events such as:

-   "the first toss is head": \\(\\{(H,x\_2,x\_3,\ldots): x\_i \in \\{H,T\\}\\}\\)
-   "it takes 3 tosses to get the first head": \\(\\{(T,T,H,x\_4,x\_5,\ldots): x\_i \in \\{H,T\\}\\}\\)
-   "head and tail appears alternately": \\(\\{(H,T,H,T,\ldots), (T,H,T,H,\ldots)\\}\\)


### Example: Time until a newly manufactured light bulb fails {#example-time-until-a-newly-manufactured-light-bulb-fails}

For modeling time until some events occur, we may define the sample
space as \\(\\{x: x>0\\}\\). For example, the time (in hours) until a newly
manufactured light bulb fails. Although we can measure "time" only up
to a certain precision, "time" is conceptually _continuous_, and
therefore often modeled as a continuous real value. We may then
consider events such as:

-   "the light bulb will not fail in the first 10,000 hours": \\(\\{x: x>10 000\\}\\)
-   "the light bulb will not last more than 100,000 hours": \\(\\{x: 0 < x \leq 100 000\\}\\)

On the other hand, in some cases, modeling the time as integers may be
sufficient, e.g. when we want to count the number of days of a patient
staying in the hospital, but do not want to consider fractional number
of days. In such case, we may take the sample space as {0, 1, 2, ...}.


### Example: Height and weight {#example-height-and-weight}

For height and weight of a person, similar to time, although we can
measure height and weight only up to a certain precision, they are
conceptually continuous, and therefore often modeled as continuous
real values. We may define the sample space as \\(\\{(h,w): h>0, w>0\\}\\)
(suppose we measure height h in "m", and weight w in "kg"). We may then
consider events such as:

-   "the height is below 1.6m": \\(\\{(h,w): 0 < h < 1.6, w>0\\}\\)
-   "the BMI is above 20, but below 24, where BMI is the weight in kg divided by the square of the height in m": \\(\\{(h,w): 20 < \frac{w}{h^2} < 24\\}\\)


## Probability space {#probability-space}

A _sample space_, the _events_, together with the probabilities of the
events, is called a _probability space_.

As we see from the examples above, some events in a sample space may
be related, it is then unsurprising that their probabilities are also
related. Probabilities are very similar to _measure_ such as
area.

Suppose there is a region (e.g. a square) with area 1.

-   Then intuitively, any smaller regions inside would have area less than or equal to 1.
-   In fact, if a region A is entirely within another region B, the area of A is not larger than the area of B.
-   Also, the total area of two non-overlapping regions is the sum of the areas of the two regions.

In a probability space:

-   the probability of the whole sample space is 1.
-   And if events A and B are exclusive of each other (have no overlaps, i.e. no common atomic events), then the probability of "A or B" is the sum of probability of A and probability of B.
-   In fact, this summation rule works for countably infinitely many exclusive events.
-   If event C is a subset of event D (event C implies event D, i.e. every atomic event in C is also in D), then the probability of C is less than or equal to the probability of D.
-   Note that even if D contains all atomic events of C, and D has some other atomic events not in C, the probability of D may still be the same as the probability of C, because the extra events may have probability zero.
-   Since any event E and "not E" are exclusive, and "E or (not E)" is the whole sample space, "probability of E" plus the "probability of not E" is equal to the probability of the sample space, which is 1. Consequently, "probability of not E" is 1 minus the "probability of E".
-   The probability is often denoted as \\(P(.)\\), e.g. \\(P(\text{event B}) = P(\\{2, 4, 6\\})\\) denotes the probability of event B in the dice throwing example above.

If the sample space is finite or countably infinite, the events are
usually all the possible subsets of the sample space, and by the
summation rule above, if we know the probabilities of each atomic
event, we can get the probability of any event by summation. For
example, we have \\(P(\\{2, 4, 6\\}) = P(\\{2\\}) + P(\\{4\\}) + P(\\{6\\})\\).

But if the sample space is uncountably infinite, the events often have
uncountably infinitely many elements, we cannot simply use the
summation rule to get the probability of an event from the sum of the
atomic events, because the rule works only for a sum of countably many
terms. We would not go into further details of such technical issues,
but it should be emphasized that the above rules still work, and we
can nevertheless define a consistent probability measure on all the
events such as \\(\\{x: x> 10 000\\}\\) and \\(\\{x: 500 \leq x < 2000\\}\\).


## Random variable {#random-variable}

From a sample space, there may be many quantities that may be of
interest. In the above examples, we have quantities such as

-   "number of heads out of 3 tosses"
-   "number of tosses to get the first head"
-   BMI, which is calculated from the weight and height

These are examples of _random variable_. Note that the values of these
quantities can all be determined by knowing which atomic event has
occurred. More formally, a _random variable_ is a _function_ from the
atomic events of a sample space to a value (which could be any object,
not just numbers). Let's look at some examples of random variables
that can be defined on the above sample spaces:


### Example: Dice throwing {#example-dice-throwing}

| Atomic event | 1  | 2   | 3 | 4  | 5    | 6 |
|--------------|----|-----|---|----|------|---|
| \\(X\_1\\)   | 1  | 2   | 3 | 4  | 5    | 6 |
| even         | 0  | 1   | 0 | 1  | 0    | 1 |
| odd          | 1  | 0   | 1 | 0  | 1    | 0 |
| \\(X\_2\\)   | 6  | 5   | 4 | 3  | 2    | 1 |
| \\(X\_3\\)   | A  | A   | B | B  | C    | C |
| \\(X\_4\\)   | 20 | 712 | 4 | -8 | 6466 | 0 |

-   Note that we may define multiple random variables on the same sample space, and the relationship between the atomic event and the value of the random variable can be arbitrarily complex, as long as for each atomic event, the random variable takes only a single value.
-   \\(X\_1\\) is simply the atomic event itself.
-   "even" and "odd" takes only the values 0 and 1, and are usually called _indicator_ variables, which assume the value 1 for a particular event of interest, and 0 otherwise.
-   \\(X\_1\\) and \\(X\_2\\) both takes the values 1 to 6, but on different atomic events.
-   \\(X\_1\\) and \\(X\_2\\) are also related in that \\(X\_1 + X\_2 = 7\\) in any case.
-   \\(X\_3\\) is an example of random variable that is not numeric, but here it takes the values "A", "B" or "C".
-   For \\(X\_4\\), you need not try to guess a succinct formula between the atomic event and its value, because I just hit the number pad randomly to get these numbers.
-   Nevertheless, \\(X\_4\\) is a perfectly valid random variable defined on the above sample space.


### Example: Coin tossing {#example-coin-tossing}

| Atomic event | H | T |
|--------------|---|---|
| \\(y\_1\\)   | 1 | 0 |
| \\(Y\_2\\)   | 0 | 1 |
| \\(Y\_3\\)   | 1 | 1 |

-   \\(Y\_1\\) is simply an indicator for "head" and \\(Y\_2\\) is an indicator for "tail".
-   Note that \\(Y\_3\\) takes the same value 1 on the different atomic events, so it is a _constant_.
-   It may seem strange that a constant would be called a _random variable_, because it is not at all _random_, and does not _vary_. But it satisfies the definition of _random variable_, though sometimes a constant may be called a degenerate random variable.


### Example: Multiple coin tossing {#example-multiple-coin-tossing}

We may define a random variable \\(Z\_1\\) as the number of tosses until
the first "head" (including the toss giving the first "head"), more
precisely, \\(Z\_1 = \min \\{t: x\_t = H\\}\\). Note that \\(Z\_1\\) can
potentially take any integer greater than 0. For example, \\(Z\_1 = 1\\) on
all of \\(\\{(H,\ldots)\\}\\), and \\(Z\_1 = 2\\) on all of \\(\\{(T,H,\ldots)\\}\\).

We may define another random variable \\(Z\_2\\) as the number of heads in
the first 100 tosses, then the possible values of \\(Z\_2\\) are integers
from 0 to 100. Note that even though each element of the sample space
is infinitely long (representing infinitely many tosses), the value of
\\(Z\_2\\) could be determined from the first 100 tosses. If we only want
to model \\(Z\_2\\), we may as well use a (finite but very large) sample
space such as \\(\\{(x\_1, x\_2, \ldots, x\_{100}): x\_i \in \\{H,T\\}\\}\\).


### Example: Time until a newly manufactured light bulb fails {#example-time-until-a-newly-manufactured-light-bulb-fails}

In this sample space \\(\\{x: x>0\\}\\), we may define a random variable
\\(W\_1\\) as the time (in hours) until a newly manufactured light bulb
fails, which would simply take the value of the atomic event.

We may also define another random variable \\(W\_2\\) as a discretized
version of \\(W\_1\\) by taking only the integer part, i.e. \\(W\_2 = 0\\) if \\(0
\leq W\_1 < 1\\), and \\(W\_2 = 1\\) if \\(1 \leq W\_1 < 2\\) and so on. In
general, we may discretize a continuous random variable by whatever
break points we like.

We may define a vector valued random variable \\(W\_3 = (t, d)\\), where \\(d
= 0\\) if \\(x > 10000\\), and \\(d = 1\\) if \\(x \leq 10000\\); \\(t = min(x,
10000)\\). Note that the two elements of the vector are related.


### Example: Height and weight {#example-height-and-weight}

| Atomic event | \\((h,w)\\)              |
|--------------|--------------------------|
| \\(V\_1\\)   | h                        |
| \\(V\_2\\)   | w                        |
| \\(V\_3\\)   | \\(\frac{w}{h^2}\\)      |
| \\(V\_4\\)   | \\((w, \frac{w}{h^2})\\) |

-   Here \\(V\_1\\) is simply the height, and \\(V\_2\\) is the weight, and \\(V\_3\\) is the BMI.
-   Note that the value of \\(V\_4\\) is a vector of the weight and the BMI.
-   In general, the value of a random variable can be even more complicated objects.


### Distribution of Random Variable {#distribution-of-random-variable}

Loosely speaking, the distribution of a random variable refers to how
likely (i.e. the probability) it would take various values or ranges
of values.

Let's consider the dice throwing example. What is the probability that
the random variable "even" is 1, i.e. what is \\(P(\text{even} = 1)\\)? We
see that \\(\text{even} = 1\\) exactly when the atomic events are either
2, 4 or 6, intuitively, \\(P(\text{even} = 1) = P(\\{2, 4, 6\\})\\), which
is equal to \\(P(\\{2\\}) + P(\\{4\\}) + P(\\{6\\})\\).

As another example, what is the probability that \\(X\_3\\) is equal to
"B", i.e. what is \\(P(X\_3 = \text{B})\\)? We see that \\(X\_3 = \text{B}\\)
exactly when the atomic event is either 3 or 4, so \\(P(X\_3 = \text{B})
= P(\\{3, 4\\})\\), which is equal to \\(P(\\{3\\}) + P(\\{4\\})\\).

We can also find the probability that a random variable takes a range
of values. For example, to find \\(P(X\_2 \geq 4)\\), we first determine
that \\(X\_2 \geq 4\\) exactly when the atomic event is either 1, 2 or 3,
so \\(P(X\_2 \geq 4) = P(\\{1, 2, 3\\})\\), which is equal to \\(P(\\{1\\}) +
P(\\{2\\}) + P(\\{3\\})\\).

For continuous random variables, the process of determining the
probability is similar. For the height and weight example, to find
\\(P(40 \leq V\_2 \leq 60)\\), we find the subset of the sample space for
which \\(40 \leq V\_2 \leq 60\\), i.e. \\(\\{(h, w): 40 \leq V\_2 \leq 60\\} =
\\{(h, w): 40 \leq w \leq 60, h > 0\\}\\). So \\(P(40 \leq V\_2 \leq 60) =
P(\\{(h, w): 40 \leq w \leq 60, h > 0\\})\\).

As another example, \\(P(20 \leq V\_3 \leq 24) = P(\\{(h, w): 20 \leq V\_3
\leq 24\\})\\), which is equal to \\(P(\\{(h, w): 20 \leq \frac{w}{h^2} \leq
24\\})\\), although it is not easy to visualize this set here.

In general, since the atomic event determines the value of a random
variable, in order to find the probability that the random variable
takes a value or a range of values, we first determine the set of
atomic events that give rise to these values, then determine
probability of this set of atomic events, which would be well-defined
exactly when this set is also an event. Technically, we would require
the random variable to be _measurable_ with respect to the probability
space, i.e. the sets of atomic events corresponding to various values
of the random variable are events, but we would not go into details.

We see that the distribution of a random variable is induced from the
underlying probability space, but if we are focusing only on one
random variable, we often only describe the probability distribution
of the random variable, without mentioning the underlying sample
space. For example, for the \\(X\_3\\) in the dice throwing example, since
it can only take the values "A", "B" or "C", when only looking at the
value of \\(X\_3\\), we can know which subset of atomic events has
occurred, but cannot pinpoint exactly which atomic event has
occurred. For instance, knowing that \\(X\_3 = \text{A}\\), we only know
that the event is \\(\\{1, 2\\}\\), but cannot pinpoint whether the event is
1 or 2. Therefore, we may summarize the distribution of \\(X\_3\\) by
specifying only \\(P(X\_3 = \text{A})\\), \\(P(X\_3 = \text{B})\\) and \\(P(X\_3 =
\text{C})\\). We would look at more examples in the next post.


## Summary {#summary}

In this post, we have looked at the following concepts:

-   probability: either the degree of belief of an event, or the limiting relative frequency of an event when the number of trials approaches infinity.
-   sample space: the set of atomic events.
-   event: subset of the sample space.
-   probability space: a sample space, the events, and a probability measure on the events.
-   random variable: a function from the sample space to a value.

In the next post, we would look at more examples of distributions of
discrete and continuous random variables, and the parametric way of
specifying distributions.