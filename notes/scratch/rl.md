notes from David Silver's Reinforcement Learning course [Advanced Topics: RL](https://www.youtube.com/watch?v=2pWv7GOvuf0&list=PL5X3mDkKaJrL42i_jhE4N-p6E2Ol62Ofa) ([see also](http://www0.cs.ucl.ac.uk/staff/d.silver/web/Teaching.html)). 2015 (COMPM050/COMPGI13).

Reinforcement learning characteristics:

- no supervisor (nothing top-down saying what's right and what's wrong as in supervised learning), only a reward signal
- feedback (reward) may be delayed (not instantaneous)
    - e.g. giving up short-term gain may be better for long-term gain
- time is important (i.e. sequences of actions)
- the agent's actions affects the subsequent data it receives

A _reward_ $R_t$ is a scalar feedback signal that indicates how well the agent is doing at step $t$. The agent's objective is to maximize cumulative reward.

Note that the reward must be scalar though you could take a reward vector over different aspects (e.g. this increases my income, but decreases my happiness) and take its dot product with some weights (e.g. I value my happiness more than my income) to retrieve a scalar value.

The _reward hypothesis_ states that goals can be described by the maximization of expected cumulative reward and is the central assumption of reinforcement learning.

The term "reward" may be a bit confusing, since they may be negative as well, i.e. "punishment".

Rewards can come at each time step (for example, if you want an agent to do something quickly, you can set a reward of -1 for each time step), or at the end of the task, or at the end of each "episode" if the task has no end (but you need to chunk up time in some way), or for certain states, etc.

The general framework is that the agent decides on and executes some action $A_t$ which affects the environment, receives observation $O_t$ from the environment, and gets a reward $R_t$.

The _history_ is the sequence of observations, actions, and rewards:

$$
H_t = A_1, O_1, R_1, \dots, A_t, O_t, R_t
$$

We are essentially looking to build a mapping from this history to some action to take next.

Practically, we can't work with the full history, so we summarize it as _state_ $S_t = f(H_t)$ (i.e. it is some function of the history) and so the task is to learn a mapping from states to some action to take next.

