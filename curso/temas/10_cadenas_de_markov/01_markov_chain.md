> "In life, good deeds often go unrewarded. However, with a large enough sample size, seemingly skewed outcomes will converge to an expected value, and things will eventually turn out in your favour."
>
> — *Kaguya-sama: Love is War*, Chapter 195  
> Aka Akasaka

# Markov Chains in AI: A Gentle Introduction

## 1. Introducing Markov Chains  
Markov chains are mathematical models that describe a sequence of events where the probability of each event (state) depends only on the state attained in the previous event – this is known as the **Markov property**, or **memorylessness**. In simple terms, the future state depends only on the **present state** and not on how that state was reached. Formally, if $s$ and $s'$ are states, a first-order Markov chain satisfies: 

\[ t(s, s') = P(s_{t+1} = s' \mid s_t = s), \] 

meaning the chance of transitioning to state *s'* at the next time step $t+1$ given the current state $s_t$ is independent of past states before $t$. All such transition probabilities for a Markov chain can be organized into a **transition matrix** $T$, where each entry $T_{ij} = P(\text{next state} = j \mid \text{current state} = i)$. Each row of $T$ sums to 1 (a stochastic matrix), reflecting that from any state $i$, the probabilities of transitioning to all possible next states $j$ add up to 100%. The set of all possible states is called the **state space** (often denoted $S$).


To build intuition, consider a few relatable examples:
- **WhatsApp Replies:** Whether you reply to a message might depend mostly on the latest message you saw (e.g. if it's a question or not) and not the entire chat history. This can be seen as a Markov chain where the state is "last message type" and influences your response (next state).
- **Video Game AI:** An NPC's behavior might be modeled with states like "patrolling," "alerted," or "chasing." The choice of next action depends on its current state (if it's alert, it may transition to chasing with some probability). 
- **Social Media Trends:** A topic on Twitter could be in states such as "dormant," "trending," or "viral." Each day it might stay trending or fade, depending only on its current status.
- **Dice Rolls and Traffic:** A fair die roll is memoryless by nature (the next roll is independent of the last), and traffic flow at an intersection can be modeled so that the next minute's congestion depends mainly on the current level (assuming sudden accidents or earlier conditions beyond the current state are negligible).

