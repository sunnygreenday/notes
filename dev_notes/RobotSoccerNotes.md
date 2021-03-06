...menustart

 - [MAXQ-OP](#3671bc7cbc9bd1056af8587cbea6fb26)
 - [1. Introduce](#36b4cc2ccb642e8a023f180968cd5d48)
 - [2. RELATED WORK](#d293cec1c68216b097f0e838a7e8ffe8)
 - [3. Background](#ce299cb2c59ef259932ce124c943dd1f)
	 - [3.1 MDP](#ec611cd465394d15a6bffb6e3ffa4b05)
	 - [3.2. MAXQ Hierarchical Decomposition](#dc0487d5842259239fe7dbbf7ddc0f37)
 - [4. ONLINE PLANNING WITH MAXQ](#c7d5283deeab21f16b4eb0d5b8392c90)

...menuend


<h2 id="3671bc7cbc9bd1056af8587cbea6fb26"></h2>

# MAXQ-OP

 - a novel online planning algorithm for large MDPs
    - that utilizes MAXQ hierarchical decomposition in online settings.
 - able to reach much more deeper states in the search tree with relatively less computation time
    - by exploiting MAXQ hierarchical decomposition online

<h2 id="36b4cc2ccb642e8a023f180968cd5d48"></h2>

# 1. Introduce 

 - offline MDP not works for soccer 
    - state space is too large
        - even ignoring some less important state variables  
            - e.g., stamina and view width for each player
        - the ball state takes four variables x, y, ẋ, ẏ
        - each player state takes six variables, (x, y, ẋ, ẏ, α, β), 
            - where (x, y), (ẋ, ẏ), α, and β are position, velocity, body angle, and neck angle, respectively. 
    - the *dimensionality* of the resulting state space is 22 × 6 + 4 = 136
    - All state variables are continuous
    - we obtain a state space containing 10⁴⁰⁸ states, if we discretize each state variable into only 10³ values
    - even worse, the transition model of the problem is subject to change given different opponent teams. 
 - online algorithms alleviate this difficulty
    - by focusing on computing a near-optimal action merely for the current state. 
    - The key observation is that an agent can only encounter a fraction of the overall states when interacting with the environment
        - eg. the total number of timesteps for a match is normally 6000
        - the agent has to make decisions only for those encountered states
        - Online algorithms evaluate all available actions for the current state and select the seemingly best one by recursively performing forward search over reachable state space.
            - 值得指出的是，在搜索过程中采用启发式技术以减少时间和内存使用并不常见，就像依赖前向搜索的许多算法一样，如实时动态规划和UCT
    - online algorithms can easily handle unpredictable changes of system dynamics
        - because in online settings, we only need to tune the decision making for a single timestep instead of the entire state space
    - However, the agent must come up with a plan for the current state in almost real time 
        - because computation time is usually very limited for online decision making 
        - (e.g., only 100ms in RoboCup 2D). 
 - Hierarchical decomposition scale MDP algorithms to large problems
    - decomposes the value function of the original MDP into an additive combination of value functions for sub-MDPs arranged over a task hierarchy
    - MAXQ advantages
        - temporal abstraction
            - temporally extended actions 时间上延伸的动作 (also known as options, skill, or macroactions) are treated as primitive actions by higher-level subtasks. 
        - state abstraction
            - aggregates the underlying system states into macrostates by eliminating irrelevant state variables for subtasks. 
            - 通过消除子任务的不相关状态变量将基础系统状态聚合成宏状态。
        - subtask sharing
            - 子任务计算策略 重用
        - For example, in RoboCup 2D, attacking behaviors generally include passing, dribbling, shooting, intercepting, and positioning. Passing, dribbling, and shooting share the same kicking skill, whereas intercepting and positioning utilize the identical moving skill. 
 - MAXQ-OP MAXQ value function decomposition for online planning
    - performs online planning to find the near-optimal action for the current state
    - while exploiting the hierarchical structure of the underlying problem. 

---

 - online algorithms find a near-optimal action for current state via forward search incrementally in depth
 - However, it is difficult to reach deeper nodes in the search tree within domains with large action space , while keeping the appropriate branching factor to a manageable size.  
    - 比如： 球员们需要很长时间才能收获一个进球
 - Hierarchical decomposition enables the search process to reach deeper states using temporally abstracted subtasks -- a sequence of actions that lasts for multiple timesteps
    - For example, when given a subtask called *moving-to-target* , the agent can continue the search process starting from the target state ,without considering the detailed plan on specifically moving toward the target , just assuming that *moving-to-target* can take care of this.
 - One advantage of MAXQ-OP 
    -  do not need to manually write down complete local policy for each subtask 
    - Instead, build a MAXQ task hierarchy by defining well the active states, the goal states, and optionally the local-reward functions for all subtasks.
    - Local-reward functions are artificially introduced by the programmer to enable more efficient search processes,as the original rewards defined by the problem
    may be too sparse to be exploited.
 - Given the task hierarchy, MAXQ-OP automatically finds the near-optimal action for the current state by simultaneously searching over the task hierarchy and building a forward search tree.
 - a *completion function* for a task gives the expected cumulative reward obtained , after finishing a subtask but before completing the task itself following a hierarchical policy.
 - Directly applying MAXQ to online planning requires knowing in advance the completion function for each task, following the recursively optimal policy.
 - Thus, obtaining the completion function is equivalent to solving the entire task which is not applicable in online settings.
    - This poses the major challenge of utilizing MAXQ online.

--- 

 - 2 major work
    - exploit the MAXQ hierarchical structure online
    - approximation method made for computing the completion function online.

---

<h2 id="d293cec1c68216b097f0e838a7e8ffe8"></h2>

# 2. RELATED WORK

 - online planning for MDPs
    - RTDP : real-time dynamic programming
        - tries to find a near-optimal action for the current state by conducting a trial-based search process
        - with greedy action selection and an admissible heuristic
    - AO\*
        - 与或图的启发式搜索算法
    - Monte Carlo tree search (MCTS)
        - combining tree search methods with Monte Carlo sampling techniques
    -  trial-based heuristic tree search (THTS)
 - they do not exploit the underlying hierarchical structure of the problem

---

 -  hierarchical reinforcement learning (HRL)
    - HRL aims to learn a policy for an MDP efficiently by exploiting the underlying structure while interacting with the environment.
    - One common approach is using state abstraction to partition the state space into a set of subspaces, namely ***macrostates***, by eliminating irrelevant state variables
    - In particular, Sutton model HRL as a semi-Markov decision process (SMDP) by introducing temporally extended actions, namely ***options***
        - Each option is associated with an inner policy that can be either manually specified or learned by the agent
    - MAXQ-based HRL methods convert the original MDP into a hierarchy of SMDPs and learn the solutions simultaneously 
 - challenge of hierarchical decomposition
    - when searching with high-level actions (tasks), it is critical to know how they can be fulfilled by low-level actions (subtasks or primitive actions).
        - For example, if a player wants to shoot the goal (high-level action), it must first know how to adjust its position and kick the ball toward a specified position (low-level actions).  
        - Unfortunately, this information is not available in advance during online planning
        - we address this challenge by introducing a termination distribution for each subtask over its terminal states and assuming that subtasks will take care of the local policies to achieve the termination distributions. 


<h2 id="ce299cb2c59ef259932ce124c943dd1f"></h2>

# 3. Background

<h2 id="ec611cd465394d15a6bffb6e3ffa4b05"></h2>

## 3.1 MDP

 - we assume that there exists a set of goal states G ⊆ S
    - for all g ∈ G and a ∈ A, we have T(g | g, a) = 1 and R(g, a) = 0
 - If the discount factor γ = 1 , the resulting MDP is then called *undiscounted goal-directed* MDP
 - It has been proved that any MDP can be transformed into an equivalent undiscounted negative-reward goal-directed MDP where the reward for nongoal states is strictly negative

<h2 id="dc0487d5842259239fe7dbbf7ddc0f37"></h2>

## 3.2. MAXQ Hierarchical Decomposition

 - decomposes the original MDP M into a set of sub-MDPs arranged over a hierarchical structure
 - Each sub-MDP is treated as an macroaction for high-level MDPs
 - Specifically, let the decomposed MDPs be {M₀, M₁,..., Mn}, then M₀ is the root subtask such that solving M₀ solves the original MDP M
 - Each subtask Mᵢ is defined as a tuple < τᵢ, Aᵢ, R̃ᵢ >
    - τᵢ: *termination predicate*  
        - partitions the state space into a set of active states Sᵢ, and a set of terminal states Gᵢ ( also known as subgoals )
    - Aᵢ: is a set of (macro)actions that can be selected by Mᵢ 
        - which can either be ***primitive actions*** of the original MDP M or ***low-level subtasks***  .
    - R̃ᵢ: (optional) *local-reward* function
        - specifies the rewards for transitions from active states Sᵢ to terminal states Gᵢ
 - A subtask can also take parameters
    -  different bindings of parameters specify different instances of a subtask
 - Primitive actions are treated as primitive subtasks 
    - such that they are always executable and will terminate immediately after execution.
 - This hierarchical structure can be represented as a directed acyclic graph -- the *task graph*
    - ![](https://raw.githubusercontent.com/mebusy/notes/master/imgs/robotsoccer_task_graph.png)
    - a subtask in the task graph is also a (macro)action of its parent
    - root task M₀ has three macroactions: M₁, M₂, and M₃ ( i.e., A₀ = {M₁, M₂, M₃}).
    - Subtasks M₁, M₂, and M₃ , are sharing lower-level primitive actions Mᵢ (4 ≤ i ≤ 8) as their subtasks
    - Each subtask must be fulfilled by a policy, unless it is a primitive action
 - Given the hierarchical structure, a *hierarchical policy π* is defined as a set of policies for each subtask π = { π₀, π₁, ... π<sub>n</sub>} ,
    - πᵢ for subtask Mᵢ , is a mapping from its active states to actions πᵢ: Sᵢ → Aᵢ
 - The *projected value function* V<sup>π</sup>(i,s) is defined as the ***expected cumulative reward***
    - follow a hierarchical policy π = { π₀, π₁, ... π<sub>n</sub>}
    - starting from state s
    - until Mᵢ terminates at one of its terminal states g ∈ Gᵢ.
    - 状态s 开始，到 Mᵢ 执行到一个中止状态  ；如果 Mᵢ 是primitive subtasks，则 V就是 R(s,a) , 否则就是 固定 π(s)的Q<sup>π</sup>  ;
 - the action value function Q<sup>π</sup> (i, s, a) for subtask Mᵢ is defined as the ***expected cumulative reward*** of
    - first performing action Mₐ (which is also a subtask) in state *s*
    - then following policy π until the termination of Mᵢ
    - 状态s，首先执行 Ma，直到 子任务 Mᵢ 终止
    - 和 V<sup>π</sup> 相比，Q<sup>π</sup> 必须从 动作 Mₐ 开始
    - V / Q 的意义和 普通的RL中的都不一样， 因为任何一个 subtask Mᵢ 的结束都不一定 导致 状态 transform, 所以不能使用 普通RL的 value iteration.

 - It has been shown that the value functions of a hierarchical policy π can be expressed recursively as follows 
    - Q<sup>π</sup> (i, s, a) = V<sup>π</sup> (a, s) + C<sup>π</sup> (i, s, a), where
        - V<sup>π</sup> (i, s) = R(s,i), if Mᵢ is primitive  
        - V<sup>π</sup> (i, s) = Q<sup>π</sup> (i, s, π(s))  , otherwise ( 固定  π(s)的Q  )
    - C<sup>π</sup> (i, s, a)  is the *completion function* that specifies the expected cumulative reward
        - obtained after finishing subtask Mₐ
        - but before completing Mᵢ
        - when following the hierarchical policy π, defined as
        - ![](https://raw.githubusercontent.com/mebusy/notes/master/imgs/robotSoccer_c_pi.png)
        - Pr is also *T*
        - Pr(s' , N | s, a) is the probability that subtask Mₐ will terminate in state s after N timesteps of execution. 
 - A *recursively optimal policy* π<sup>\*</sup> can be found by recursively computing the optimal projected value function:
    - Q<sup>\*</sup> (i, s, a) = V<sup>\*</sup> (a, s) + C<sup>\*</sup> (i, s, a)    -- (8)
        - ![](https://raw.githubusercontent.com/mebusy/notes/master/imgs/robotSoccer_Q_star.png)
 - In Equation (8), C<sup>\*</sup> (i, s, a) = C<sup>π\*</sup> (i, s, a) ,  is the completion function of the recursively optimal policy π<sup>\*</sup>
 - Given the optimal value functions, the optimal policy πᵢ<sup>\*</sup> for subtask Mᵢ is then given as
    - ![](https://raw.githubusercontent.com/mebusy/notes/master/imgs/robotSoccer_pi_i_star.png)
 
<h2 id="c7d5283deeab21f16b4eb0d5b8392c90"></h2>

# 4. ONLINE PLANNING WITH MAXQ

## 4.1. Overview of MAXQ-OP

we introduce depth array d and maximal search depth array D

 - d[i] is current search depth in terms of macroactions for subtask Mᵢ
 - D[i] gives the maximal allowed search depth for subtask Mᵢ

A heuristic function H is also introduced to estimate the value function when exceeding the maximal search depth. 

That means we will terminate and return H(i,s)  if d[i] >= D[i]

## 4.2 Main Procedure of MAXQ-OP

 - ALGORITHM 1: OnlinePlanning()
     - Input: an MDP model with its MAXQ hierarchical structure
     - Output: the accumulated reward r after reaching a goal

```
r ← 0;
s ← GetInitState();
root_task ← 0;
depth_array ← [0, 0,..., 0];

// loops over until a goal state in G₀ is reached
while s ∉ G₀ do
    // EvaluateStateInSubtask -- key procedure
    // evaluates each subtask by depth-first search
    // returns the seemingly best action for the current state
    // called with a depth array containing all zeros for all subtasks.
    (v,a_p) ← EvaluateStateInSubtask(root_task, s, depth_array);
    r ← r + ExecuteAction(a_p, s);
    // next state after execute action
    s ← GetNextState(); 
return r
```  

## 4.3. Task Evaluation over Hierarchy

 - ALGORITHM 2: EvaluateStateInSubtask(i, s, d)
    - Input: subtask Mᵢ, state s , depth array d
    - Output: (V<sup>\*</sup>(i, s), a primitive action a<sub>p</sub><sup>\*</sup> ) 

```
// (1) the subtask is a primitive action
if Mᵢ is primitive then return ( R(s, Mᵢ), Mᵢ ) ;
// (2) the state is a goal state or a state 
//     beyond the scope of this subtask’s active states;
else if s ∉ Sᵢ and s ∉ Gᵢ then return (-∞, nil) ;
else if s ∈ Gᵢ then return (0, nil);
// (3) the maximal search depth is reached—that is, d[i] ≥ D[i].
else if d[i] ≥ D[i] then return ( HeuristicValue(i, s), nil ) ;
else
    // recursively evaluates all lower-level subtasks
    // and finds the seemingly best (macro)action
    (v* , a_p*) ← (0, nil) 
    for M_k ∈ Subtasks(Mᵢ) do
        if M_k is primitive or s ∉ G_k then
            (v , a_p) ← EvaluateStateInSubtask(k, s, d);
            v ← v + EvaluateCompletionInSubtask(i, s, k, d);
            if v > v∗ then
                (v* , a_p*) ← (v , a_p) 
    return (v* , a_p*)
```

 - Note that each subtask can have different maximal depths
    - e.g., subtasks in the higher level may have smaller maximal depth in terms of evaluated macroactions

## 4.4 Completion Function Approximation

 - ALGORITHM 3: EvaluateCompletionInSubtask(i, s, a, d)
    - Input: subtask Mᵢ, state s, action M and depth array d