We can represent a state for the environment as well (distinct from the agent's state $S_t^a$, and typically hidden from the agent) as $S_t^e$. In multi-agent simulations, from the perspective of a single agent, the other agents can be represented as part of the environment.

We can more specifically define an _information state_ (also called a _Markov state_), which is a state in which:

$$
P[S_{t+1}|S_t] = P[S_{t+1}|S_1, \dots, S_t]
$$

That is, we make a Markov assumption that the probability of the next state depends only on the current state. This assumption allows us to effectively ignore the history previous to the present state.


How we represent state greatly affects learning. Consider an experiment in which a rat either gets shocked or gets a piece of cheese. The rat observes the following two sequences:

1. light, light, lever, bell -> shock
2. bell, light, lever, lever -> cheese

Consider this sequence:

- lever, light, lever, bell

Will the rat get shocked or get cheese?

You might think "shock" because the same 3-item sequence is at the end (light, lever, bell). However, if you represent the state as numbers of each event, you'd say cheese (1 bell, 1 light, 2 levers). Or you could represent it as the entire 4-item sequence in which case it's not clear what the result will be.


In a _fully observable_ environment, the agent directly observes environment state, i.e. $O_t = S_t^a = S_t^e$. In this case we have a Markov decision process (MDP).

In a _partially observable_ environment, the agent indirectly observes the environment (a lot of information is hidden). In this case the agent and environment states are not congruent. This is a partially observable Markov decision process (POMDP). In this case we must construct the agent's state representation separately - one possibility is as the beliefs of the environment state, or use a recurrent neural network to generate the next state, etc.

The main components that may be included in an RL agent:

- policy: the agent's behavior function
    - a map from state to action
    - it may be deterministic, i.e. $a = \pi(s)$
    - or it may be stochastic, i.e. $\pi(a|s) = P[A=a|S=s]$
- value function: how good is each state and/or action
    - a prediction of (i.e. expected) future reward
    - evaluates the goodness or badness of states
    - used to select between actions
    - e.g. $v_{\pi}(s) = E_{\pi}[R_t + \gamma R_{t+1} + \gamma^2 R_{t+2} + \dots | S_t = s]$
    - the value function $V$ is the state value function, which values states, and $Q$ is the action value function, which values actions (given states)
- model: agent's representation of the environment
    - often has two components
    - the _transitions_ $P$ predicts the next state (the "dynamics"), e.g. $P_{ss'}^a = P[S'=s'|S=s,A=a]$
    - the _rewards_ $R$ predicts the next immediate reward, e.g. $R_s^a = E[R|S=s,A=a]$
    - but there are many model-free methods as well

RL agents can be categorized according to which of these components they have:

- _value based_ agents
    - have a value function
    - if it has a value function, the policy is implied (choose the highest-valued action)
- _policy based_ agents
    - have an explicit policy
    - without a value function
- _actor critic_ agents
    - has both a policy and a value function

Further distinction is made between model-free agents (the agent doesn't model the environment, just has a policy and/or value function) or model-based agents (has a model and a policy and/or value function).

Prediction vs control:

- prediction: evaluate the future, given a policy
- control: optimize the future, find the best policy

---

# Markov Decision Processes

Markov decision processes formally describe a fully-observable environment for reinforcement learning. Almost all RL problems can be formalized as MDPs, including partially-observable RL problems: partially-observable problems can be converted into MDPs.

We make the Markov assumption (see above) for MDPs. That is, the current state captures all relevant information from the history, so the history can be thrown away.

We have a _state transition matrix_ which encodes the state transition probabilities, i.e. the probability of going to some state $s'$ from a state $s$.

A Markov process then is a memoryless (i.e. Markov) random process, i.e. a stochastic sequence of states, i.e. a Markov chain. We define a Markov process as a tuple $(S, P)$, where $S$ is a finite set of state and $P$ is the state transition probability matrix:

$$
P_{ss'} = p[S_{t+1} = s' | S_t = s]
$$

A _Markov reward process_ is a Markov chain with values. It is described by a tuple $(S,P,R,\gamma)$, which includes the reward function $R$ and the discount factor $\gamma \in [0,1]$:

$$
R_s = E[R_{t+1} | S_t = s]
$$

The _return_ $G_t$ ("G" for "goal") is the total discounted reward from time-step $t$, i.e.:

$$
G_t = R_{t+1} + \gamma R_{t+2} + \dots = \sum_{k=0}^{\infty} \gamma^k R_{t+k+1}
$$

The value function $v(s)$ gives the long-term value of state $s$. Formally, it is the expected return of a Markov reward process from state $s$, i.e.:

$$
v(s) = E[G_t | S_t = s]
$$

The Bellman Equation decomposes the value function into two parts:

- the immediate reward $R_{t+1}$
- the discounted value of successor state $\gamma v(S_{t+1})$

So it turns it into a recursive function:

$$
v(s) = E[R_{t+1} + \gamma v(S_{t+1}) | S_t = s]
$$

i.e.:

$$
v(s) = R_s + \gamma \sum_{s' \in S} P_{ss'} v(s')
$$

Or written more concisely using matrices:

$$
v = R + \gamma P v
$$

As an aside, the Bellman equation is a linear equation, so it can be solved directly:

$$
\begin{aligned}
v &= R + \gamma Pv \\
(I - \gamma P) v &= R \\
v &= (I - \gamma P)^{-1}R
\end{aligned}
$$

Where $I$ is the identity matrix.

This however has a computational complexity  of $O(n^3)$ for $n$ states so generally it is not practical.

There are other iterative methods for solving MRPs which are more important/practical, e.g. dynamic programming, Monte-Carlo evaluation, and Temporal-Difference learning.

A Markov decision process (MDP) is a Markov reward process with decisions. It is described by a tuple $(S,A,P,R,\gamma)$ which now includes a finite set of actions $A$, thus now $P$ and $R$ become:

$$
\begin{aligned}
P_{ss'}^a &= p[S_{t+1} = s' | S_t = s, A_t = a] \\
R_s^a &= E[R_{t+1} | S_t = s, A_t=a]
\end{aligned}
$$

A policy $\pi$ is a distribution over actions given states:

$$
\pi(a|s) = p[A_t = a | S_t = s]
$$

A policy fully defines the behavior of an agent. Because it depends only on the current state (and not the history), it is said to be _stationary_ (time-independent).

The _state-value function_ $v_{\pi}(s)$ of an MDP is the expected return starting from state $s$ and then following policy $\pi$:

$$
v_{\pi}(s) = E_{\pi}[G_t|S_t = s]
$$

This is different than the value function, which does not involve a policy for selecting actions but rather only proceeds randomly.

The _action-value function_ $q_{\pi}(s,a)$ is the expected return starting from state $s$, taking action $a$, and then following policy $\pi$:

$$
q_{\pi}(s,a) = E_{\pi}[G_t|S_t = s, A_t = a]
$$

We can also define Bellman equations for these value functions (these are called Bellman expectation equations):

$$
\begin{aligned}
v_{\pi}(s) &= E_{\pi} [R_{t+1} + \gamma v_{\pi} (S_{t+1}) | S_t = s] \\
&= \sum_{a \in A} \pi(a|s) q_{\pi}(s,a) \\
q_{\pi}(s,a) &= E_{\pi} [R_{t+1} + \gamma q_{\pi} (S_{t+1}, A_{t+1}) | S_t = s, A_t = a] \\
&= R_s^a + \gamma \sum_{s' \in S} P_{ss'}^a v_{\pi}(s')
\end{aligned}
$$

We can combine these:

$$
v_{\pi}(s) = \sum_{a \in A} \pi(a|s) (R_s^a + \gamma \sum_{s' \in S} P_{ss'}^a v_{\pi}(s'))
$$

There is some optimal state-value function $v_*(s) = \max_{\pi} v_{\pi}(s)$, i.e. the maximum value function over all policies.

Similarly, there is an optimal action-value function $q_*(s,a) = \max_{\pi} q_{\pi}(s,a)$. This gives us the optimal way for the agent to behave (i.e. we can get the optimal policy from this by always choosing the best action from the optimal action-value function), so this is the most important thing to figure out! That is, the MDP is "solved" when we know $q_*(s,a)$.

From this we can get the Bellman optimality equation:

$$
\begin{aligned}
v_*(s) &= \max_a q_*(s,a) \\
q_*(s,a) &= R_s^a + \gamma \sum_{s' \in S} P_{ss'}^a v_*(s')
\end{aligned}
$$

Which can then be combined:

$$
v_*(s) = \max_a R_s^a + \gamma \sum_{s' \in S} P_{ss'}^a v_*(s')
$$

Which can equivalently be written:

$$
q_*(s,a) = R_s^a + \gamma \sum_{s' \in S} P_{ss'}^a \max_{a'} q_*(s', a')
$$

The Bellman optimality equation is non-linear, unlike the Bellman equation we saw previously. In general there is no closed form solution, but we can use iterative methods, e.g. value iteration, policy iteration, Q-learning, and Sarsa.

---

## Dynamic programming

"Dynamic" refers to some sequential or temporal aspect to the problem and we want to optimize some program, i.e. a policy.

Dynamic programming is a method for solving complex problems by breaking them into subproblems, then solving the subproblems (divide-and-conquer).

For dynamic programming to work, the problem must have two properties:

- optimal substructure; i.e. the optimal solutions of the subproblems tell us about the optimal solution for the overall problem
- the subproblems should "overlap"; i.e. they should recur throughout the overall problem. That way, by solving a subproblem, we are simultaneously solving many parts of the overall problem (the solutions can be cached and reused).

MDPs satisfy both of these properties with the Bellman equation (it is recursive decomposition) and the value function stores and reuses solutions.

Dynamic programming assumes full knowledge of the MDP, so it is used for _planning_ in an MDP (i.e. prediction and control).

For _prediction_ it takes a fully-specified MDP $(S,A,P,R,\gamma)$ (or MRP $(S,P^{\pi}, R^{\pi}, \gamma)$) and a policy $\pi$ and gives us a value function $v_{\pi}$. So this is not reinforcement learning because the MDP is fully-specified (nothing needs to be learned!).

For _control_ it takes a fully-specified MDP and gives us the optimal value function $v_*$ which gives us an optimal policy $\pi_*$ as well.

### Policy evaluation

Policy evaluation involves taking a policy and an MDP and computing the expected reward for following that policy.

To solve this, we apply the Bellman expectation equation as an iterative update.

We start off with some arbitrary value function $v_1$ (e.g. value of every state is 0), then, use _synchronous_ backups (i.e. consider each state in turn):

- at each iteration $k+1$
- for all states $s \in S$
- update $v_{k+1}(s)$ from $v_k(s')$, where $s'$ is a successor state of $s$

$$
v_{k+1}(s) = \sum_{a \in A} \pi(a|s) (R_s^a + \gamma \sum_{s' \in S} P_{ss'}^a v_k(s'))
$$

This eventually converges to $v_{\pi}$.

### Policy iteration

Now that we can evaluate a policy, we can figure out how to improve it.

Given a policy $\pi$:

- evaluate the policy $\pi$: $v_{\pi}(s) = E[R_{t+1} + \gamma R_{t+2}  + \dots | S_t = s]$
- improve the policy by acting greedily wrt to $v_{\pi}$: $\pi' = \text{greedy}(v_{\pi})$ (greedy just means we move to the state with the highest value)

And we can iteratively apply this approach (called _greedy policy improvement_), which will converge to the optimal policy $\pi^*$ (no matter how you start).

![Policy iteration intuition from <https://webdocs.cs.ualberta.ca/~sutton/book/ebook/node46.html>](assets/policy_iteration.png)

The policy and the value function influence each other, since the policy dictates which states are explored and the value function influences how the policy chooses states, so they push off each other to convergence.

With the greedy policy, we always choose the best state (remember that the value of a state takes into account future reward from that state as well!) so we update the value function for that state so that it is equal to or greater than it was before. This is how the greedy policy improves the value function.

Because the value function drives the greed policy, that in turn improves the policy.

Eventually the value function will only be equal to (rather than greater than or equal to) what it was before; at this point convergence is achieved.

### Value iteration

The solution $v_*(s)$ can be found with one-step lookahead:

$$
v_*(s) \leftarrow \max_{a \in A} (R_s^a + \gamma \sum_{s' \in S} P_{ss'}^a v_*(s'))
$$

Then apply these updates iteratively. This is _value iteration_.

Basically we iterate over states and apply this update until convergence, giving us the optimal value function.

The pseudocode is:

- using synchronous backups
    - at each iteration $k + 1$
    - for all states $s \ in S$
    - update $v_{k+1}(s)$ from $v_k(s')$
- until convergence

Unlike policy iteration, there is no explicit policy here, and the intermediate value functions may not correspond to any policy. But once we have $v_*(s)$ we get the optimal policy.

### Asynchronous dynamic programming

The methods so far have used synchronous backups, i.e. each iteration we look at and update every single state simultaneously before updating any state again.

We can instead use _asynchronous backups_ in which states are backed up individually in any order, without waiting for other states to update. This can significantly reduce computation. As long as we continue to select all states (again, order doesn't matter), we will still have convergence.

Three asynchronous dynamic programming methods:

#### In-place dynamic programming

Synchronous value iteration stores two copies of the value function, i.e. for all $s \in S$, we have:

$$
v_{\text{new}}(s) \leftarrow \max_{a \in A}(R_s^a + \gamma \sum_{s' \in S} P_{ss'}^a v_{\text{old}}(s'))
$$

_In-place_ value iteration, on the other hand, only stores one copy of the value function, i.e. for all $s \in S$, we instead have:

$$
v(s) \leftarrow \max_{a \in A}(R_s^a + \gamma \sum_{s' \in S} P_{ss'}^a v(s'))
$$

That is, we immediately update the value function and always use the newest value function.

#### Prioritised sweeping

Sometimes the order can affect how quickly you reach the optimal value function. _Prioritised sweeping_ uses the magnitude of the Bellman error to guide state selection, e.g.:

$$
|\max_{a \in A}(R_s^a + \gamma \sum_{s' \in S} P_{ss'}^a v(s')) - v(s)|
$$

We backup the state with the largest remaining Bellman error, then update the Bellman error of affected states after each backup.

The magnitude of the Bellman error tells us how much our value estimate for that state changed; the intuition is that this change was really big, we want to attend to that state first, since will likely have a large influence on other values.

#### Real-time dynamic programming

With _real-time dynamic programming_ we only select and update the states that the agent actually visits.

### Complexity

These algorithms (i.e. algorithms based on the state-value function $v_{\pi}(s)$ or $v_*(s)$) have complexity $O(mn^2)$ per iteration for $m$ actions and $n$ states.

Dynamic programming uses _full-width_ backups which means we consider _all_ actions and _all_ successor states (i.e. we consider the full branching factor), which is really expensive. Furthermore, to actually do these branch lookaheads we need to know the MDP transitions and reward function (i.e. have a model of the dynamics of the environment).

So we run into the problem of dimensionality, where the number of states $n = |S|$ grows exponentially with the number of state variables. So this is not a great approach for larger problems.

One way around this is by sampling - instead of considering the entire branching factor, we sample a single complete trajectory.

But sometimes the problem is so big that even one backup is too expensive - in these cases, we can use _sample backups_, i.e. we start in a state, sample one action according to our policy, sample one transition according to our dynamics, etc, then backup this branch.

With sample backups, because we are sampling from the dynamics of the environment, this also frees us from needing a model of the dynamics of the environment.

## Model-free prediction

We have an environment that can be represented as an MDP but we are not given the dynamics or reward function (i.e. we don't know what the MDP is). With _model-free_ prediction methods, we can still learn a value function even without this knowledge (i.e. without needing to model the environment).

### Monte-Carlo learning

Not necessarily efficient, but effective.

MC methods learn directly from _complete_ episodes of experience. Note that MC methods must learn from episodes (i.e. they are only applicable to _episodic MDPs_, where episodes eventually terminate). So by "complete" episode, this means the episode is expanded to a terminal state. We do this a lot (each episode is a sample) and estimate the value of our start state as the mean return from these episodes.

Note that the _return_ is the total discounted reward.

The value function is usually the expected return (where $G_t$ is the return):

$$
v_{\pi}(s) = E_{\pi}[G_t | S_t = s]
$$

But Monte-Carlo policy evaluation uses the _empirical mean_ return instead of the expected return (i.e. as mentioned before, we collect sample episodes and compute the mean of their returns).

So how do we get these empirical mean returns for _all_ states in the environment?

There are two methods:

- _first-visit_ Monte-Carlo policy evaluation: to evaluate a state $s$, the first time-step $t$ that state $s$ is visited in an episode (i.e. if the state is returned to later one, ignore), increment counter $N(s) \leftarrow N(s) + 1$, which tracks number of visits to the state, increment the total return $S(s) \leftarrow S(s) + G_t$, then the value is estimated as the mean return $V(s) = \frac{S(s)}{N(s)}$. By the law of large numbers, $V(s) \to v_{\pi}(s)$ as $N(s) \to \infty$; that is, with enough samples, this will converge on the true value.
- _every-visit_ Monte-Carlo policy evaluation: same as the first-visit variant, except now we do these updates for _every_ visit to the state (instead of just the first)

The mean can be computed incrementally, i.e. we can do it in an online fashion. The mean $\mu_1, \mu_2, \dots$ of a sequence $x_1, x_2, \dots$ can be computed incrementally:

$$
\begin{aligned}
\mu_k &= \frac{1}{k} \sum_{j=1}^k x_j \\
&= \frac{1}{k}(x_k + \sum_{j=1}^{k-1} x_j) \\
&= \frac{1}{k} (x_k + (k-1) \mu_{k-1}) \\
&= \mu_{k-1} + \frac{1}{k}(x_k - \mu_{k-1})
\end{aligned}
$$

Using this, we'd change our $V(s)$ update to:

$$
V(s) \leftarrow V(s) + \frac{1}{N(s)} (G_t - V(s))
$$

For non-stationary problems (which is the typical case) we might want to have a running mean (i.e. forget old episodes):

$$
V(s) \leftarrow V(s) + \alpha (G_t - V(s))
$$

### Temporal Difference learning

TD methods, like Monte Carlo learning, learns from episodes, but it can learn from _incomplete_ episodes by _bootstrapping_ (similar to dynamic programming). We substitute the rest of the trajectory (the rest of the episode, before it is finished) with an estimate of what will happen from that state onwards. You take another step, generating an estimate for that step and updating the previous estimate with what what you've learned from then new step.

So whereas with Monte-Carlo learning we update $V(S_t)$ towards the _actual_ return $G_t$, with temporal-difference learning ($TD(0)$) we update $V(S_t)$ towards the _estimated_ return $R_{t+1} + \gamma V(S_{t+1})$ (like the Bellman equations):

$$
V(S_t) \leftarrow V(S_t) + \alpha (R_{t+1} + \gamma V(S_{t+1}) - V(S_t))
$$

$R_{t+1} + \gamma V(S_{t+1})$ is called the _TD target_.

$\delta_t = R_{t+1} + \gamma V(S_{t+1}) - V(S_t)$ is called the _TD error_ (the difference between our estimate before and after taking the step).

The fact that TD methods do not require complete methods means that it can be applied to non-terminating environments.

TDs are often more efficient for Markov environments because they exploit the Markov property (assuming the current state summarizes all previous states). MC methods, on the other hand, do not exploit this property.

Both MC and TD use samples, whereas dynamic programming does not (dynamic programming is exhaustive).

### $TD(\lambda)$

A compromise between TD (looks 1 step into the future) and MC (looks all the way to the end) is to look $n$ steps into the future (i.e. we observe $n$ steps and update the estimate $n$ steps in the past).

So we define the $n$-step return as:

$$
G_t^{(n)} = R_{t+1} + \gamma R_{t+2} + \dots + \gamma^{n-1} R_{t+n} + \gamma^n V(S_{t+n})
$$

So $n$-step temporal-difference learning's update is:

$$
V(S_t) \leftarrow V(S_t) + \alpha (G_t^{(n)} - V(S_t))
$$

We don't have to commit to one value for $n$, we can actually average over $n$-step returns over different $n$. In fact, we can consider _all_ $n$ values with $TD(\lambda)$. Here, we compute the $\lambda$-return $G_t^{\lambda}$ which combines all $n$-step returns $G_t^{(n)}$:

$$
G_t^{\lambda} = (1 - \lambda) \sum_{n=1}^{\infty} \lambda^{n-1} G_t^{(n)}
$$

$\lambda$ takes a value from 0 to 1.

So the update is (this is called _forward-view_ $TD(\lambda)$):

$$
V(S_t) \leftarrow V(S_t) + \alpha (G_t^{\lambda} - V(S_t))
$$

Note that like MC this can only be computed from complete episodes.

There is _backward-view_ $TD(\lambda)$ which allows online updates at every step (i.e. from incomplete sequences).

We compute _eligibility traces_ for the credit assignment problem which helps us figure out the eligibility of the states we saw for credit assignment (i.e. responsibility for the reward at the end). These combine the frequency heuristic (assign credit to the most frequent states) and the recency heuristic (assign credit to the most recent states):

$$
\begin{aligned}
E_0(s) &= 0 \\
E_t(s) &= \gamma \lambda E_{t-1}(s) + \mathbb{1}(S_t = s)
\end{aligned}
$$

So in backward-view $TD(\lambda)$ we keep an eligibility trace for every state $s$. We update value $V(s)$ for every state $s$ in proportion to TD-error $\delta t$ and eligibility trace $E_t(s)$:

$$
\begin{aligned}
\delta_t &= R_{t+1} + \gamma V(S_{t+1}) - V(S_t) \\
V(s) \leftarrow V(s) + \alpha \delta_t E_t(s)
\end{aligned}
$$

When $\lambda=0$ we have the "vanilla" TD algorithm as presented earlier.

## Model-Free Control

So how does an agent in a new environment learn how to maximize its reward? Another way of putting this: how do we optimize the value function of an unknown MDP?

There are two main paradigms for control:

- _on-policy_: learning about policy $\pi$ while sampling experiences from following $\pi$
- _off-policy_: learning about policy $\pi$ while sampling experiences from following some other policy $\mu$

Generally, the MDP might be unknown, but we can sample experiences, or the MDP might be known, but too big to use except by sampling it. So in either case, model-free control is appropriate.

We can take approach as we saw with dynamic programming: generalized policy iteration. That is, we evaluate our policy (estimate $v_{\pi}$), e.g. iterative policy evaluation, and then improve our policy (generate $\pi' \geq \pi$), e.g. greedy policy improvement.

So generalized policy iteration has two components:

- policy evaluation
- policy improvement (e.g. greedy policy improvement)

### Monte Carlo control

Unfortunately we can't just drop in a model-free policy evaluation method (e.g. Monte-Carlo evaluation) as our policy evaluation method to use with greedy policy improvement.

One reason is because of exploration - with model-free policy evaluation methods, we only sample experiences so there may be some we miss out on. If we use greedy policy improvement, we'll never explore those other experiences.

Another is because greedy policy improvement over $V(s)$ requires a model of the MDP:

$$
\pi'(s) = \argmax_{a \in A} R_s^a + P_{ss'}^a V(s')
$$

That is, we need $P_{ss'}^a$ and $R_s^a$, which we don't know (because we are model-free).

However, greedy policy improvement over $Q(s,a)$ is model-free:

$$
\pi'(s) = \argmax_{a \in A} Q(s,a)
$$

So instead our Monte-Carlo policy evaluation estimates $q_{\pi}$ instead of $v_{\pi}$, and now we can use it with greedy policy improvement.

However, we still have the problem of exploration.

One way around this is _$\epsilon$-greedy exploration_, where with probability $1 - \epsilon$ we choose the greedy action and with probability $\epsilon$ we choose an action at random.

For any $\epsilon$-greedy policy $\pi$, the $\epsilon$-greedy policy $\pi'$ with respect to $q_{\pi}$ is guaranteed to be an improvement, i.e. $v_{\pi}'(s) \geq v_{\pi}(s)$:

$$
\begin{aligned}
q_{\pi} (s, \pi'(s)) &= \sum_{a \in A} \pi'(a|s) q_{\pi}(s,a) \\
&= \frac{\epsilon}{m} \sum_{a \in A} q_{\pi}(s,a) + (1 - \epsilon) \max_{a \in A} q_{\pi}(s,a) \\
&\geq \frac{\epsilon}{m} \sum_{a \in A} q_{\pi}(s,a) + (1 - \epsilon) \sum_{a \in A} \frac{\pi(a|s) - \frac{\epsilon}{m}}{1 - \epsilon} q_{\pi}(s,a) \\
&= \sum_{a \in A} \pi(a|s) q_{\pi}(s,a) = v_{\pi}(s)
\end{aligned}
$$

So now our policy iteration strategy has become:

- policy evaluation: Monte-Carlo policy evaluation
- policy improvement: $\epsilon$-greedy policy improvement
- every episode

We can do this update after each episode.

Given infinite time, this will eventually explore all states, but it could happen very slowly (there may be some hard-to-access states that take awhile to randomly reach). In practice, we don't want to explore ad infinitum, we want to explore until we find an optimal policy, then stop exploring.

_Greedy in the limit with infinite exploration_ (GLIE) has two properties:

- all state-action pairs are explored infinitely many times: $\lim_{k \to \infty} N_l(s,a) = \infty$
- the policy converges on a greedy policy: $\lim_{k \to \infty} \pi_k (a|s) = \mathbb{1} (a = \argmax_{a' \in A} Q_k(s, a'))$

For example, $\epsilon$-greedy is GLIE if $\epsilon$ reduces to zero at $\epsilon_k = \frac{1}{k}$ (i.e. decay $\epsilon$ slowly towards 0).

We can modify our Monte-Carlo control to be GLIE:

- sample $k$th episode using $\pi$: $\{S_1, A_1, R_2, \dots, S_T\} \sim \pi$
- for each state $S_t$ and action $A_t$ in the episode:

$$
\begin{aligned}
N(S_t, A_t) &\leftarrow N(S_t, A_t) + 1 \\
Q(S_t, A_t) &\leftarrow Q(S_t, A_t) + \frac{1}{N(S_t, A_t)} (G_t - Q(S_t, A_t))
\end{aligned}
$$

That is, we keep track of how many times we've encountered this state-action pair and update its values according to how many times we've encountered it.

Then we improve the policy:

$$
\begin{aligned}
\epsilon &\leftarrow \frac{1}{k} \\
\pi &\leftarrow \epsilon\text{-greedy}(Q)
\end{aligned}
$$

This converges to the optimal action-value function $q^*(s,a)$.

### Temporal-difference control

TD learning has several advantages over MC:

- lower variance
- online
- can use incomplete sequences

So we just use TD instead of MC for policy evaluation, where we apply TD to $Q(S,A)$, use $\epsilon$-greedy policy improvement, and update every time-step (as opposed to between episodes). This method is called _Sarsa_:

$$
Q(S,A) \leftarrow Q(S,A) + \alpha (R + \gamma Q(S', A') - Q(S, A))
$$

We start with some state-action pair $S,A$, we see a reward $R$, end up in state $S'$, and sample the policy again to generate $A'$ (hence "SARSA").

So our (on-policy) policy iteration strategy has become:

- policy evaluation: Sarsa
- policy improvement: $\epsilon$-greedy policy improvement
- every time-step

In greater detail:

- arbitrarily initialize $Q(s,a)$ for each $s \in \mathcal S, a \in \mathcal A(s)$ and $Q(\text{terminal state}, \cdot) = 0$
- repeat, for each episode:
    - initialize $S$
    - choose $A$ from $S$ using policy derived from $Q$ (e.g. $\epsilon$-greedy)
    - repeat, for each step of episode:
        - take action $A$, observe $R, S'$
        - choose $A'$ from $S'$ using policy derived from Q (e.g. $\epsilon$-greedy)
        - $Q(S,A) \leftarrow Q(S,A) + \alpha (R + \gamma Q(S', A') - Q(S, A))$
        - $S \leftarrow S'$, $A \leftarrow A'$
    - until $S$ is terminal

Here $Q$ is a lookup-table, but it doesn't have to be (later other forms will be covered).

This is on-policy because we are evaluating and improving the policy we are currently using.

Sarsa converges to the optimal action-value function $Q(s,a) \to q^*(s,a)$ under the following conditions:

- GLIE sequence of policies $pi_t(a|s)$ (e.g. can use the same modifications we did with MC above)
- Robbins-Monro sequence of step-sizes $\alpha_t$:

$$
\begin{aligned}
\sum_{t=1}^{\infty} \alpha_t &= \infty \\
\sum_{t=1}^{\infty} \alpha_t^2 &< \infty
\end{aligned}
$$

These basically say:

- your step size is sufficiently large that your $Q$ value can change as much as it needs to (e.g. if your initial estimate is way off)
- eventually the changes to your $Q$ values need to trend towards 0

In practice we often don't worry about the Robbins-Monro sequence of step-sizes, and sometimes even GLIE, and Sarsa still works out.

### $n$-step Sarsa

What is the middle ground between TD and MC control?

We can again consider $n$-step returns, like we did previously comparing TD and MC evaluation methods.

The $n$-step Q-return would be:

$$
q_t^{(n)} = R_{t+1} + \gamma R_{t+2} + \dots + \gamma^{n-1} R_{t+n} + \gamma^n Q(S_{t+n})
$$

Then $n$-step Sarsa updates $Q(s,a)$ towards the $n$-step Q-return:

$$
Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha (q_t^{(n)} - Q(S_t, A_t))
$$

From this we get Sarsa$(\lambda)$ and there are forward view and backward view variants like there are with $TD(\lambda)$.

With forward view Sarsa$(\lambda)$ we also use weight $(1 - \lambda)\lambda^{n-1}$, i.e.:

$$
q_t^{\lambda} = (1 - \lambda) \sum_{n=1}^{\infty} \lambda^{n-1} q_t^{(n)}
$$

And use this with the general $n$-step Sarsa update above:

$$
Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha (q_t^{(n)} - Q(S_t, A_t))
$$

And in this way we are averaging over these $n$-step Q-returns.

With backward view Sarsa$(\lambda)$ we can again use eligibility traces in an online algorithm, the difference from $TD(\lambda)$ is that Sarsa$(\lambda)$ has one eligibility trace for each state-action pair:

$$
\begin{aligned}
E_0(s,a) &= 0 \\
E_t(s,a) = \gamma \lambda E_{t-1}(s,a) + \mathbb 1 (S_t = s, A_t = a)
\end{aligned}
$$

Basically this says, every time we visit a state-action pair $(s,a)$ we increment its eligibility trace by 1 (using the indicator variable), and we also decay the eligibility trace of all state-action pairs (even the ones we don't visit) by a little.

$Q(s,a)$ is updated for every state $s$ and action $a$ in proportion to TD-error $\delta_t$ (i.e. the difference between actual reward and what we expected) and eligibility trace $E_t(s,a)$:

$$
\begin{aligned}
\delta_t &= R_{t+1} + \gamma Q(S_{t+1}, A_{t+1}) - Q(S_t, A_t) \\
Q(s,a) &\leftarrow Q(s,a) + \alpha \delta_t E_t(s,a)
\end{aligned}
$$

Sarsa$(\lambda)$ in more detail:

- arbitrarily initialize $Q(s,a)$ for all $s \in \mathcal S, a \in \mathcal A(s)$
- repeat (for each episode):
    - $E(s,a) = 0$ for all $s \in \mathcal S, a \in \mathcal A(s)$
    - initialize $S, A$
    - repeat (for each step of episode):
        - take action $A$, observe $R, S'$
        - choose $A'$ from $S'$ using policy derived from $Q$ (e.g. $\epsilon$-greedy)
        - $\delta \leftarrow R + \gamma Q(S', A') - Q(S,A)$
        - $E(S,A) \leftarrow E(S,A) + 1$
        - for all $s \in mathcal S, a \in \mathcal A(s)$:
            - $Q(s,a) \leftarrow Q(s,a) + \alpha \delta E(s,a)$
            - $E(s,a) \leftarrow \gamma \lambda E(s,a)$
        - $S \leftarrow S'$, $A \leftarrow A'$
    - until $S$ is terminal


Essentially what the $\lambda$ value dictates is how far information propagates backwards through the trajectory the agent took.

Consider a situation where a long path is taken, with zero reward at each step, until the agent reaches a terminal state with a positive reward (one episode).

With $\lambda=0$, i.e. one-step Sarsa, only the state immediately before the terminal state is updated; i.e. the reward propagates backwards only one step. The agent would have to travel along that same path many times for the reward to propagate further along that trajectory.

With $\lambda > 0$, the reward propagates backwards further in one episode (each previous step has its eligibility trace increased, i.e. it is assigned more responsibility for leading to that reward)

### Off-policy learning

Evaluate target policy $\pi(a|s)$ to compute $v_{\pi}(s)$ or $q_{\pi}(s,a)$ while following a different behavior policy $\mu(a|s)$ (i.e. we use this policy to choose actions).

This can be thought of as "learning by observation", i.e. the agent watches another agent perform some task, and from that observation it can actually learn a better policy than the one it observed.

This can also be thought of as "reusing" experience learned from old policies.

Off-policy learning has a couple advantages in what it enables us to do:

- learn about an optimal policy while following an _exploratory_ policy
- learn about _multiple_ policies while following one policy

So how can we do this?

#### Importance sampling

Importance sampling gives us a means to estimate the expectation of a different distribution using another distribution:

$$
\begin{aligned}
E_{X \sim P}[f(X)] &= \sum P(X) f(X) \\
&= \sum Q(X) \frac{P(X)}{Q(X)}f(X) \\
&= E_{X \sim Q}[\frac{P(X)}{Q(X)}f(X)]
\end{aligned}
$$

For off-policy Monte Carlo, we can apply important sampling and use returns generated from $\mu$ to evaluate $\pi$. We weight the return $G_t$ according to the similarity between policies and multiply importance sampling corrections along the whole episode:

$$
G_t^{\pi/\mu} = \frac{\pi(A_t|S_t)}{\mu(A_t|S_t)} \frac{\pi(A_{t+1}|S_{t+1})}{\mu(A_{t+1}|S_{t+1})} \dots \frac{\pi(A_T|S_T)}{\mu(A_T|S_T)} G_t
$$

Then update value towards the corrected return:

$$
V(S_t) \leftarrow V(S_t) + \alpha (G_t^{\pi/\mu} - V(S_t))
$$

(we can't use this if $\mu$ is zero when $\pi$ is non-zero)

However, importance sampling can increase variance to be too high to be practical (since Monte Carlo learning is already high variance). Because of the high variance, the target and behavior policies never get close enough to be useful.

So off-policy TD is preferred; it has much lower variance. We use the TD targets generated from $\mu$ to evaluate $\pi$, and we weight the TD target $R + \gamma V(S')$ by importance sampling. We only need to importance sample over one step because we're bootstrapping off of one step:

$$
V(S_t) \leftarrow V(S_t) + \alpha (\frac{\pi(A_t|S_t)}{\mu(A_t|S_t)} (R_{t+1} + \gamma V(S_{t+1})) - V(S_t))
$$

### Q-learning

The best offline-policy is Q-learning, which does not require importance sampling.

We choose the next action using the behavior policy, $A_{t+1} \sim \mu(\cdot | S_t)$.

We also consider an alternative successor action from our target policy, $A' \sim \pi(\cdot | S_t)$.

Then we update $Q(S_t, A_t)$ (of the action we actually took) towards the value of the alternative action:

$$
Q(S_t, A_t) \leftarrow Q(S_t, A_t) + \alpha (R_{t+1} + \gamma Q(S_{t+1}, A') - Q(S_t, A_t))
$$

There's a special case of Q-learning which is usually what is meant when the Q-learning algorithm is referred to, where the target policy $\pi$ is greedy with respect to $Q(s,a)$:

$$
\pi(S_{t+1}) = \argmax_{a'} Q(S_{t+1}, a')
$$

The behavior policy $\mu$ is, for example, $\epsilon$-greedy with respect to $Q(s,a)$.

The Q-learning target then simplifies:

$$
\begin{aligned}
&R_{t+1} + \gamma Q (S_{t+1}, A') \\
&R_{t+1} + \gamma Q (S_{t+1}, argmax_{a'} Q(S_{t+1}, a')) \\
&R_{t+1} + \max_{a'} \gamma Q (S_{t+1}, a')
\end{aligned}
$$

What happens here is both the behavior and target policies improve.

So the updates then are:

$$
Q(S,A) \leftarrow Q(S,A) + \alpha (R+ \gamma \max_{a'} Q(S', a') - Q(S,A))
$$

and it converges to the optimal action-value function.

## Value Function Approximation

If state spaces get very large (e.g. with Go, with $10^{170}$ states, or any problem with a continuous state space), the methods discussed so far (i.e. these tabular methods where we essentially maintain a lookup table of values, e.g. a table of values where each state $s$ has an entry $V(s)$ or where every state-action pair $s,a$ has an entry $Q(s,a)$) will not scale. We won't have enough memory or they will be very slow to learn (too many spaces to explore).

Instead, we can consider _value function approximation_ in which we don't need to distinctly map each state to some value, rather we use some function to approximate the value of states, such that nearby states take similar values (i.e. generalize to states we have not encountered before).

These allow us to scale up the model-free methods we've seen so far for both prediction and control.


So there is some true value function for a policy $v_{\pi}(s)$ (or action-value function $q_\pi(s,a)$). With value function approximation we estimate it by fitting a function to it (e.g. with a neural network):

$$
\begin{aligned}
\hat v(s, w) &\approx v_{\pi}(s) \\
\hat q(s, a, w) &\approx q_{\pi}(s, a)
\end{aligned}
$$

where $w$ is some vector of parameters/weights. We can update it using MC or TD learning.

The function we learn could:

- take an input state $s$ and return $\hat v(s,w)$
- take an input state $s$ and an input action $a$ and return $\hat q(s,a,w)$
- take an input state $s$ and return values for all actions $\hat q(s,a,w) \forall a \in \mathcal A$

We want our training method to work with non-stationary (our policy will be changing, causing its value function to change), non-iid (e.g. where I am next correlates highly with where I just was) data. This is a main difference between supervised and reinforcement learning.

Broadly, we can break value function approximation into two categories:

- _incremental methods_, in which the approximator (e.g. a neural network) makes an update after each time step (somewhat similar to online methods, but not exactly)
- _batch methods_, where the approximator looks at whole sets of data

As an aside, note that table lookup is actually a special case of linear value function approximation, where each state is a one-hot vector and its associated weight is its lookup value.

### Incremental methods

Basically, we use stochastic gradient descent to fit the approximator.

We can represent states with a feature vector.

Instead of computing mean squared error with respect to the true value function $v_{\pi}(s)$ (which we don't know), we use a target, i.e.:

- for MC, the target is the return $G_t$, so the parameter update is:

$$
\Delta w = \alpha(G_t - \hat v(S_t, w)) \nabla_w \hat v (S_t, w)
$$

- for $TD(0)$, the target is the TD target $R_{t+1} + \gamma \hat v(S_{t+1}, w)$, so the parameter update is:

$$
\Delta w = \alpha(R_{t+1} + \gamma \hat v(S_{t+1}, w) - \hat v(S_t, w)) \nabla_w \hat v (S_t, w)
$$

- for $TD(\lambda)$, the target is the $\lambda$-return $G_t^{\lambda}$

$$
\Delta w = \alpha(G_t^{\lambda} - \hat v(S_t, w)) \nabla_w \hat v (S_t, w)
$$

Monte-Carlo with value function approximation involves running episodes, and these episodes essentially are "training data", e.g.:

$$
(S_1, G_1), (S_2, G_2), \dots, (S_T, G_T)
$$

The result is basically a supervised learning problem. This will converge to a local optimum in both linear and non-linear cases.

TD learning ($TD(0)$) with value function approximation is similar in that we also assemble "training data":

$$
(S_1, R_2 + \gamma \hat v(S_2, w)), (S_2, R_3 + \gamma \hat v(S_3, w)), \dots, (S_{T-1}, R_T)
$$

But note that unlike Monte-Carlo, this sample is biased because we are using our neural network to estimate further reward at each step.

Linear $TD(0)$ converges (close) to a global optimum.

$TD(\lambda)$ also produces a (biased) sample of "training data":

$$
(S_1, G_1^{\lambda}), (S_2, G_2^{\lambda}), \dots, (S_{T-1}, G_{T-1}^{\lambda})
$$

### Control with value approximation

- Policy evaluation: approximate policy evaluation $\hat q(\cdot, cdot, w) \approx q_{\pi}$
- Policy improvement: $\epsilon$-greedy policy improvement

So we want to approximate the action-value function (so we can remain model-free):

$$
\hat q(S,A,w) \approx q_{\pi}(S,A)
$$

Again we want to minimize the mean-squared error between the approximate action-value function $\hat q(S,A,w)$ and the true action-value function $q_{\pi}(S,A)$ and we again can use stochastic gradient descent.

We can represent both the states and actions with feature vectors.

Like with approximating the value function, we don't know the true action-value function, so we can't optimize against it. Rather, we again substitute a target for $q_{\pi}(S,A)$:

- for MC, the target is the return $G_t$, so the parameter update is:

$$
\Delta w = \alpha(G_t - \hat q(S_t, A_t, w)) \nabla_w \hat q (S_t, A_t, w)
$$

- for $TD(0)$, the target is the TD target $R_{t+1} + \gamma \hat q(S_{t+1}, A_{t+1}, w)$, so the parameter update is:

$$
\Delta w = \alpha(R_{t+1} + \gamma \hat q(S_{t+1}, A_{t+1}, w) - \hat q(S_t, A_t, w)) \nabla_w \hat v (S_t, w)
$$

- for $TD(\lambda)$, the target is the $\lambda$-return $q_t^{\lambda}$. For forward-view $TD(\lambda)$:

$$
\Delta w = \alpha(q_t^{\lambda} - \hat q(S_t, A_t, w)) \nabla_w \hat q (S_t, A_t, w)
$$

For backward-view $TD(\lambda)$:

$$
\begin{aligned}
\delta_t &= R_{t+1} + \gamma \hat q(S_{t+1}, A_{t+1}, w) - \hat q(S_t, A_t, w) \\
E_t &= \gamma \lambda E_{t-1} + \nabla_w \hat q(S_t, A_t, w) \\
\Delta w &= \alpha \delta_1 E_t
\end{aligned}
$$

Note that TD (weights) can sometimes blow up. A rough summary of when TD is guaranteed to converge ("ok") and when it _might_ diverge ("maybe"):

| On/Off-Policy | Algorithm     | Table Lookup | Linear | Non-Linear |
|---------------|---------------|--------------|--------|------------|
| On            | MC            | ok           | ok     | ok         |
|               | TD(0)         | ok           | ok     | maybe      |
|               | TD($\lambda$) | ok           | ok     | maybe      |
| Off           | MC            | ok           | ok     | ok         |
|               | TD(0)         | ok           | maybe  | maybe      |
|               | TD($\lambda$) | ok           | maybe  | maybe      |

Although MC works in all cases, it can be very, very slow (variance is too high), so we generally want to bootstrap (i.e  have $\lambda \neq 1$).

The reason TD may diverge in off-policy or non-linear approximations is because it does not follow the gradient of _any_ objective function.

_Gradient TD_ on the other hand follows the true gradient of projected Bellman error, so it will guaranteed converge for all cases.

For control algorithms:

| Algorithm           | Table Lookup | Linear | Non-linear |
|---------------------|--------------|--------|------------|
| Monte-Carlo Control | ok           | ~ok    | maybe      |
| Sarsa               | ok           | ~ok    | maybe      |
| Q-learning          | ok           | maybe  | maybe      |
| Gradient Q-learning | ok           | ok     | maybe      |

Where "~ok" means it wiggles around the near-optimal value function (it might get a little worse, then get better, etc; we aren't guaranteed that every update will be an improvement).

### Batch methods

The incremental methods waste experience - we have an experience, use it to update, then we toss that experience away. They aren't "sample efficient".

Batch methods try to find the value function that best fits all the data we've seen (all the experience).

With _experience replay_ we just store all the experiences. Then, at every time step, we sample from these stored experiences and use it for a stochastic gradient descent update.

_Deep Q-networks_ use experience replay and fixed Q-targets (essentially Q-learning):

- take action $a_t$ according to $\epsilon$-greedy policy
- store transition $(s_t, a_t, r_{t+1}, s_{t+1})$ in replace memory $\mathcal D$
- sample random mini-batch of transitions $(s,a,r,s')$ from $\mathcal D$
- compute Q-learning targets wrt to old, fixed parameters $w^-$
- optimize MSE between Q-network and Q-learning targets using variant of stochastic gradient descent:

$$
L_i(w_i) = \mathbb E_{s,a,r,s' \sim \mathcal D_i} [(r + \gamma \max_{a'} Q(s', a'; w_i^-) - Q(s,a,;w_i))^2]
$$

What prevents this from blowing up is experience replay and the fixed Q-targets.

Experience replay stabilizes updates by decorrelating the data (you're not seeing them in a specific sequence; they're randomized by sampling past experiences)

Here there are actually two neural networks (one with parameters $w^-$ and one with parameters $w$). One network is basically the "frozen" old network, i.e. with the old parameters $w^-$, and we use that for bootstrapping (instead of always using the latest updated parameters) for some iterations, then we freeze the newer network as the old network and repeat. This also stabilizes the learning by decorrelating.

## Policy Gradient methods

Policy gradient methods work directly with the policy, i.e. without using a value or action-value function. The general class of methods that directly learn a policy is called _policy-based_ reinforcement learning (in contrast to _value-based_ reinforcement learning which just learns a value function).

To review: we have thus far tried to learn a value (or action-value) function (e.g. approximating it with parameters $\theta$):

$$
\begin{aligned}
V_{\theta}(s) &\approx V^{\pi}(s) \\
Q_{\theta}(s, a) &\approx Q^{\pi}(s, a)
\end{aligned}
$$

and generated an _implicit_ policy _from_ this value function by using some policy improvement method, e.g. $\epsilon$-greedy. We don't learn a policy directly, we just use the value or action-value function to pick actions.

Policy gradient methods directly parametrize the policy itself:

$$
\pi_{\theta}(s,a) = \mathbb P[a | s, \theta]
$$

So the policy itself can be approximated by some function approximator, e.g. a neural network.

Here we again focus on model-free methods.

Advantages of policy-based RL:

- better convergence properties: remember that value-based methods could wiggle around points or even diverge under certain conditions; policy-based methods tend to be more stable since you're just following the policy gradient
- effective in high-dimensional or continuous action spaces: we don't have to deal with any maximums, i.e. we don't have to figure out the maximum-valued action; this is good because that maximization itself can be very expensive, e.g. in continuous action spaces where there are many, many actions to maximize over. Policy-based methods give us the action to take in a more direct way.
- can learn stochastic policies: deterministic policies may be easily exploited (e.g. in rock-paper-scissors, where the optimal policy is actually a uniform random policy)

Disadvantages:

- typically converge to a local rather than global optimum
- evaluating a policy is typically inefficient and high variance: value-based methods, by using the max, aggressively push you towards what you think is the best policy, but policy gradient methods take small steps in the right direction.

Learning stochastic policies are useful in partially-observed environments (i.e. we are dealing with an POMDP rather than an MDP), i.e. when we have limited information about states.

A simple example is the aliased gridworld, where we don't know our exact position; rather, we only have features like "is there a wall to my north", "is there a wall to my south", etc. _State aliasing_ refers to environments where states which may actually be different can't be distinguished given the features we know about. For example, for the following grid world:

    _____________________
    | 0 | 1 | 2 | 3 | 4 |
    |___|___|___|___|___|
    | 5 |   | 6 |   | 7 |
    |___|   |___|   |___|

Cells 1 and 3 are aliased, as are 5, 6 and 7. From the perspective of the agent, 1 and 3 are identical (both have walls just to the north and south), and 5, 6, and 7 are identical (both have walls to the west, east, and south).

A deterministic policy must always take the same action from aliased states. For example, if it learns to go left in cell 3, it will also go left in cell 1 (the two states are identical as far as the agent is concerned).

A stochastic policy, on the other hand, may learn to go left or right with 50/50 probability in cell 3 and will do the same for cell 1.

Remember that there is a deterministic optimal policy for any MDP, but this is not the case for POMDPs, where optimal policies may be stochastic.

### Policy objective functions

So how do we optimize a policy? Remember, we want to find the best parameters $\theta$ for a given policy $\pi_{\theta}(s,a)$. So we need some way to measure the quality of policy $\pi_{\theta}$, i.e. an objective function.

We have a few options:

- in episodic environments (i.e. where there is a clear notion of a "starting state") we can use the _start value_:

$$
J_1(\theta) = V^{\pi_{\theta}}(s_1) = \mathbb E_{\pi_{\theta}}[v_1]
$$

- in continuing environments we can use the _average value_:

$$
J_{\text{av}V}(\theta) = \sum_s d^{\pi_{\theta}}(s) V^{\pi_{\theta}}(s)
$$

- or the _average reward per time-step_:

$$
J_{\text{av}R}(\theta) = \sum_s d^{\pi_{\theta}}(s) \sum_a \pi_{\theta} (s,a) R_s^a
$$

where $d^{\pi_{\theta}}(s)$ is the stationary distribution of the Markov chain for $\pi_{\theta}}$.

Fortunately, the policy gradient is essentially the same for all of these objective functions.

So we want to optimize the policy, i.e. find $\theta$ that maximizes $J(\theta)$.

Let $J(\theta)$ be any policy objective function.

Policy gradient algorithms search for a local maximum in $J(\theta)$ by ascending the gradient of the policy wrt parameters $\theta$ (i.e. gradient ascent):

$$
\Delta \theta = \alpha \nabla_{\theta} J(\theta)
$$

where $\nabla_{\theta} J(\theta)$ is the policy gradient, i.e.:

$$
\nabla_{\theta} J(\theta) = \begin{bmatrix}
\frac{\partial J(\theta)}{\partial \theta_1} \\
\vdots
\frac{\partial J(\theta)}{\partial \theta_n}
\end{bmatrix}
$$

and $\alpha$ is a step-size parameter.

### Finite difference policy gradient

Computing gradients by finite differences just involves perturbing the parameters in each dimension and seeing if the objective function result is better:

- for each dimension $k \in [1,n]$
    - estimate $k$th partial derivative of objective function wrt $\theta$ by perturbing $\theta$ by a small amount $\epsilon$ in the $k$th dimension:

$$
\frac{\partial J(\theta)}{\partial \theta_k} \approx \frac{J(\theta + \epsilon u_k) - J(\theta)}{\epsilon}
$$

where $u_k$ is the unity vector with 1 in the $k$th component and 0 elsewhere.

This requires $n$ evaluations to estimate the policy gradient in $n$ dimensions. It is simple, noisy, inefficient, but sometimes effective, and it works for arbitrary policies, even if it is not differentiable.

### Monte Carlo policy gradient

There are analytic ways of computing the gradient so we don't have to numerically compute it in each dimension.

We assume the policy $\pi_{\theta}$ is differentiable whenever it is non-zero (i.e. wherever you are taking actions) and that we know the gradient $\nabla_{\theta} \pi_{\theta}(s,a)$ (because we define the structure of the policy, e.g. a neural network).

We can use _likelihood ratios_. These exploit the following identity:

$$
\begin{aligned}
\nabla_{\theta} \pi_{\theta} (s,a) &= \pi_{\theta}(s,a) \frac{\nabla_{\theta} \pi_{\theta}(s,a)}{\pi_{\theta}(s,a)} \\
&= \pi_{\theta}(s,a) \nabla_{\theta} \log \pi_{\theta}(s,a)
\end{aligned}
$$

$\nabla_{\theta} \log \pi_{\theta}(s,a)$ is called the _score function_. You see this in maximum likelihood learning; here this essentially tells us how to adjust the policy to get more of a particular action.

For example, consider a (linear) softmax policy. We weight actions using a linear combination of features $\phi(s,a)^T \theta$. The probability of an action is proportional to exponentiated weight:

$$
\pi_{\theta}(s,a) \propto e^{\phi(s,a)^T \theta}
$$

The score function is:

$$
\nabla_{\theta} \log \pi_{\theta}(s,a) = \phi(s,a) - \mathbb E_{\pi_{\theta}} [\phi(s, \cdot)]
$$

i.e. the feature for the action we took ($\phi(s,a)$) minus the average feature for all actions we might have taken.

For another example, we could use a (linear) Gaussian policy, which is good for continuous action spaces. The mean is a linear combination of state features $\mu(s) = \phi(s)^T \theta$. The variance may be fixed $\sigma^2$ or also parametrized.

The policy is Gaussian, so $a \sim \mathcal N(\mu(s), \sigma^2)$.

The score function is:

$$
\nabla_{\theta} \log \pi_{\theta}(s,a) = \frac{(a - \mu(s))\phi(s)}{\sigma^2}
$$

So how do we apply this?

Consider the simple case of a one-step MDP, i.e. our episodes consist of only one step. So we start in some state $s \sim d(s)$ and terminate after one time step with reward $r = \mathcal R_{s,a}$.

We use likelihood ratios to compute the policy gradient:

$$
\begin{aligned}
J(\theta) &= \mathbb E_{\pi_{\theta}}[r] \\
&= \sum_{s \in \mathcal S} d(s) \sum_{a \in \mathcal A} \pi_{\theta}(s,a) \mathcal R_{s,a} \\
\nabla_{\theta} J(\theta) &= \sum_{s \in \mathcal S} d(s) \sum_{a \in A} \pi_{\theta}(s,a) \nabla_{\theta} \log \pi_{\theta}(s,a) \mathcal R_{s,a} \\
&= \mathbb E_{\pi_{\theta}} [\nabla_{\theta} \log \pi_{\theta}(s,a) r]
\end{aligned}
$$

You can see that this is still model-free since we replace $R_{s,a}$ with an explicit value $r$, that is, we don't need to model the reward function; we only need to look at the actual reward $r$ we received.

We can expand this to multi-step MDPs by just replacing the instantaneous reward $r$ with the long-term value $Q^{\pi}(s,a)$.

The _policy gradient theorem_ tells us that this ends up being the true gradient of the policy (this holds for any of the policy objectives we mentioned, i.e. start state objective, average reward and average value objectives):

The policy gradient theorem: For any differentiable policy $\pi_{\theta}(s,a)$, for any of the policy objective functions  $J = J_1, J_{\text{av}R}, \frac{1}{1 - \gamma}J_{\text{av}V}$, the policy gradient is:

$$
\nabla_{\theta} J(\theta) = \mathbb E_{\pi_{\theta}} [\nabla_{\theta} \log \pi_{\theta}(s,a) Q^{\pi_{\theta}}(s,a)]
$$

We can use this for _Monte-Carlo Policy Gradient_, which is essentially the _REINFORCE_ algorithm:

- update parameters by stochastic gradient ascent (get rid of the expectation, just sample it instead, see below)
- using policy gradient theorem
- using return $v_t$ as an unbiased sample of $Q^{\pi_{\theta}}(s_t, a_t)$:

$$
\Delta \theta_t = \alpha \nabla_{\theta} \log \pi_{\theta}(s_t,a_t) v_t
$$

REINFORCE:

- arbitrarily initialize $\theta$
- for each episode ${s_1, a_1, r_2, \dots, s_{T-1}, a_{T-1}, r_T} \sim \pi_{\theta}$ do (at the end of the episode):
    - for $t = 1$ to $T-1$ do:
        - $\theta \leftarrow \theta + \alpha \nabla_{\theta} \log \pi_{\theta}(s_t,a_t) v_t$
- return $\theta$

where, like before, $v_t$ is the accumulated reward from that time step $t$ to the end of the episode.

Monte-Carlo gradient methods, however, tend to be slow, again, because of high variance in the sampled rewards (for example, if playing an Atari game, in one episode I may get 10,000 points and in the next I may get 0; over the course of any episode there are so many things that could occur that can lead to very different outcomes; it is very noisy).

### Actor-Critic policy gradient methods

_Actor-critic policy gradient methods_ try to get the best of both worlds - they learn a policy explicitly _and_ learn a value function.

We use a _critic_ to estimate the action-value function using a function approximator:

$$
Q_w(s,a) \approx Q^{\pi_{\theta}}(s,a)
$$

So instead of using the return to estimate the action-value function (which, as just stated, is high in variance), we estimate it directly with the critic. This reduces the variance so training is quicker.

So we have two sets of parameters:

- the critic: updates action-value function parameters $w$
- the actor: updates policy parameters $\theta$, in direction suggested by critic

So the actor is the one that actually has the policy and is deciding and taking actions, the critic is just evaluating the actor.

Actor-critic algorithms follow an _approximate_ policy gradient:

$$
\begin{aligned}
\nabla_{\theta} J(\theta) &\approx \mathbb E_{\pi_{\theta}} [\nabla_{\theta} \log \pi_{\theta}(s,a) Q_w(s,a)] \\
\Delta \theta &= \alpha \nabla_{\theta} \log \pi_{\theta}(s,a) Q_w(s,a)
\end{aligned}
$$

So we've replaced the true action value function $Q^{\pi_{\theta}}(s,a)$ with our estimate $Q_w(s,a)$.

So the critic is basically just incorporating policy evaluation, i.e. how good is policy $\pi_{\theta}$ for current parameters $\theta$? We've already looked at this with Monte-Carlo policy evaluation, Temporal-Difference learning, and $TD(\lambda)$.

For example, we could use a linear value function approximator $Q_w(s,a) = \phi(s,a)^T w$ and update the critic's parameters $w$ with linear $TD(0)$ and update the actor's parameters $\theta$ by policy gradient:

- initialize $s, \theta$
- sample $a \sim \pi_{\theta}$
- for each step do:
    - sample reward $r = \mathcal R_s^a$; sample transition $s' \sim P_s^a$
    - sample action $a' \sim \pi_{\theta}(s', a')$
    - $\delta = r + \gamma Q_w(s',a') - Q_w(s,a)$
    - $\theta = \theta + \alpha \nabla_{\theta} \log \pi_{\theta}(s,a) Q_w(s,a)$
    - $w \leftarrow w + \beta \delta \phi(s,a)$
    - $a \leftarrow a', s \leftarrow s'$

(note that because we are using TD we don't need to wait until the end of the episode, we can update at each step)

This is similar to policy iteration: we evaluate the policy (like value-based RL), then improve it, except now we're improving it explicitly using its gradient (like policy-based RL).

How can we improve this?

We can reduce variance using a _baseline_. We subtract a baseline function $B(s)$ from the policy gradient. This can reduce variance without changing expectation:

$$
\begin{aligned}
\mathbb E_{\pi_{\theta}} [\nabla_{\theta} \log \pi_{\theta}(s,a) B(s)] &= \sum_{s \in \mathcal S} d^{\pi_{\theta}}(s) \sum_a \nabla_{\theta}\pi_{\theta}(s,a) B(s) \\
&= \sum_{s \in \mathcal S} d^{\pi_{\theta}} B(s) \nabla_{\theta} \sum_{a \in \mathcal A} \pi_{\theta}(s,a) \\
&= 0
\end{aligned}
$$

So the expectation of this term is 0, so it does not affect the expectation wrt to our action-value function.

We choose a baseline function explicitly to reduce variance (e.g. mean 0, roughly the right scale, etc).

A good baseline is the state value function $B(s) = V^{\pi_{\theta}}(s)$.

So we subtract the state value function from the action-value function. This gives us what is called the _advantage function_ $A^{\pi_{\theta}}(s,a)$, which tells us how much better than usual is it to take action $a$.

So we can rewrite the policy gradient using the advantage function:

$$
\begin{aligned}
A^{\pi_{\theta}}(s,a) &= Q^{\pi_{\theta}}(s,a) - V^{\pi_{\theta}}(s) \\
\nabla_{\theta} J(\theta) &= \mathbb E_{\pi_{\theta}} [\nabla_{\theta} \log \pi_{\theta}(s,a) A^{\pi_{\theta}}(s,a)]
\end{aligned}
$$

So basically if we take an action that has a better-than-usual result, we update the policy to encourage (towards) the action; if worse-than-usual, we update the policy to discourage (away from) that action.

This can significantly reduce the variance of the policy gradient.

So what we really want is for the critic to estimate the advantage function, e.g. by estimating both $V^{\pi_{\theta}}(s)$ and $Q^{\pi_{\theta}}(s)$. So we can use two function approximators with two parameter vectors:

$$
\begin{aligned}
V_v(s) &\approx V^{\pi_{\theta}}(s) \\
Q_w(s,a) &\approx Q^{\pi_{\theta}}(s,a) \\
A(s,a) &= Q_w(s,a) - V_v(s)
\end{aligned}
$$

And then update both value functions by e.g. TD learning.

However, we can make this a bit simpler.

For the true value function $V^{\pi_{\theta}}(s)$, the TD error $\delta^{\pi_{\theta}}$ is:

$$
\delta^{\pi_{\theta}} = r + \gamma V^{\pi_{\theta}}(s') - V^{\pi_{\theta}}(s)
$$

and it is actually an unbiased estimate of the advantage function:

$$
\begin{aligned}
\mathbb E_{\pi_{\theta}} [\delta^{\pi_{\theta}}|s,a] &= \mathbb E_{\pi_{\theta}} [r + \gamma V^{\pi_{\theta}} (s') | s,a] - V^{\pi_{\theta}}(s) \\
&= Q^{\pi_{\theta}}(s,a) - V^{\pi_{\theta}}(s) \\
&= A^{\pi_{\theta}}(s,a)
\end{aligned}
$$

Remember that $E_{\pi_{\theta}} [r + \gamma V^{\pi_{\theta}} (s') | s,a]$ is the definition of $Q^{\pi_{\theta}}(s,a)$.

So we can just use the TD error the compute the policy gradient:

$$
\nabla_{\theta} J(\theta) = \mathbb E_{\pi_{\theta}} [\nabla_{\theta} \log \pi_{\theta}(s,a) \delta^{\pi_{\theta}}]
$$

In practice we can use an approximate TD error:

$$
\delta_v = r + \gamma V_v(s') - V_v(s)
$$

So the critic only needs to learn one set of parameters, $v$.

We already know how to have the critic work on different time-scales (e.g. Monte Carlo requires the full episode, $TD(0)$ only looks at one step, and $TD(\lambda)$ looks somewhere in between).

We can do the same for the actor; i.e. have it consider the critic's response from multiple time-steps, i.e. the policy gradient can also be estimated at many time-scales.

The Monte-Carlo policy gradient uses error from the complete return, $v_t$:

$$
\Delta \theta = \alpha (v_t - V_v(s_t))\nabla_{\theta} \log \pi_{\theta}(s_t,a_t)
$$

The actor-critic policy gradient uses the one-step TD error, $r + \gamma V_v(s_{t+1})$:

$$
\Delta \theta = \alpha (r + \gamma V_v(s_{t+1}) - V_v(s_t))\nabla_{\theta} \log \pi_{\theta}(s_t,a_t)
$$

(remember that the terms in parentheses, e.g. $v_t - V_v(s_t)$ and $r + \gamma V_v(s_{t+1}) - V_v(s_t)$, are estimates of the advantage function)

We can combine the policy gradient with eligibility traces to get what we want - to consider the critic's response from multiple time-steps.

Like forward-view $TD(\lambda)$ we can mix over time-scales:

$$
\Delta \theta = \alpha (v_t^{\lambda} - V_v(s_t))\nabla_{\theta} \log \pi_{\theta}(s_t,a_t)
$$

where $v_t^{\lambda} - V_v(s_t)$ is a biased estimate of the advantage function.

Like backward-view $TD(\lambda)$, we can also use eligibility traces:

- by equivalence with $TD(\lambda)$, subsisting $\phi(s) = \nabla_{\theta} \log \pi_{\theta}(s,a)$:

$$
\begin{aligned}
\delta &= r_{t+1} + \gamma V_v(s_{t+1}) - V_v(s_t) \\
e_{t+1} &= \lambda e_t + \nabla_{\theta} \log \pi_{\theta}(s,a) \\
\Delta \theta &= \alpha \delta e_t
\end{aligned}
$$

You can see it is similar to eligibility traces with value-based RL.

This update can be applied online, to incomplete sequences.

## Integrating Learning and Planning

### Model-based RL

Previously we learned value functions directly from experience, then we looked at how to learn policies directly from experience. Those methods were all model-free. Here we'll see how we can learn a model directly from experience; i.e. the agent learns the dynamics of the environment (i.e. the state transition function and the reward function).

Then we can use that model to _plan_; i.e. construct a value function or policy by looking ahead/imagining/simulating what will happen in the future.

The general idea:

- the agent gains experience by interacting with the real world to learn a model for the world (i.e. an MDP)
- the agent uses this model to plan (e.g. solve the learned MDP), generating a value function or a policy
- the agent uses this value function or policy to decide how to act
- repeat

Advantages of model-based RL:

- can be more compact than learning a value function or policy directly
- can efficiently learn a model by supervised learning methods
- can reason about model uncertainty

Disadvantages:

- first learn a model, the construct a value function: thus, two sources of approximation error

Formally, a model $\mathcal M$ is a representation of an MDP $(\mathcal S, \mathcal A, \mathcal P, \mathcal R)$ parametrized by $\eta$.

We assume the state space $\mathcal S$ and action space $\mathcal A$ are known (for now - there are forms of model-based RL that also learn the state and action spaces).

So a model $\mathcal M = (\mathcal P_{\eta}, \mathcal R_{\eta})$ represents state transitions $\mathcal P_{\eta} \approx \mathcal P$ and rewards $\mathcal R_{\eta} \approx \mathcal R$.

$$
\begin{aligned}
S_{t+1} &\sim \mathcal P_{\eta}(S_{t+1} | S_t, A_t) \\
R_{t+1} &= \mathcal R_{\eta} (R_{t+1} | S_t, A_t)
\end{aligned}
$$

We typically assume conditional independence between state transitions and rewards:

$$
\mathbb P [S_{t+1}, R_{t+1} | S_t, A_t] = \mathbb P [S_{t+1} | S_t, A_t] \mathbb P[R_{t+1} | S_t, A_t]
$$

So our goal is to estimate a model $M_{\eta}$ from experience $(S_1, A_1, R_2, \dots, S_T)$.

This can be framed as a supervised learning problem:

$$
\begin{aligned}
S_1, A_1 &\to R_2, S_2 \\
S_2, A_2 &\to R_3, S_3 \\
&\vdots \\
S_{T-1}, A_{T-1} &\to R_T, S_T
\end{aligned}
$$

Learning $s, a \to r$ is a regression problem, and learning $s, a \to s'$ is a density estimation problem (e.g. we learn a distribution over states).

So we pick a loss function, e.g. mean-squared error, KL divergence, and the find the parameters $\eta$ that minimize empirical loss.

Some example models:

- table lookup model
- linear expectation model
- linear gaussian model
- gaussian process model
- deep belief network model
- etc

As a really simple example, consider a table lookup model.

We can just model $\hat \mathcal P$ with counts, i.e. each time we end up in state $s'$ from state-action pair $s,a$, just increment the count of $s,a \to s'$ by 1. Then to use our model we just convert these counts into a probability distribution.

Similarly, to model rewards we can just take the average reward received for state-action pair $s,a$.

Alternatively, we can just record each experience tuple $(S_t, A_t, R_{t+1}, S_{t+1})$ at time $t$ and then, to sample the model, we just randomly pick a tuple matching $(s,a,\cdot,\cdot)$.

### Planning with a model

Given a model $\mathcal M_{\eta} = (\mathcal P_{\eta}, \mathcal R_{\eta})$, to plan we just solve the MDP $(\mathcal S, \mathcal A, \mathcal P_{\eta}, \mathcal R_{\eta})$ using a planning algorithm such as value iteration, policy iteration, tree search, etc.

One such algorithm is _sample-based planning_, which is a simple but powerful approach. We use the model _only_ to generate samples, i.e. we sample experience from the model:

$$
\begin{aligned}
S_{t+1} &\sim \mathcal P_{\eta}(S_{t+1} | S_t, A_t) \\
R_{t+1} &= \mathcal R_{\eta}(R_{t+1} | S_t, A_t)
\end{aligned}
$$

Then we apply model-free RL to the samples, e.g. Monte-Carlo control, Sarsa, Q-learning, etc.

Sample-based planning methods are often more efficient because, by sampling, we consider experiences that are more likely to happen.

This is basically the agent learning from the "imagined" or "simulated" world its learned from experience.

It then acts based on its model-free learning and generates more experiences which further refine the model, which it then samples to learn a better value function or policy, which it then acts from, and so on.

This can be extended to incorporate uncertainties with Bayesian model-based reinforcement learning (not covered here).

The performance of model-based RL is limited to the optimal policy for the approximate MDP; i.e. it's only as good as the estimated model. So we have to consider the case where the model is imperfect/inaccurate, i.e. $(\mathcal P_{\eta}, \mathcal R_{\eta}) \neq (\mathcal P, \mathcal R)$, where a suboptimal policy will result.

There are two solutions:

- when the model is wrong, use model-free RL
- or reason explicitly about the model's uncertainty (e.g. Bayesian methods)

### Integrated architectures

So we can generate experience in two ways:

- real experience, i.e. sampled from the environment (the true MDP)
- simulated experience, i.e. sampled from the model (the approximate MDP)

Model-free RL has no model and _learns_ a value function (and/or policy) from _real_ experience.

Model-based RL (here, using sample-based planning), learns a model from real experience and _plans_ a value function (and/or) policy from _simulated_ experience.

We can combine model-free RL and model-based RL with the _Dyna_ architecture to make the most of both real and simulated experience:

- learn a model from real experience
- _learn and plan_ value function (and/or policy) from _real and simulated_ experience

The canonical Dyna algorithm is the _Dyna-Q_ algorithm:

- initialize $Q(s,a)$ and $\text{Model}(s,a)$ for all $s \in \mathcal S$ and $a \in \mathcal A(s)$
- do forever:
    - $S \leftarrow$ current (nonterminal) state
    - $A \leftarrow \epsilon\text{-greedy}(S,Q)$
    - execute action $A$, observe resulting reward $R$ and state $S'$
    - $Q(S,A) \leftarrow Q(S,A) + \alpha[R + \gamma \max_a Q(S', a) - Q(S,A)]$
    - $\text{Model}(S,A) \leftarrow R, S'$ (assuming deterministic environment)
    - repeat $n$ times:
        - $S \leftarrow$ random previously observed state
        - $A \leftarrow$ random action previously taken in $S$
        - $R,S' \leftarrow \text{Model}(S,A)$
        - $Q(S,A) \leftarrow Q(S,A) + \alpha[R + \gamma \max_a Q(S', a) - Q(S,A)]$

If you train a Dyna-Q agent in one environment and then change the environment on it, its model suddenly becomes inaccurate, but, if exploration is encouraged (there will be more on this later; but one simple variation is Dyna-Q+ in which a little bonus is assigned to unexplored states) then it can reorient itself fairly quickly.

### Simulation-based search

This part focuses on the planning part; i.e. expanding on sample-based planning.

The key ideas are sampling and forward search.

Forward search algorithms select the best action by lookahead. They build a search tree with the current state $s_t$ at the root and then use the model of the MDP to look ahead. So we don't need to solve the whole MDP, just the sub-MDP starting from the current state.

_Simulation-based search_ is a forward search paradigm that uses sample-based planning. We simulate episodes of experience from the current state with the model, i.e.:

$$
\{s_t^k, A_t^k, R_{t+1}^k, \dots, S_T^k\}_{k=1}^K \sim \mathcal M_v
$$

then apply model-free RL to the simulated episodes, e.g.:

- if we use Monte-Carlo control, we have _Monte-Carlo search_.
- if we use Sarsa, we have _TD search_.

#### Simple Monte-Carlo Search

- given a model $\mathcal M_v$ and a _simulation policy_ $\pi$
- for each action $a \in \mathcal A$
    - simulate $K$ episodes from current (real) state $s_t$:

$$
\{s_t, A_t, R_{t+1}^k, S_{t+1}^k, A_{t+1}^k ,\dots, S_T^k\}_{k=1}^K \sim \mathcal M_v, \pi
$$

- evaluate actions by mean return (_Monte-Carlo evaluation_):

$$
Q(s_t, a) = \frac{1}{K} \sum_{k=1}^K G_t \to^P q_{\pi} (s_t, a)
$$

- select current (real) action with maximum value:

$$
a_t = \argmax_{a \in \mathcal A} Q(s_t, a)
$$

(this approach is "simple" because we aren't improving the simulation policy $\pi$ as we go)

#### Monte-Carlo Tree Search

At time of writing, this is a state-of-the-art search method.

- given a model $\mathcal M_v$
- simulate $K$ episodes from current state $s_t$ using current simulation policy $\pi$

$$
\{s_t, A_t, R_{t+1}^k, S_{t+1}^k, A_{t+1}^k ,\dots, S_T^k\}_{k=1}^K \sim \mathcal M_v, \pi
$$

- build a search tree containing visited states and actions
- evaluate states $Q(s,a)$ by mean return of episodes from $s,a$ (Monte Carlo evaluation):

$$
Q(s,a) = \frac{1}{N(s,a)} \sum_{k=1}^K \sum_{u=t}^T \mathbb 1 (S_u, A_u = s,a) G_u \to^P q_{\pi}(s,a)
$$

(where $N(s,a)$ is the number of times the pair $s,a$ was encountered)

- after search is finished, select current (real) action with maximum value in the search tree:

$$
a_t = \argmax_{a \in \mathcal A} Q(s_t,a)
$$

So now instead of just evaluating the root actions, as we did with simple Monte-Carlo search, we're evaluating _every_ state-action pair we visit.

With MCTS, the simulation policy $\pi$ improves (again, we did not see this with simple Monte-Carlo search).

Each simulation consists of two phases: _in-tree_ and _out-of-tree_:

- _tree policy_ (improves): pick actions to maximize $Q(S,A)$. We can do this while we're "within" the tree
- _default policy_ (fixed): pick actions randomly. We use this once we've gone "beyond" the tree (where we don't have any information)

Repeat for each simulation:

- evaluate states $Q(S,A)$ by Monte Carlo evaluation
- improve tree policy, e.g. by $\epsilon\text{-greedy}(Q)$

This is basically Monte-Carlo control applied to simulated episodes of experience, starting from the current (i.e. root) state.

This converges on the optimal search tree, $Q(S,A) \to q_*(S,A)$

For example: in a game of Go, white vs black, the agent is playing as black.

Starting at the current state, it rolls out (sample) one episode to a terminal state and see that it wins ([images from here](http://www0.cs.ucl.ac.uk/staff/d.silver/web/Teaching_files/dyna.pdf)):

![](assets/mcts_01.png)

So according to our tree policy, we select the next state in that branch (since it's the highest valued at the moment), and roll out another episode starting from there. We get to a terminal state and see that it loses:

![](assets/mcts_02.png)

So from our tree policy we pick another state and roll an episode out from there; at the terminal state it wins:

![](assets/mcts_03.png)

So then our tree policy directs us to the next state in that branch and we roll out an episode from there; at the terminal state it loses:

![](assets/mcts_04.png)

So then from our tree policy we pick another state and roll out an episode from there; at the terminal state it wins:

![](assets/mcts_05.png)

and so on.

(note that when I say our "from our tree policy we pick another state and roll an episode out from there", technically we are always restarting from the root, i.e. rolling the episode out from the root, but our tree policy directs us to choose actions in such a way that we end up on what is the most promising node)

Advantages of MCTS:

- highly selective best-first search: we search through the space is by what is most promising, so we are quite efficient.
- evaluates states dynamically (unlike e.g. DP): we don't have to worry about the entire state space, only the states that are relevant to us now
- uses sampling to break curse of dimensionality: we don't have to consider everything that could happen in the environment
- works for "black-box" models: we only need samples
- computationally efficient, anytime, parallelizable

### Temporal-Difference search

MCTS is just one example of a family of very effective search algorithms. The key ideas are to use forward search and sampling.

For example, we can use TD instead of MC, i.e. we can use bootstrapping. So whereas MC tree search applies MC control to the sub-MDP from the current state, TD search applies Sarsa to the sub-MDP from the current state.

For simulation-based search, as we saw in model-free reinforcement learning, bootstrapping is helpful:

- TD search reduces variance but increases bias
- TD search is usually more efficient than MC search
- $TD(\lambda)$ search can be much more efficient than MC search

TD search:

- simulate episodes from current (real) state $s_t$
- estimate action-value function $Q(s,a)$
- for each step of simulation, update action-values by Sarsa:

$$
\Delta Q(S,A) = \alpha (R + \gamma Q(S', A') - Q(S,A))
$$

- select actions based on action-values $Q(s,a)$, e.g. $\epsilon$-greedy
- you can also use function approximation for $Q$

This is an especially nice approach for domains where there are many ways to arrive at a given state. With bootstrapping, you'll already have some knowledge about a state if you re-visit it through another trajectory.

### Dyna-2

_Dyna-2_ incorporates Dyna (using both real and simulated experience to learn and plan our value function and/or policy) with forward search. The agent stores two set of feature weights (i.e. two value functions):

- long-term memory
- short-term (working) memory

Long-term memory is updated from real experience using TD learning (this can be thought of as general domain knowledge that applies to any episode).

Short-term memory is updated from simulated experience using TD search (this can be thought of as specific local knowledge about the current situation).

The value function is the sum of long and short-term memories.


## Exploration and Exploitation

How does a RL agent balance exploitation and exploration? That is, how do we balance making the best decision given what we know (exploitation) and gathering more information (exploration)?

The best long-term strategy may involve sacrificing some short-term gains by exploring.

Thus far we have just used the $\epsilon$-greedy method, but there are better ways.

Broadly, there are three approaches:

- random exploration
    - explore random actions (e.g. $\epsilon$-greedy, softmax)
- optimism in the face of uncertainty
    - estimate uncertainty on value
    - prefer to explore states/actions with highest uncertainty
- information state space
    - consider agent's information as part of its state
    - lookahead to see how information helps reward
    - this is the most "correct" way but also the most computationally difficult

We can explore two different spaces:

- state-action exploration
    - systematically explore state-space/action-space
    - e.g. pick different action $A$ each time $S$ is visited
- parameter exploration
    - parametrize policy $\pi(A|S,u)$
    - e.g. pick different parameters and try for awhile
    - advantage: consistent exploration
    - disadvantage: doesn't know about state/action space

We'll focus on state-action exploration.

### Multi-armed bandits

A very simplified version of the reinforcement learning problem is the _multi-armed bandit_ problem, which is useful for introducing concepts around exploration vs exploitation.

A multi-armed bandit is a tuple $(\mathcal A, \mathcal R)$ (we don't have any states anymore).

$\mathcal A$ is a known set of actions (also called "arms").

$\mathcal R^a(r) = \mathbb P[R = r|A=a]$ is an unknown probability distribution over rewards.

At each step $t$ the agent selects an action $A_t \in \mathcal A$. The environment then generates a reward $R_t \sim R^{A_t}$. The agent's goal is to maximize the cumulative reward $\sum_{\tau=1}^t R_{\tau}$.

The _action-value_ is the mean reward for an action $a$:

$$
q(a) = \mathbb E[R|A=a]
$$

The _optimal value_ $v_*$ is:

$$
v_* = q(a^*) = \max_{a \in \mathcal A} q(a)
$$

The _regret_ is the opportunity loss for one step (i.e. the difference between the best we could have gotten and what we actually did get):

$$
l_t \ mathbb E[v_* - q(A_t)]
$$

The _total regret_ is the total opportunity loss:

$$
L_t = \mathbb E[ \sum_{\tau=1}^t v_* - q(A_{\tau})]
$$

So maximizing cumulative reward is equivalent to minimizing total regret.

The _count_ $N_t(a)$ is the expected number of selections for action $a$.

The _gap_ $\Delta_a$ is the difference in value between action $a$ and optimal action $a^*$, i.e. $\Delta_a = v_* - q(a)$.

Regret is a function of gaps and the counts:

$$
\begin{aligned}
L_t &= \mathbb E [\sum_{\tau=1}^t v_* - q(A_{\tau})] \\
&= \sum_{a \in \mathcal A} \mathbb E [N_t(a)] (v_* - q(a)) \\
&= \sum_{a \in \mathcal A} \mathbb E [N_t(a)] \Delta_a
\end{aligned}
$$

So basically, we want to have small counts of large gaps (i.e. we want to take few actions that are much worse than the optimal action).

However, we don't know what these gaps are.

![](assets/total_regret.png)

With strategies we've seen so far, e.g. greedy and $\epsilon$-greedy, total regret increases linearly - with the greedy strategy, where we _never_ explore, we may just get stuck with a suboptimal action, so total regret keeps accumulating; similarly with $\epsilon$-greedy, where we explore _forever_, we will always sometimes randomly choose an action which is most likely to be suboptimal (not much of a chance we randomly pick the optimal action). Ideally, we want to have a sublinear total regret (one that eventually stops increasing).

#### Random exploration

First let's go more into the greedy algorithm:

- we consider algorithms that estimate $Q_t(a) \approx q(a)$.
- we estimate the value of each action by Monte-Carlo evaluation, i.e.:

$$
Q_t(a) = \frac{1}{N_t(a)} \sum_{t=1}^T \mathbb 1 (A_t = a) R_t
$$

- the _greedy_ algorithm selects the action with the highest value, i.e.:

$$
A_t = \argmax_{a \in \mathcal A} Q_t(a)
$$

As mentioned before, this can lock onto a suboptimal action forever, so it has linear total regret.

An enhancement on the greedy algorithm is the greedy algorithm with _optimistic initialization_, in which we initialize all values to the maximum possible, i.e. $Q(a) = r_{\text{max}}$; that is we assume all actions are really good (over time these assumptions get attenuated with the actual observations using Monte-Carlo evaluation), then we act greedily, i.e. $A_t = \argmax_{a \in \mathcal A} Q_t(a)$.

Note that you can also initialize the counts $N_t(a)$ in addition to the values - this is how we can assert our confidence in these optimistic initializations (e.g. if we set $N_t(a)=1$ then we aren't very confident, but if we set it to $N_t(a)=1000$ then we are much more confident).

This optimistic initialization encourages exploration of unknown values, which is great. But a few unlucky samples can also cause the algorithm to lock onto a suboptimal action (though in practice, this method can work very well). So this has linear total regret as well.

Note that by assigning more confidence to the optimistic prior, you can withstand more unlucky samples and the algorithm will perform better.

Now let's consider the $\epsilon$-greedy algorithm. As mentioned before, it has linear total regret. But we can easily make this sublinear by decaying $\epsilon$. For example, consider the following schedule:

$$
\begin{aligned}
c &> 0 \\
d &= \min_{a| \Delta_a > 0} \Delta_a \\
\epsilon_t &= \min \{1, \frac{c|\mathcal A|}{d^2t}\}
\end{aligned}
$$

This has logarithmic asymptotic total regret.

In practice, that schedule is impossible because it requires knowledge of the gaps (i.e. of $v_*$), which we don't have.

So ideally we want to find an algorithm with sublinear regret for any multi-armed bandit which does not require knowledge of $\mathcal R$.

There is a lower bound to total regret (i.e. no algorithm can possibly do better than this lower bound), so we want to get as close to that lower bound as possible.

The difficulty of a multi-armed bandit problem is in large part determined by the similarity between the optimal arm/action and the other arms/actions. That is, if suboptimal arms have a reward distribution very similar to the optimal one, it will take a long time to discern the optimal one (contrast to an easier situation where one arm is "obviously" optimal in that it consistently gives higher reward than all the other arms).

We can describe this formally with the gap $\Delta_a$ and the similarity in distributions $KL(\mathcal R^a || \mathcal R^{a^*})$.

Theorem (Lai and Robbins):

Asymptotic total regret is at least logarithmic in number of steps:

$$
\lim_{t \to \infty} L_t \geq \log t \sum_{a| \Delta_a > 0} \frac{\Delta_a}{KL(\mathcal R^a || \mathcal R^{a^*})}.
$$

Intuitively, the larger the gap $\Delta_a$, the larger regret will occur, and is inversely proportional to the difference (KL divergence) in distributions (the less different they are, the more regret).

#### Optimism in the face of uncertainty

The principle of _optimism in the face of uncertainty_ states that we should not the action that is certain to be best, but the action that has the potential to be best.

Consider the following example with three arms/actions:

![](assets/optimism.png)

We are most certain about $a_3$ and least certain about $a_1$. The principle of optimism in the face of uncertainty dictates that we should try $a_1$ even though our current mean for it is less than $a_3$ since it has the potential to yield higher rewards (i.e. it's possible that its mean is actually higher than $a_3$'s, we just don't know enough to be certain yet; this will be formalized in a bit). If the mean of $a_1$ is actually lower then $a_3$, eventually we'll realize this as we start to be more and more certain about its value through more experience. Then we'd switch back to $a_3$.

More formally (for "potential to yield higher rewards"), we estimate an upper confidence $U_t(a)$ for each action value such that $q(a) \leq Q_t(a) + U_t(a)$ with high probability. This depends on the number of times $N(a)$ has been selected:

- small $N_t(a) \to$ large $U_t(a)$ (estimated value is uncertain)
- large $N_t(a) \to$ small $U_t(a)$ (estimated value is accurate)

Then we select the action maximizing the _upper confidence bound_ (UCB):

$$
A_t = \argmax_{a \ in \mathcal A} Q_t(a) + U_t(a)
$$

In statistics, there is a theorem known as _Hoeffding's Inequality_:

Let $X_1, \dots, X_t$ be iid random variables in $[0,1]$, and let $\bar X_t = \frac{1}{\tau} \sum_{\tau=1}^t X_{\tau}$ be the sample (empirical) mean. Then:

$$
\mathbb P[\mathbb E[X] > \bar X_t + u] \leq e^{-2tu^2}
$$

i.e. the probability that we're wrong in our estimate of the empirical mean by the amount $u$ is less than or equal to $e^{-2tu^2}$; this is true for _any_ distribution.

We can apply Hoeffding's Inequality to the rewards of the bandit conditioned on selecting action $a$, i.e.:

$$
\mathbb P [q(a) > Q_t(a) + U_t(a)] \leq e^{-2N_t(a)U_t(a)^2}
$$

So we can pick a probability $p$ that the true value exceeds the UCB and then solve for $U_t(a)$:

$$
\begin{aligned}
e^{-2N_t(a)U_t(a)^2} &= p \\
U_t(a) &= \sqrt{\frac{-\log p}{2 N_t(a)}}
\end{aligned}
$$

For example, we can set $p=0.05$ to get the 95% UCB.

We can reduce $p$ as we observe reward, e.g. $p = t^{-4}$:

$$
U_t(a) = \sqrt{{2 \log t}{N_t(a)}}
$$

This ensures we select the optimal action as $t \to \infty$.

This leads to the _UCB1 algorithm_:

$$
A_t = \argmax_{a \ in \mathcal A} Q_t(a) + \sqrt{{2 \log t}{N_t(a)}}
$$

This achieves a logarithmic asymptotic total regret.

This was the frequentist approach to multi-armed bandits; we made no assumptions about the distributions of the bandits.

##### Bayesian bandits

Alternatively, we can use a Bayesian approach (_Bayesian bandits_) where we exploit prior knowledge about rewards, $p[\mathcal R^a]$.

Consider a distribution $p[Q | w]$ over an action-value function with parameter $w$, e.g. we could assume the $Q$ functions are independent Gaussians: $w = [\mu_1, \sigma_1^2, \dots, \mu_k, \sigma_k^2]$ for $a \in [1,k]$.

Bayesian methods compute a posterior distribution over $w$, $p[w|R_1, \dots, R_t]$. Then we use this posterior to guide exploration, e.g. through upper confidence bounds or probability matching. If the prior knowledge is accurate, we get better performance.

So basically we describe $Q$ as a distribution which we parameterize with $w$.

In more detail, the upper confidence bound method:

- we use Bayes law to compute posterior $p[w | R_1, \dots, R_{t-1}]$
- then we compute the posterior distribution over action-values $p[Q(a) | R_1, \dots, R_{t-1}] = p[Q(a)|w] p[w|R_1, \dots, R_{t-1}]$
- then we estimate an upper confidence from the posterior, e.g. $U_t(a) = c \sigma(a)$, where $\sigma(a)$ is a standard deviation of $p(Q(a)|w)$, i.e. the upper confidence bound could be $c$ standard deviations from the mean
- then we pick the action that maximizes $Q_t(a) + c \sigma(a)$

So it's quite similar to UCB1, we just incorporate priors.

Alternative to the UCB method, we can use _probability matching_, where we select an action $a$ according to the probability that $a$ is the optimal action:

$$
\pi(a) = \mathbb p[Q(a) = \max_{a'} Q(a') | R_1, \dots, R_{t-1}]
$$

Probability matching is optimistic in the face of uncertainty (uncertain action shave a higher probability of being max). But it can be difficult to compute $\pi(a)$ analytically from the posterior.

To do this, we can use _Thompson sampling_, which is sample-based probability matching:

$$
\pi(a) = \mathbb E[\mathbb 1(Q(a) = \max_{a'} Q(a')) | R_1, \dots, R_{t-1}]
$$

We use Bayes law to compute the posterior distribution $p_q(Q|R_1, \dots, R_{t-1})$. Then we sample an action-value, i.e. a single value for $Q(a)$, from the posterior. Then we select the action associated with the maximum sample, $A_t = \argmax_{a \in \mathcal A} Q(a)$.

For Bernoulli bandits, this achieves the Lai and Robbins lower bound on regret (i.e. it is optimal).

Note that "optimism in the face of uncertainty" won't work where the action-space is infinite (it will forever explore and never end up exploiting anything) or when it is expensive to explore (this isn't very "safe" exploration, e.g. if you have an expensive robot, you don't want it trying _everything_).

#### Information state space

With exploration we gain information. If we could quantify the value of that information, we could compute the exploration-exploitation trade off perfectly (i.e. the long-term reward after getting information minus the immediate reward of exploitation).

Information gain is higher in uncertain situations - for instance, if you already know everything about a situation, there's no information to be gained. So we want to explore uncertain situations more in the optimal way.

We can introduce an _information state_ $\tilde {\mathcal S}$ summarizing all information accumulated so far.

Each action $A$ causes a transition to a new information state $\tilde {\mathcal S}'$ (by adding information) with probability $\tilde {\mathcal P}_{\tilde {\mathcal S}, \tilde {\mathcal S}'}^A$.

So now we have an MDP $\tilde {\mathcal M}$ in the augmented information state space:

$$
\tilde {\mathcal M} = (\tilde {\mathcal S}, \mathcal A, \tilde {\mathcal P}, \mathcal R, \gamma)
$$

For example, in a Bernoulli bandit (i.e. where there are only two outcomes, 0 or 1), we may have an information state $\tilde {\mathcal S} = (\alpha, \beta)$ where $\alpha_a$ counts the pulls of arm $a$ where the reward was 0 and $\beta_a$ counts the pulls of arm $a$ where the reward as 1.

Now we have an infinite MDP, since there are infinitely many information states. We can solve this MDP with reinforcement learning, e.g. model-free reinforcement learning or Bayesian model-based reinforcement learning. The latter approach is known as _Bayes-adaptive_ RL, where we characterize our information state as a posterior distribution.

For example, with Bernoulli bandits:

- start with a $\text{Beta}(\alpha_a, \beta_a)$ prior over reward function $\mathcal R^a$
- each time action $a$ is selected, update posterior for $\mathcal R^a$:
    - $\text{Beta}(\alpha_a + 1, \beta_a)$ if reward is 0
    - $\text{Beta}(\alpha_a, \beta_a + 1)$ if reward is 1
- this defines transition function $\tilde {\mathcal P}$ for the Bayes-adaptive MDP
- each information state $(\alpha, \beta)$ corresponds to a model $\text{Beta}(\alpha, \beta)$
- each state transition corresponds to a Bayesian model update
- solving the Bayes-adaptive MDP takes account of the value of information because the effect of new information is factored into the model update

The Bernoulli case of the Bayes-adaptive MDP can be solved by dynamic programming; the solution is known as the _Gittins index_.

Exactly solving a Bayes-adaptive MDP is typically intractable, but methods like large-scale (large state space) planning methods (e.g. Monte-Carlo tree search) can be applied.

### Contextual bandits

So far we've introduced state as only information state but we can bring in our previous notions of state as well.

A _contextual bandit_ is a tuple $(\mathcal A, \mathcal S, \mathcal R)$.

This is just like multi-arm bandits except that we've reintroduced states $\mathcal S$. $\mathcal S = \mathbb P[S]$ is an unknown distribution over states (also called "_contexts_").

$\mathcal R_s^a(r) = \mathbb P[R=r |S=s, A=a]$ is an unknown probability distribution over rewards.

At each step $t$:

- environment generates state $S_t \sim \mathcal S$
- agent selects action $A_t \in \mathcal A$
- environment generates reward $R_t \sim \mathcal R_{S_t}^{A_t}$

The goal is to maximize cumulative reward $\sum_{\tau=1}^t R_{\tau}$.

(not covered in lecture)


### Extending to MDPs

The UCB approach can be generalized to full MDPs to give a model-free RL algorithm, i.e. we can maximize the upper confidence bound on the action-value function:

$$
A_t = \argmax_{a \in \mathcal A} A(S_t, a) + U(S_t, a)
$$

One thing worth noting is that this ignores that, in MDPs, the policy will likely improve (if we're doing control with the MDP) and the $Q$ values will get better. So the uncertainty should take into account that the $Q$ value might be wrong not only because of the uncertainty in the estimate (e.g. because the policy evaluation isn't good yet) but also because the policy may still have a lot of improving to do.

You can also extend the general "optimism in the face of uncertainty" approach by replacing the reward for unknown or poorly estimated states with $r_max$, then just solve the MDP with some planning algorithm (e.g. policy iteration, value iteration, tree search, etc). An example of this is the _Rmax algorithm_.

The information state space approach can also be extended to MDPs. Here, the augmented state is now $\tilde S = (S,I)$ where $S$ is the original state within the MDP and $I$ is the accumulated information. The Bayes-adaptive approach maintains a posterior model corresponding to each augmented state, $\mathbb P[\mathcal P, \mathcal R|I]$. Solving the Bayes-adaptive MDP finds the optimal exploration/exploitation trade-off with respect to prior, so this is the "correct" approach. Like before, the augmented MDP is typically enormous, but Monte-Carlo tree search has proven effective here.