![Example of a Markov chain shown as a state transition diagram with three states](https://upload.wikimedia.org/wikipedia/commons/7/7f/Stochmatgraph.png)
*Figure:* Example of a Markov chain shown as a state transition **diagram** with three states. Arrows indicate possible transitions between states, labeled by their transition probabilities. Such a diagram is an equivalent representation to a transition matrix, illustrating the Markov property: from each state (1, 2, or 3), the chain "hops" to the next state based only on the current node's outgoing probabilities, forgetting where it was earlier.

[Source: Wikimedia Commons - Stochmatgraph](https://commons.wikimedia.org/wiki/File:Stochmatgraph.png)

In all these cases, the process can be thought of as a sequence of states $s_0 \to s_1 \to s_2 \to \cdots$ where each transition $s_t \to s_{t+1}$ follows a fixed probability rule $P(s_{t+1}\mid s_t)$ given by the matrix $T$. Because only the present matters, Markov chains dramatically simplify modeling sequential phenomena. Instead of tracking the entire history, we just keep track of the **current state**. This makes analysis and computation feasible for AI systems that must predict or plan over sequences, as we will see next.

### 1.2 Mathematical Form
A **Markov chain** is a sequence of random variables $\{s_t\}_{t=0}^{\infty}$ taking values in a countable set $S$ (the **state space**). The defining property is that the probability of transitioning to the next state depends only on the current state and not on the states before that. Formally:

$P(s_{t+1} = j \mid s_t = i, s_{t-1} = i_{t-1}, \ldots, s_0 = i_0) = P(s_{t+1} = j \mid s_t = i),$

for all $i, j \in S$. This **memorylessness** is the **Markov property**.

#### 1.2.1 Transition Probabilities and Matrix

For a **discrete-time, finite** Markov chain with finite state space $S = \{1,2,\ldots,n\}$, we define a **transition probability** $p_{ij}$ as

$p_{ij} = P(s_{t+1} = j \mid s_t = i).$

These probabilities form an $n \times n$ **transition matrix** $P = [p_{ij}]$ with two key properties:

1. **Nonnegativity**: $p_{ij} \ge 0$.
2. **Row-Stochastic**: $\sum_{j=1}^n p_{ij} = 1$ for each $i$.

Hence each row of $P$ corresponds to a probability distribution over next states. Because each row is a probability distribution, the matrix PP is sometimes called a row-stochastic matrix.

### 1.3 Example

Throughout this chapter, we will anchor discussions with a lighthearted example from *Kaguya-sama: Love is War*. Imagine three states describing the dynamic between Kaguya Shinomiya and Miyuki Shirogane:

* **Distanced (D)**
* **Engaged (E)**
* **Standoff (S)**

Initially, we leave the transition probabilities abstract, simply noting that from D, they might stay in D or move to E or S. We will specify concrete probabilities in later sections to illustrate calculations (e.g., computing a steady-state distribution).

## 2. Designing Markov Chains  

## 2.1 Intuition
When designing a Markov chain model for an AI problem, the first step is to decide on a clear **state representation**. Each state should capture all the relevant information about the past that is needed to determine the future. If the environment is **fully observable** (the agent has access to the true state) and **stochastic** (there's randomness in transitions), we can often model its dynamics as a Markov chain. The transition probabilities can be estimated from data or defined from domain knowledge. Essentially, you ask: *"Given state X at time t, what are the probabilities of moving to each possible state at time t+1?"* Those probabilities form the entries of the transition matrix $T$. In a game of tic-tac-toe, for instance, a "state" might be the configuration of the board; if we were to randomize players' moves, we could form a Markov chain of game states. In a traffic system, a state could be the current traffic level (e.g. light, medium, heavy) and transitions model how traffic tends to build up or dissipate in the next hour.

Designing a Markov chain also involves understanding the **environment's structure**. In a fully observable environment, the agent knows exactly which state it's in, so it can directly use the Markov chain. In a partially observable setting, the agent might still **assume** an underlying Markov chain (this leads to Hidden Markov Models, touched on later). The environment's behavior can be **deterministic or stochastic**. If it's deterministic (the next state is fixed given the current state), the transition probabilities are 0 or 1; if it's stochastic, we have a distribution over next states. AI modelers often design Markov chains to approximate complex processes. For example, a simplified turn-based game like *Dungeons & Dragons (D&D)* might have states representing the turn order or phase of the game (exploration, battle, etc.), and probabilities capturing the likelihood of moving from one phase to another due to player actions or dice rolls.

In game theory and multi-agent settings, Markov chains can model evolving **game states or player strategies**. Consider a competitive video game with two players: one could model the state of the match (who is winning, special conditions active, etc.) as a Markov chain, where transitions occur based on the combined strategies and randomness. Even though players choose actions, if we fix a policy for each player or use a mixed (probabilistic) strategy, the *resulting state evolution* can be seen as a Markov chain. For instance, in a simplified rock-paper-scissors tournament, the state might be the score difference, and given the current difference, the match might swing to a win, loss, or tie with certain probabilities (assuming certain strategy patterns). Markov chains are useful in such scenarios to analyze long-run behavior or the likelihood of states (like eventually winning the game from a given situation).

![State transition diagram for Pac-Man](https://upload.wikimedia.org/wikipedia/commons/9/9a/Transition_graph_pac-man.png)
*Figure:* State transition diagram for a video game (Pac-Man as an example). Each oval is a game state (numbered regions in a Pac-Man grid) and each arrow shows a possible state transition with a probability. For example, from state 5 the game might move to state 6 with 1/3 probability, or back to state 8 with 1/4, etc., based on Pac-Man's movement and random ghost behavior. Such diagrams help in **designing Markov chains** by visualizing how an agent (like Pac-Man) can move through the state space. In designing your Markov model, you'd ensure that from any given state (node), the outgoing arrows and their probabilities accurately reflect the environment's rules. By carefully choosing states and transition probabilities, we create a Markov chain model that captures the essential dynamics of the system or game.

[Source: Wikimedia Commons - Transition graph pac-man](https://commons.wikimedia.org/wiki/File:Transition_graph_pac-man.png)


### 2.2 Formal Definition

### 2.2.1 State Space and Observability
A crucial modeling step is choosing what constitutes a **state** so that it encapsulates all the relevant information for predicting the next state. In a **fully observable** environment, the current state is fully known; if we assume Markovian dynamics, then we only need that one state to predict the next. If states are only partially observable, the model becomes a **Hidden Markov Model**.

1. **State Definition**: Ensure each state is well-defined and comprehensive enough that the Markov property realistically holds.
2. **Deterministic vs. Stochastic**: Deterministic environments yield transition matrices with 0s and 1s; stochastic environments have rows that are "spread" over multiple possible next states.

### 2.2.2 Estimating Transition Probabilities
If data is available, one estimates $p_{ij}$ from observed transitions:

$p_{ij} \approx \frac{\text{\#(times the chain transitioned from state }i\text{ to }j)} {\text{\#(times the chain was observed in state }i\text{)}}.$

Where data is sparse, smoothing methods or domain knowledge can help define probabilities.

### 2.3 Example
Suppose we hypothesize:

* From **Distanced (D)**: 50% chance they remain **Distanced**, 30% chance they become **Engaged**, 20% chance a **Standoff** arises.
* From **Engaged (E)**: 20% chance to step back to **Distanced**, 40% chance to stay **Engaged**, 40% chance to escalate to **Standoff**.
* From **Standoff (S)**: 10% chance to move to **Distanced**, 40% chance to revert to **Engaged**, and 50% chance remain in **Standoff**.

Then the transition matrix is:

$P = \begin{bmatrix} 
0.5 & 0.3 & 0.2\\
0.2 & 0.4 & 0.4\\
0.1 & 0.4 & 0.5 
\end{bmatrix},$

## 3. Computing Markov Chains  

### 3.1 Intuition
Once we have a Markov chain defined (state space and transition matrix $T$), we can perform various computations to understand its behavior. A central concept is the **steady-state distribution** (also called the *stationary distribution* $\pi$). This is a probability distribution over states that remains *unchanged* by the transition process – in other words, $\pi$ satisfies $\pi T = \pi$. If you start the chain in the steady-state distribution, it will stay in that distribution at all times. Intuitively, as the chain runs for a long time, it "forgets" its initial state and the fraction of time it spends in each state converges to the steady-state probabilities. For example, if a particular state has a steady-state probability of 0.25, then in the long run the chain will be in that state about 25% of the time (on average). Finding $\pi$ involves solving a system of linear equations (one for each state), or by computing $T^n$ for large $n$ and looking at where the rows (or columns) stabilize.

To compute **n-step transition** probabilities (the probability of going from state $i$ to state $j$ in exactly $n$ steps), we can take the matrix power $T^n$. The $(i,j)$ entry of $T^n$ gives $P(s_n = j \mid s_0 = i)$. This is useful for questions like "If the system is in state A now, what's the probability it will be in state B after 5 transitions?" – you'd look at $(T^5)_{AB}$. Computationally, one can do this by repeated matrix multiplication or using algorithms like **power iteration** which repeatedly multiplies an initial distribution by $T$ (simulating the chain) until convergence. Power iteration is a simple way to approximate the steady-state: start with any guess $\pi^{(0)}$ and compute $\pi^{(k+1)} = \pi^{(k)} T$ at each step; as $k$ grows, $\pi^{(k)}$ will approach the steady-state $\pi$ for an *ergodic* chain.

**Ergodicity** is a property that ensures a Markov chain will converge to a unique steady-state distribution. An ergodic chain is one that is both **irreducible** (it's possible to eventually reach any state from any other state, at least in theory) and **aperiodic** (it doesn't get caught in cycles with a fixed period). If these conditions hold, $T^n$ as $n \to \infty$ approaches a matrix where each row is the steady-state $\pi$. Many real-world processes have this tendency of "forgetting" initial conditions if they run long enough and if no state is absorbing. For example, consider web page surfing as a Markov chain: given some reasonable randomness (the basis of Google's PageRank algorithm), eventually the probability of being on any given page stabilizes (that's a steady-state).

Other computations of interest include **expected hitting times** (e.g., "how many steps on average to eventually reach state X starting from state Y?") and **absorption probabilities** in chains that have absorbing states (states that once entered cannot be left). These can often be derived from $T^n$ or by solving systems of linear equations. In summary, by leveraging linear algebra (matrix powers, eigenvalues, eigenvectors) we can analyze Markov chains quantitatively. For instance, steady-state probabilities correspond to an eigenvector of $T$ (specifically, the left eigenvector with eigenvalue 1). Tools like eigenvalue decomposition give insight into the chain's long-term behavior and how fast it converges to steady-state (the second-largest eigenvalue magnitude relates to the convergence rate).

![Two-state Markov chain](https://upload.wikimedia.org/wikipedia/commons/2/2b/Markovkate_01.svg)
*Figure:* A simple two-state Markov chain (State A and State E). Arrows indicate the probability of transitioning from one state to the other (e.g., the arrow from A to E might have probability $p$, and from A to A is $1-p$). In such a chain, it's easy to compute the steady-state by hand – it will be the ratio of the opposing transition probabilities. For example, if $P(A \to E) = P(E \to A) = 0.5$, the steady-state distribution is 50% A and 50% E. In more complex chains, we use algorithms to find this equilibrium.

[Source: Wikipedia - Markov chain](https://en.wikipedia.org/wiki/Markov_chain)

### 3.2 Formal definition

Once a Markov chain is defined, several fundamental properties and quantities are of great interest: **n-step transition probabilities**, **steady-state (stationary) distributions**, **ergodicity**, **hitting times**, **absorption probabilities**, and **eigenvalue-based convergence**. Below, we present formal definitions and detailed numerical examples to illustrate each concept thoroughly.

#### 3.2.1 n-Step Transition Probabilities
Define $P^n$ as the $n$-th matrix power of $P$. The entry $(P^n)_{ij}$ gives
$$(P^n)_{ij} = P(s_n=j \mid s_0=i).$$

Hence, $(P^2)_{ij}$ yields the probability of going from $i$ to $j$ in 2 steps, etc.

**Why does matrix powering work?** Because the probability of a *length-n* path factors into the product of transition probabilities along the path. When you multiply $P$ by itself, you sum over all intermediate possibilities, systematically accounting for every path of *length-n*.

**Example:** In the 3-state Kaguya-Shirogane chain $(D,E,S)$, computing $P^2$ by matrix multiplication gives you 2-step transition probabilities. For instance, if from Distanced to Standoff in 2 steps is $(P^2)_{D,S}$, you can see how likely it is for Kaguya and Shirogane to escalate from being Distanced now to a Standoff 2 interactions later.

#### 3.2.2 Steady-State (Stationary) Distribution
A stationary distribution $\pi$ (a row vector) satisfies:

* $\pi P = \pi$,
* $\sum_{i \in S} \pi_i = 1$, $\pi_i \geq 0$.

Equivalently, $\pi$ is the left eigenvector of $P$ associated with the eigenvalue $1$. Interpreting $\pi$ as a probability distribution over states, condition $\pi P = \pi$ means that if the chain is "started" in $\pi$, it remains in $\pi$ at each step.

* **Left Eigenvector**: Algebraically, $\pi$ being a solution to $\pi P = \pi$ means $\pi$ is a **left eigenvector** of $P$ with eigenvalue $1$.
* **Intuition**: If $\pi$ is your current "distribution" of states, multiplying by $P$ doesn't change it. You can see $\pi$ as a balance of "inflows" and "outflows" for each state: the fraction of probability that arrives into state $i$ from other states equals what leaves state $i$. That self-consistency yields no net change. Another way to see it, if you view probabilities as flows along edges in the Markov chain, each state’s incoming flow equals its outgoing flow under ππ. This makes ππ an equilibrium point—no net movement occurs in that distribution.
* **Existence and Uniqueness**: If the chain is **irreducible** and **aperiodic** (see 3.2.4), then $\pi$ is unique and is called the **steady-state**. If the chain is not irreducible, multiple stationary distributions might exist or they might be confined to subsets of states.
* **Practical Computation**: Often, one solves $\pi P=\pi, \sum_i\pi_i=1$ via linear algebra. Alternatively, **power iteration** repeatedly multiplies an initial vector by $P$ until convergence, which also yields $\pi$.


#### 3.2.3 Existence and Uniqueness

* A finite Markov chain may have more than one stationary distribution if it is not irreducible (i.e., if the state space is broken into multiple "communication classes").
* If the chain is irreducible and aperiodic (see Section 3.2.4), there is exactly one stationary distribution $\pi$, often called the steady-state distribution.

**Interpretation:** If you run an irreducible, aperiodic Markov chain for a long time, the fraction of time it spends in each state converges to $\pi$, regardless of the initial state. This is sometimes referred to colloquially as "the chain forgets its initial condition."

##### 3.2.3.1 Computing $\pi$ – Numerical Example

Using the Kaguya-Shirogane transition matrix:
$$P = \begin{bmatrix}
0.5 & 0.3 & 0.2\\
0.2 & 0.4 & 0.4\\
0.1 & 0.4 & 0.5
\end{bmatrix},$$

we want to find $\pi = (\pi_D, \pi_E, \pi_S)$ such that:
$$\pi P = \pi, \quad \pi_D + \pi_E + \pi_S = 1, \quad \pi_i \geq 0.$$

**System of Equations**

Writing $\pi P = \pi$ explicitly:
$$\begin{cases}
\pi_D = 0.5 \pi_D + 0.2 \pi_E + 0.1 \pi_S,\\
\pi_E = 0.3 \pi_D + 0.4 \pi_E + 0.4 \pi_S,\\
\pi_S = 0.2 \pi_D + 0.4 \pi_E + 0.5 \pi_S,\\
\pi_D + \pi_E + \pi_S = 1.
\end{cases}$$

We can solve by standard linear algebra methods (e.g., subtract each row from $\pi_i$, then add the normalization constraint). A quick method is also power iteration:

1. Choose an initial $\pi^{(0)}$, say $(1/3, 1/3, 1/3)$.
2. Repeatedly compute $\pi^{(k+1)} = \pi^{(k)} P$.
3. Iterate until $\pi^{(k+1)} \approx \pi^{(k)}$ within a tolerance $\varepsilon$.

**Illustrative Result**

One finds a unique $\pi \approx (0.326, 0.352, 0.322)$. This means that in the long run, the chain spends roughly 32.6% of the time in Distanced, 35.2% in Engaged, and 32.2% in Standoff—no matter where it started (as long as the chain is irreducible and aperiodic).

#### 3.2.4 Ergodicity: Irreducibility and Aperiodicity

**Definition:** A Markov chain is ergodic if it converges to a unique steady-state distribution from any initial state distribution. Two properties ensure ergodicity in a finite chain:

1. **Irreducible:** The chain is irreducible if for all states $i$ and $j$, there exists some integer $n$ such that $(P^n)_{ij} > 0$. In other words, any state is reachable from any other state (in some number of steps).
2. **Aperiodic:** The period of a state $i$ is the greatest common divisor (gcd) of all $n$ such that $(P^n)_{ii} > 0$. A state is aperiodic if its period is 1. A chain is aperiodic if all its states are aperiodic.

**Theorem (Ergodic Theorem)**

If a finite Markov chain is irreducible and aperiodic, then there is a unique stationary distribution $\pi$. Moreover,
$$\lim_{n \to \infty} P^n = \begin{bmatrix} \pi \\ \pi \\ \vdots \\ \pi \end{bmatrix},$$

meaning every row of $P^n$ converges to $\pi$ as $n \to \infty$.


##### Ergodic Theorem—Meaning & Importance
* **Exact Statement**: $\displaystyle \lim_{n\to\infty}P^n$ has each row equal to the unique steady-state vector $\pi$.
* **Intuition**: From any initial state distribution, repeated applications of $P$ "mix" the chain so thoroughly that the past is "forgotten." Eventually, you stabilize to the same distribution $\pi$ regardless of how you began. Because the chain is irreducible, there's only one "pool" of states, and being aperiodic prevents the chain from cycling with a fixed interval. Over many steps, any initial distribution "mixes" across all states until it stabilizes at $\pi$. Thus, the row $(P^n)_{i,\cdot}$ (the distribution after $n$ steps from state $i$) converges to $\pi$ for every $i$.
* **Importance**: This allows us to discuss **long-run behavior** unambiguously: e.g., in the Kaguya-Shirogane chain, the fraction of time in each rivalry state "converges" to a single set of proportions. It also underlies algorithms like PageRank, where the chain's "steady-state" reveals the page importance.
* **Who made it?**: John von Neuman considered it one if his greatest contributions to science.


#### 3.2.5 Hitting Times

Let $T_j$ be the **first time** the chain visits state $j$. Formally:

$T_j = \min\\{n \ge 0 : s_n = j\\}$.

If our chain starts in state $i \neq j$, then $T_j$ measures *how many steps* it takes (from time zero onward) to reach $j$ for the first time. We define the **expected hitting time** as

$h_i = E[T_j|s_0 = i]$.

##### Why the Standard Formula?

- If $i = j$, clearly $h_j = 0$ because we are already in state $j$.
- If $i \neq j$, in one step we move to some state $k$ with probability $p_{ik}$.
  - If $k = j$, we have arrived after exactly 1 step (no more steps needed).
  - If $k \neq j$, we still need $h_k$ additional steps on average from $k$.

Hence we write:

$h_j = 0,$

$h_i = 1 + \sum_{k \neq j} p_{ik}(h_k), \text{for } i \neq j.$

The "1" accounts for the step we just took from $i$. After that, with probability $p_{ik}$, we either finish if $k = j$ or continue for $h_k$ more steps if $k \neq j$. Summing over all $k$ weighted by $p_{ik}$ yields a self-consistent linear equation for $h_i$.

##### Mathematical Intuition

1. **Linearity of Expectation**: We decompose the expected number of steps into "1 step taken now" + "expected steps remaining afterward."
2. **System of Equations**: The unknowns $\\{h_i\\}$ are linked by these equations; solving them simultaneously gives each $h_i$.
3. **Interpretation**: $h_i$ is the average time from $i$ to first visit $j$. In reliability engineering, $j$ might be "failure." In AI planning, $j$ might be a "goal" state. Hitting times thus quantify how long we expect before hitting a key event.




##### 3.2.5.1 Numeric Example of Hitting Time

Consider a smaller 2-state chain to keep the computation simple:
$$P = \begin{bmatrix} 0.6 & 0.4 \\ 0.2 & 0.8 \end{bmatrix},$$

with states $\{1,2\}$. Suppose we want the expected time to hit state 2 starting from state 1. Let $h_1$ denote $E[T_2 \mid s_0 = 1]$. Then:

* $h_2 = 0$ by definition (if you start in state 2, the hitting time of 2 is zero).
* For state 1:
  $h_1 = 1 + 0.6 h_1 + 0.4 h_2$,
  but $h_2 = 0$  
  Rearranging:
  $h_1 - 0.6 h_1 = 1$,
  $0.4 h_1 = 1$,
  $h_1 = \frac{1}{0.4} = 2.5$

Thus, starting in state 1, on average it takes 2.5 steps to reach state 2.

One can analogously define and solve for $E[T_1 \mid s_0 = 2]$.

#### 3.2.6 Absorbing States and Absorption Probabilities

A state $a$ is absorbing if $p_{aa} = 1$. Once entered, it cannot be left. For instance, if Kaguya and Shirogane ultimately confess and become a couple, that state might be "absorbing" in our story's simplified chain model: once there, they do not revert to the old rivalry states.

**Canonical Form:** When analyzing absorbing states, we can reorder states so that the transition matrix has the block form:
$$P = \begin{bmatrix} Q & R \\ 0 & I \end{bmatrix},$$

where

* $Q$ (dimension $t \times t$) is transitions among transient states.
* $R$ (dimension $t \times r$) is transitions from transient to absorbing states.
* $I$ (dimension $r \times r$) is the identity, reflecting that absorbing states never change.

##### 3.2.6.1 Fundamental Matrix and Absorption Probabilities

The fundamental matrix is $N = (I - Q)^{-1}$. If you label transient states as $1, \ldots, t$ and absorbing states as $t+1, \ldots, n$, then the matrix of absorption probabilities is:
$$B = N R.$$

Algebraically, you can view $N$ as $(I + Q + Q^2 + \dots)$, a convergent series if no transient state traps you forever. * **Why $(I-Q)^{-1}$?** Intuitively, each power $Q^m$ describes the probability of staying among transient states for $m$ steps. Summing across all $m$ captures the total expected number of visits to each transient state. Thus, $N_{ik}$ is how many times, on average, you visit state $k$ if you start in state $i$ **before** absorption.

$B = N \,R$, a $t \times r$ matrix. Each entry $B_{ij}$ gives the probability that, starting in transient state $i$, you eventually end up in absorbing state $j$. The logic is: once you know how many times you expect to be in each transient state (from $N$), you multiply by $R$ to see where you go when you leave the transient set.

##### 3.2.6.2 Kaguya-Shirogane Example With Two Absorbing States

Extend the state set to $\{D, E, S, B, X\}$:

* $D, E, S$ are transient (Distanced, Engaged, Standoff).
* $B$ = "Breakthrough": They confess or reach a conclusive relationship. (Absorbing)
* $X$ = "Deadlock": Pride overwhelms them; they quit interacting meaningfully. (Absorbing)
* $I$ = "Identity Matrix": Is the identity matrix with the dimention requiered for the operation.


A possible $5 \times 5$ transition matrix (in canonical order $[D,E,S \mid B,X]$):
$$P = \begin{bmatrix}
0.4 & 0.4 & 0.2 & 0 & 0 \\
0.2 & 0.5 & 0.3 & 0 & 0 \\
0.1 & 0.3 & 0.4 & 0.2 & 0 \\
0 & 0 & 0 & 1 & 0 \\
0 & 0 & 0 & 0 & 1
\end{bmatrix},$$

so
$$Q = \begin{bmatrix}
0.4 & 0.4 & 0.2 \\
0.2 & 0.5 & 0.3 \\
0.1 & 0.3 & 0.4
\end{bmatrix}, \quad
R = \begin{bmatrix}
0 & 0 \\
0 & 0 \\
0.2 & 0
\end{bmatrix}.$$

Then compute $N = (I - Q)^{-1}$ and $B = N R$. The resulting $B$ will show how likely it is for the chain to end in $B$ (Breakthrough) vs. $X$ (Deadlock) from each of $D, E, S$.

#### 3.2.7 Eigenvalues and Convergence Rates

For an $n \times n$ transition matrix $P$:

* The largest eigenvalue is always 1.
* All other eigenvalues lie in the closed unit disk (by the Perron–Frobenius theorem for stochastic matrices).

If the chain is irreducible and aperiodic, the eigenvalue 1 is simple (i.e., has algebraic multiplicity 1), and the corresponding eigenvector is the unique $\pi$. The second-largest eigenvalue in magnitude determines the rate of convergence to $\pi$. Specifically, if $\lambda_2$ is the eigenvalue with largest magnitude below 1, then $\|P^n - J\|$ (where $J$ is the matrix of repeated rows $\pi$) decreases roughly on the order of $\lambda_2^n$.

If the chain is **ergodic**, that eigenvalue is unique in magnitude—other eigenvalues have $|\lambda| < 1$.

**Why does $|\lambda| < 1$ force convergence?** When you repeatedly multiply by $P$, any component tied to an eigenvalue $\lambda$ with $|\lambda|<1$ shrinks to zero as $n\to\infty$. Only the part aligned with the eigenvalue 1 remains. In Markov chains, that part corresponds to the stationary distribution $\pi$. Hence, $P^n\to$ a matrix of repeated rows $\pi$.

**Practical Impact**: A larger gap between 1 and the second-largest eigenvalue typically means faster convergence. In algorithms like PageRank, ensuring all other eigenvalues are strictly below 1 in magnitude speeds up the approach to the steady state.



## 4. Inference and AI Agents  
Markov chains provide a foundation for how AI agents *infer* the dynamics of their environment and make decisions. If an AI agent assumes the world follows the Markov property, it can simplify its decision-making by focusing on the current state. Different types of agents use Markov chain reasoning in various ways:

- **Simple Reflex Agents:** These agents choose actions based only on the current state, ignoring history. This aligns perfectly with the Markov property. If the agent's rule says "if in state S, do action A," it's effectively leveraging the transition probability $t(S, \cdot)$ (even if implicitly) to react. For example, a thermostat is a simple reflex agent – it doesn't care whether it was hot an hour ago, only whether it's hot *now* (current state) to decide turning the AC on. We can model the room temperature as a Markov chain where the next temperature depends only on the current temperature and the AC/heater actions.
- **Model-Based Reflex Agents:** These have an internal **model** of the world's transition dynamics (a model of $t(s,s')$). In other words, the agent builds or is given a Markov chain of the environment. It can predict next states: "If I'm in state S now, my model says there's a 70% chance I go to S' and 30% chance to S'' after one step." This is essentially using the transition matrix $T$ for inference. For instance, a game AI might know the Markovian rules of the game (like how opponents usually behave from the current game state) and can update its beliefs about what will happen one turn ahead.
- **Goal-Based Agents:** These agents consider future sequences of states to achieve a goal. They can use Markov chain properties to plan by looking ahead multiple steps (often this is done with search or planning algorithms). If the agent knows the transition matrix $T$, it can compute $T^n$ or simulate many transitions to see where it might end up. Goal-based reasoning often involves finding a sequence of transitions (a path through the Markov chain) that leads to a **goal state**. In Markov terms, the agent might be interested in the probability of eventually reaching a goal state (a hitting probability) or the expected number of steps to reach it from the current state. By examining the chain (sometimes using algorithms like dynamic programming if rewards are involved), the agent can choose actions that increase the likelihood of reaching the goal. In a path-planning AI for navigation, for example, the map can be viewed as a Markov chain of locations, and the goal-based agent searches for a high-probability path to the destination.
- **Utility-Based Agents:** These agents not only have goals but also a utility (value) function to evaluate how desirable each state is. When an environment is modeled as a Markov chain, a utility-based agent will combine the transition probabilities with the utilities of future states to decide the best action. Essentially, it performs a kind of *expectimax* or expectated utility calculation: it considers "if I am in state S, and I take action A, the next state could be S' or S'' with certain probabilities (from the Markov model); how much utility will I likely get?" The agent will prefer the action that leads to the highest expected utility. In practice, this often leads into the realm of **Markov Decision Processes (MDPs)** – which are Markov chains extended with actions and rewards. The utility-based agent, via algorithms like value iteration or policy iteration, calculates the optimal policy on an underlying Markov chain of states.
- **Learning Agents:** These agents improve over time by learning from experience. In the context of Markov chains, a learning agent might start with unknown transition probabilities and then **learn** them by observing the frequencies of state transitions (essentially doing empirical estimation of $t(s,s')$). This is exactly what happens in **reinforcement learning**, where an agent explores an environment and estimates the state transition model and/or the value of states. By assuming the environment is Markovian, the learning algorithms (like Q-learning or Monte Carlo simulation) update estimates based on the current state and the observed next state, gradually converging to an accurate model of the Markov chain. Learning agents connect game theory and Markov chains when, for example, they adjust their strategy in a game (the transition probabilities might change as the opponent learns too – a more complex scenario!). Nonetheless, many AI learning scenarios use the Markov assumption as a backbone; even if the agent doesn't explicitly learn "the matrix," it often learns a policy that is optimal for the Markov chain dynamics of the environment.

It's worth noting how Markov chains are the stepping stone to more advanced models in AI:
- A **Hidden Markov Model (HMM)** is essentially a Markov chain where the states are not directly observed (they're "hidden") and we only see some observations emitted from states. The underlying chain might model, say, the emotional state of a user (happy, neutral, sad) which is not directly observable, while the observations are their messages or actions. The AI must infer the hidden state distribution using algorithms like forward-backward – all built on the Markov assumption for state transitions.
- A **Markov Decision Process (MDP)**, as mentioned, is a Markov chain augmented with actions and rewards. In an MDP, when an agent takes a given action in state $s$, the state transitions according to $P(s_{t+1}=s' \mid s_t=s, a_t=a)$. This still satisfies the Markov property (next state depends only on current state and action), and forms the core of how we model decision-making problems for AI. Solving an MDP (via dynamic programming, reinforcement learning, etc.) essentially means finding a good policy assuming the environment's transitions are a Markov chain influenced by actions.
- A **Partially Observable MDP (POMDP)** generalizes HMMs and MDPs – here the agent doesn't directly observe the true state, so it keeps a **belief** (a probability distribution over states). The belief itself can be seen as a state in an *even larger* Markov process! Markov chains thus even underlie the belief-state dynamics: given the current belief (distribution over where we might be), and an action and observation, we update to a new belief (this belief update can be seen as a transition in a belief-state Markov chain).

In summary, Markov chains provide the reasoning framework for AI agents to predict and plan. They allow agents to **simulate the environment** in their mind, reason about "if I'm in state X now, what's likely next?" and make informed decisions. From reflexive actions to strategic planning, assuming a Markovian world simplifies the complexity. AI researchers and engineers often start with this assumption because it's mathematically tractable and often a reasonable approximation of many domains.

![Markov Decision Process diagram](https://oscarbastardo.com/mdp/figure1.png)
*Figure:* An illustration of an AI agent interacting with its environment, which can be modeled as a Markov process. The agent perceives the current **state** $S_t$ and takes an **action** $A_t$; the environment then transitions to a new **state** $S_{t+1}$ (following the Markov chain dynamics) and provides a **reward** $R_{t+1}$ (in purely Markov chains without actions, you can ignore the action and reward). This loop (often depicted as in the figure) highlights the Markov property: the environment's transition to $S_{t+1}$ depends only on the state $S_t$ and the agent's action, not on any earlier history.

[Source: Oscar Bastardo - Markov Decision Process](https://oscarbastardo.com/2021-09-26/mdp/)

## 5. Wrap-Up & Next Steps  
Markov chains are a fundamental concept in AI and many other fields because they provide a simple yet powerful way to model sequential processes. In this introductory chapter, we defined Markov chains, topics such as **Hidden Markov Models (HMMs)** for sequence data in AI (like speech or text), **Markov Decision Processes (MDPs)** for optimal decision-making in uncertain environments, and **Partially Observable MDPs (POMDPs)** for dealing with uncertainty in perception. All these advanced topics build on the ideas introduced here: states, transitions, and the memoryless property.

To reinforce these concepts, let's consider a creative real-world example of a Markov chain in action. Imagine the trending topics on a social media platform (like Twitter or TikTok). We can define states for a given topic such as: **Dormant** (hardly anyone is talking about it), **Trending** (it's gaining popularity rapidly), or **Viral** (it's extremely popular and widespread). On each day, the topic transitions between these states with certain probabilities. Perhaps if a topic is Trending today, there's a 50% chance it stays Trending tomorrow, a 40% chance it goes Viral, and a 10% chance it falls off to Dormant if interest dies down. This can be captured in a transition matrix. Over time, this forms a Markov chain of topic popularity. We could use it to answer questions like "What's the long-term fraction of time a typical topic spends in Viral state?" or "What's the probability that a currently Viral meme will be Dormant in two days?" – all using the computations we discussed (by looking at $T^n$ or solving for steady-state). This social network dynamics example shows how Markov chains can model everyday phenomena inhttps://claude.site/artifacts/58803752-5355-47aa-b34d-fa2fb8adeb22 AI – in this case, the behavior of users collectively driving topics between popularity states.

As you move forward, keep in mind how the **Markov property** simplifies learning and inference. In more complex models (HMMs, MDPs, etc.), we often assume an underlying Markov chain because it makes the mathematics workable and often approximates reality well. In the coming chapters, we will delve into those topics: for instance, we'll see how an HMM uses a Markov chain to model sequences with hidden states, and how solving an MDP is essentially finding an optimal policy on a Markov chain of states when actions are involved. By mastering Markov chains, you'll be well-equipped to understand and build upon these advanced AI techniques. 

Markov chains demonstrate that sometimes **memorylessness is more than enough** – by focusing on "now", we can predict the future and make intelligent decisions without drowning in past data. Now that you've been introduced to Markov chains, you're ready to explore the rich landscape of sequential models in AI that build on this concept. Happy learning as you step into HMMs, MDPs, POMDPs, and beyond, armed with the knowledge of Markov chains!

## A. Mathematical Apendix

front_end_string = '''
### A.1 Algebraic Derivation (Neumann Series)

Consider the scalar analogy: for $\lvert x\rvert < 1$, we know

$\frac{1}{1-x} = 1 + x + x^2 + x^3 + \dots$.

There is a well-known **matrix version** of this geometric-series identity: if $Q$ is a matrix with spectral radius $\rho(Q)<1$, then

$(I-Q)^{-1} = \sum_{m=0}^{\infty} Q^m = I + Q + Q^2 + \dots$.

**Why?** Multiply $(I-Q)$ by the finite partial sum $S_m = I + Q + \dots + Q^m$:

$(I - Q) S_m = (I + Q + \dots + Q^m) - (Q + Q^2 + \dots + Q^m + Q^{m+1}) = I - Q^{m+1}$.

As $m \to \infty$, if $\rho(Q)<1$, then $Q^{m+1} \to 0$, so

$(I - Q)(\lim_{m\to\infty} S_m) = I$,

meaning $\lim_{m\to\infty} S_m$ is indeed $(I-Q)^{-1}$.

In the absorbing-chain scenario, $\rho(Q)<1$ because "once you leave the transient states, you can't come back," so $Q^m\to 0$ as $m\to\infty$. Hence the infinite sum converges to $(I-Q)^{-1}$.

### A.1.2 Markov Chain Interpretation

When we say $N = (I - Q)^{-1} = I + Q + Q^2 + \dots$, we are summing up all possible **powers** of $Q$.

* $Q^m$ represents "the probabilities of going from one transient state to another in exactly $m$ steps **without being absorbed**."
* $I$ (the identity) is effectively "0-step transitions" (you start in state $i$ with probability 1 of being in $i$ at step zero).
* **Adding** them up, $\sum_{m=0}^{\infty} Q^m$ accumulates the probability—and eventually the expected **number of visits**—of being in each transient state at each step up to infinity.

Because the chain cannot linger in transient states forever (there is always a chance to transition to an absorbing state), powers $Q^m$ become negligible for large $m$. In fact, $Q^m\to 0$, capturing that eventually you **escape** the transient set with probability 1. Mathematically, that ensures the infinite series converges, and **algebraically** that it equals $(I-Q)^{-1}$.

Hence, from a **Markov chain perspective**:

1. $N = (I - Q)^{-1}$ **sums up** contributions from all possible "transient circuits" you might do before absorption.
2. $N_{ik}$ is the **expected number of visits** to transient state $k$ starting from transient state $i$.
3. When you multiply $N$ by $R$ (the transitions from transient to absorbing), you get $B = N R$, which gives **the probability** of ending in each absorbing state.

All of this is ultimately grounded in the spectral property that $\rho(Q)<1$, which ensures $(I-Q)$ is invertible and the series $\sum_{m=0}^\infty Q^m$ converges.
