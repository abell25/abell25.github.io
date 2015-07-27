---
layout: post
title: "probability puzzle"
date: 2015-07-26 21:11:48 -0500
comments: true
categories: 
---



# Probability Puzzle

I found an interesting puzzle [here](http://www.wallstreetoasis.com/forums/hardest-probability-and-statistics-interview-questions) that I thought I would go over to refresh my probability skillz.
> Here is the puzzle: 
> 
*Let's say A keep tossing a fair coin, until he get 2 consecutive heads, define X to be the number of tosses for this process; B keep tossing another fair coin, until he get 3 consecutive heads, define Y to be the number of the tosses for this process.*
>
>**Calculate P(X>Y).**
>

In Part 1 We will look at how to calculate probabilities of binary sequences ending with some pattern.  In Part 2 we will use these sequence probabilities to solve the main question: $P(X>Y) = ?$.

## Part 1. Coin flip sequences
The first question to answer is "What is the probability that coin A is flipped N times?"

A coin that is flipped until 2 heads appears has a sequence that does not have 2 heads as a subsequence.  In other words `THTTHH` and `TTTTHH` are valid sequences of length 6 for A but not `THHTHH` nor `HHHTHH`.  We can model this as a markov model:

![enter image description here](https://lh3.googleusercontent.com/-5vI1Wl8s-3Q/VbVto8ip6QI/AAAAAAAAAI8/lPhGIMHE0Xw/s0/Selection_089.png "HH.png")

When we are flipping the coin $A$, with probability $1/2$ we will get a tail and start-over.  With probability $1/2$ we will get a heads and move to the next state of the graph where we have equal probability of terminating and starting over.  We can write this as a recursive relation and solve for $E[X]$ directly:
$$
E[X] = \frac142 + \frac14(2 + E[X]) + \frac12(1 + E[X])
$$
This equation basically says: 

*With probability 1/4 we will be done (with 2 rolls), with probability 1/4 we will use 2 rolls and start over again anyways, and with probability 1/2 we will use 1 roll and start over again.*  The key to this equation is that when you start over, your probability does not change from the previous time you were in the start state. Now solving for $E[X]$:


![enter image description here](https://lh3.googleusercontent.com/-hOKvK71XD4A/VbWhE3nRZDI/AAAAAAAAAJo/cHqKIhqo3vs/s0/Selection_092.png "Selection_092.png")

Therefore, the expected number of rolls is 6.

To verify this solution is valid we can take a more rigorous approach:
Let $S_N$ be a valid sequence for coin $A$ of length $N$.  We can write the sequence in a recursive way:


![enter image description here](https://lh3.googleusercontent.com/-qtR46cPrPg4/VbWhf3oQgDI/AAAAAAAAAJ0/Wk3z2KZ8MW0/s0/Selection_093.png "Selection_093.png")

Since 2 heads cannot appear in a row except at the end, the sequence either has a tail followed by a valid sequence of size N-1 for A or a head then tail followed by a valid sequence of size N-2 for A.  N=2 is the base case, which is the 2 heads at the end.  We can use this equation to determine the probability of the number of flips:

![enter image description here](https://lh3.googleusercontent.com/-9w-rHM9Cr0o/VbWhpxtxmnI/AAAAAAAAAKA/7iQ8LRrB05Y/s0/Selection_094.png "Selection_094.png")

Which gives us our equation to solve:
$$P(S_N) = \frac12P(S_{N-1}) + \frac14P(S_{N-2})$$
We can solve this problem then starting with the base cases:

![enter image description here](https://lh3.googleusercontent.com/---mtdz4j5Og/VbWh0QUpCmI/AAAAAAAAAKM/53ur8BjfTDg/s0/Selection_095.png "Selection_095.png")

The expected value $E[X]$ will have to be summed over all values of $N$, using the equation:
$$
E[X] = \sum_{N=0}^{\infty} N * P(S_{N})
$$

I wrote a small python program to sum this up:
```
def S_N(N):
    S = [0, 0, 0.25, 0.125]
    for k in range(4,N+1):
        S.append(0.25*S[k-2] + 0.5*S[k-1])
    return S

S = S_N(10000)    
result = sum([k*S[k] for k in range(len(S))])
```
This gives `5.9999999999999` $\approx$ 6 which agrees to the original result above.

Therefore $E[X] = 6$
For $Y$, we have the following markov model:

![enter image description here](https://lh3.googleusercontent.com/-Fd90aMkWJg8/VbV-GVQ5JUI/AAAAAAAAAJU/SRYm3klwI94/s0/Selection_090.png "HHH.png")

This time it ends on 3 heads instead of 2.  The model gives us the following recursive equation:
$$
E[Y] = \frac183 + \frac18(3 + E[Y]) + \frac14(2 + E[Y]) + \frac12(1 + E[Y])
$$
Re-arranging, we obtain the solution for $E[Y]$:

![enter image description here](https://lh3.googleusercontent.com/-7Si9mrd49Xo/VbWh-uaYGtI/AAAAAAAAAKY/tWv5RD4ZaUA/s0/Selection_096.png "Selection_096.png")

The recursive relation for the sequence is (using $R$ instead of $S$ for sequences ending with 3 heads):

![enter image description here](https://lh3.googleusercontent.com/-FMTdln2qPF0/VbWiOGGtsRI/AAAAAAAAAKo/6vGA6vREyeo/s0/Selection_097.png "Selection_097.png")

This gives the recursive probability equation:

![enter image description here](https://lh3.googleusercontent.com/-YVEoXWRBi10/VbWiWjMZJWI/AAAAAAAAAK0/-rmPM0glIMk/s0/Selection_098.png "Selection_098.png")

Using a small python program I verified the equation:
```
def R_N(N):
    R = [0, 0, 0, 0.125]
    for k in range(4,N+1):
        R.append(0.125*R[k-3] + 0.25*R[k-2] + 0.5*R[k-1])
    return R[N]

R = R_N(10000)
result = sum([k*R[k] for k in range(len(R))])
```
Which gives `14.0` for the expectation.

## Part 2. Calculating P(X > Y)
Now our goal is to calculate $P(X > Y)$.  We can write this by marginalizing Y as:
$$
P(X > Y) = \sum_{k=0}^{\infty} P(X > Y | Y = k)P(Y = k)
$$
We already know how to calculate $P(Y = k)$, it's just $P(R_k)$. But how do we calculate $P(X > Y| Y = k)$?  

We can re-write this as $P(X > Y|Y = k) \longrightarrow P(X > k)$, which can be written as an infinite series:
$$
P(X > k) = \sum_{n={k+1}}^{\infty} P(X = n)
$$
$P(X = n)$ is just $P(S_n)$.  This gives us the following equation to solve:

![enter image description here](https://lh3.googleusercontent.com/-BZaJ0ubXSss/VbWigb8g7-I/AAAAAAAAALA/FHmGzkQfPeE/s0/Selection_099.png "Selection_099.png")

Using the small python programs from above, This equation can be evaluated:
```
result = sum([
	sum([S_N[n] for n in range(k+1, 10000)])*R_N[k] 
	for k in range(10000)])
```
This gives `0.2124`.  

Running a simulation of $P(Y > X)$ (below), we obtain $0.2129$.
```

def flipUntil(patt='H'):
    ss = ''
    for k in range(1000):
        res = (random() > 0.5)
        ss += ('H' if res else 'T')
        if patt in ss:
            return len(ss)

def trialsComp(num=100000, patt1='HH', patt2='HHH'):
    results = []
    for k in range(num):
        results.append(flipUntil(patt1) > flipUntil(patt2))
    return float(sum(results))/float(len(results))

trailsComp(100000, 'HH', 'HHH')
```
The simulation confirms our results to ~00.05% accuracy.

I also simulated running sequences ending with `HH` and `HHH` to confirm the math above here:
```
def trials(num_trials, patt):
    res = ([float(flipUntil(patt)) for k in range(num_trials)])
    return float(sum(res))/float(len(res))

X_empirical = trials(100000, 'HH')   #  5.9984
Y_empirical = trials(100000, 'HHH')  # 14.04246
```
