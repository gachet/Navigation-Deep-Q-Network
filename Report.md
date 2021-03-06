# Report: Navigation using Deep Q-Network

## Learning Algorithm
What we're dealing with here is an envirornment with continuous state space and discrete action space with 4 possible actions. Deep Q-Network is an excellent choise to address this problem.

I'm gonna solve this environment using [vanilla Deep Q-Netwok](http://www.readcube.com/articles/10.1038/nature14236) (with fixed Q-targets) and then we'll try different improvements such as:

- Deep Reinforcement Learning with Double Q-learning. [Paper](https://arxiv.org/abs/1509.06461)
- Dueling Network Architectures for Deep Reinforcement Learning. [Paper](https://arxiv.org/abs/1511.06581)
- Prioritized Experience Replay. [Paper](https://arxiv.org/abs/1511.05952)
- Noisy Networks for Exploration. [Paper](https://arxiv.org/abs/1706.10295)

To finish off, I'll use all of them at once and hopefully we'll see a big performance boost.

### Hyperparameters
_Experience Replay_ is a technique we use to decorrelate transitions observed by the agent and that are stored for later resused and learnt from. It has been shown that this greatly stabilizes and improves the DQN training procedure. These transitions are stored in a buffer, ```buffer_size=1e5``` , and will be sampled in batches, ```batch_size=64```, 

The _discount factor_ is a measure of how far ahead in time the algorithm looks. If we wanted to prioritise rewards in the distant future, we'd keep the value closer to one. On the other hand if we wanted to consider only rewards in the immediate future, then we'd use a discount factor closer to zero. I'll be using ```gamma=0.99```

We use _Fixed Q-targets_ to prevent the "chasing a moving target" effect when using the same parameters (weights) for estimating the target and the Q value. This is because there is a big correlation between the TD target and the parameters we are changing. The solution is using a second network whose weights will be [softly updated](https://github.com/jscriptcoder/Navigation-Deep-Q-Network/blob/master/agent/agent.py#L228), with interpolation parameter ```tau=1e-3```, and that we'll be using to calculate the TD target.

We'll be using Adam optimizer with a learning rate ```lr=5e-4``` for our models. 

How ofter we're gonna learn is dictated by the parameter ```update_every=4```, which means every 4 steps we're gonna learn from previous experiences and update the weights of our networks.

### Algorithms
1. **Vanilla DQN**:
Deep Q-learning algorithm using a simple Experience Replay buffer, Fixed Q-Target improvement and epsilon-greedy exploration method. This is an [excellent article](https://medium.com/@jonathan_hui/rl-dqn-deep-q-network-e207751f7ae4) about this algorithm.

2. **Double Q-Network**:
Double DQNs, or double Learning, was introduced by [Hado van Hasselt](https://papers.nips.cc/paper/3964-double-q-learning). This method handles the problem of the overestimation of Q-values. There is a problem when calculating the TD target: how are we sure that the best action for the next state is the action with the highest Q-value?. As a consequence, at the beginning of the training we don't have enough information about the best action to take. Therefore, taking the maximum Q-value as the best action can lead to false positives (overestimation). There is an [interesting article](https://towardsdatascience.com/double-deep-q-networks-905dd8325412) explaining this solution.

3. **Dueling Q-Network**:
The Q-values correspond to how good it is to be at a state and taking an action at that state ```Q(s,a)```. The idea behind this architecture is to separate the estimator of two elements: ```V(s)```, the value of being at that state and ```A(s,a)```, the advantage of taking that action at that state (how much better is to take this action versus all other possible actions at that state), using two new streams, and then we combine them through a special aggregation layer to get an estimate of ```Q(s,a)```. [Here](https://medium.com/@awjuliani/simple-reinforcement-learning-with-tensorflow-part-4-deep-q-networks-and-beyond-8438a3e2b8df#af34) there is a better explanation of the architecture.

4. **DQN with Prioritized Experience Replay**:
Prioritized Experience Replay (PER) was introduced in 2015 by [Tom Schaul](https://arxiv.org/search/?searchtype=author&query=Schaul%2C+T). In the famous DQN algorithm, a buffer of past experiences is used to stabilize training by decorrelating the training examples in each batch used to update the neural network. But some experiences may be more important than others for our training, and might occur less frequently. Because we sample the batch uniformly (selecting the experiences randomly) these rich experiences that occur rarely have practically no chance to be selected. with PER we try to change the sampling distribution by using a criterion to define the priority of each tuple of experience. You can find more details about experience replay and PER in [this article](https://medium.com/@badamivikas/deeper-intuition-on-prioritized-experience-replay-33fcc55de44e).

5. **DQN using Noisy Networks for exploration**:
One fundamental problem of Reinforcement Learning is Exploration/Exploitation dilemma. This issue raises from the fact that our agent needs to keep the balance between exploring the environment and using what's learned from this exploration. There are different ways to address this problem. Most common one is using epsilon-greedy: with some probability epsilon (given as hyperparameter) your agent takes a random step instead of acting according to policy it has learned. DeepMind proposes very simple and surprisingly efficient way of tackling the above issue. The method basically consists of adding the gaussian noise to the last (fully-connected) layers of the network. You can read more about the technique in this [blog post](https://medium.com/@shmuma/summary-noisy-networks-for-exploration-c8ba6e2759c7).

6. **All of them combined**:
We'll see in a moment what it is to use all this improvements at the same time.

### Neural Networks Architecture
I'm using [ReLU](https://machinelearningmastery.com/rectified-linear-activation-function-for-deep-learning-neural-networks/) activation function in all the NN layers. For Noisy Networks, I'm extending [torch.nn.Linear class](https://pytorch.org/docs/stable/nn.html#linear) as suggested in the [paper](https://arxiv.org/abs/1706.10295). See [NoisyLinear class](agent/model.py#L7)

1. **Vanilla DQN**
<img src="images/vanilla_dqn_agent.png" width="450" title="Vanilla DQN Agent" />

2. **Double DQN**
<img src="images/double_dqn_agent.png" width="450" title="Double DQN Agent" />

3. **Dueling DQN**
<img src="images/dueling_dqn_agent.png" width="450" title="Dueling DQN Agent" />

4. **Prioritized DQN**
<img src="images/per_dqn_agent.png" width="450" title="DQN Agent using PER" />

5. **NoisyNets DQN**
<img src="images/noisy_dqn_agent.png" width="450" title="DQN Agent using NoisyNets" />

6. **All of them**
<img src="images/all_dqn_agent.png" width="450" title="DQN Agent using combined solutions" />

## Plot of Rewards
Following I'll present two plots for each solution:
- First plot shows the rewards during training with a fitted line showing the trend and maximum score achieved by the agent.
- Second plot shows detailed results of the training with information such as moving average score in a rolling window of 100 episodes, at which epoch the agent reached the target score of +13 in the last 100 episodes and the maximum moving average score achieved during training.

1. **Vanilla DQN**:
<img src="images/vanilla_dqn_training.png" width="700" title="Vanilla DQN training" />
<img src="images/vanilla_dqn_performance.png" width="700" title="Vanilla DQN performance" />

2. **Double DQN**:
<img src="images/double_dqn_training.png" width="700" title="Double DQN training" />
<img src="images/double_dqn_performance.png" width="700" title="Double DQN performance" />

3. **Dueling DQN**:
<img src="images/dueling_dqn_training.png" width="700" title="Dueling DQN training" />
<img src="images/dueling_dqn_performance.png" width="700" title="Dueling DQN performance" />


4. **Prioritized DQN**:
<img src="images/per_dqn_training.png" width="700" title="PER DQN training" />
<img src="images/per_dqn_performance.png" width="700" title="PER DQN performance" />

5. **NoisyNets DQN**:
<img src="images/noisy_dqn_training.png" width="700" title="Noisy DQN training" />
<img src="images/noisy_dqn_performance.png" width="700" title="Noisy DQN performance" />

6. **All of them**:
<img src="images/all_dqn_training.png" width="700" title="Combined DQN training" />
<img src="images/all_dqn_performance.png" width="700" title="Combined DQN performance" />

As we can see, most of the improvements achieve better performance than Vanilla DQN, except for Prioritized Experience Replay, which reaches the target later in time. It does though get higher score during training than Vanilla DQN.

We also can see that for this environment, using Dueling DQN is the best solution, making the agent reach the target very quickly compared to the other improvements. Second one is using Noisy Networks for exploration, which makes the agent learn quite fast at the beginning. And when all of them are combined, the performance boost is quite noticeable. We can have a better view of this in the next plot, where I'm showing the moving average score in a rolling window of 100 episodes for all the algorithms:

<img src="images/performance_comparison.png" width="700" title="Performance comparison" />

## Ideas for Future Work
There are other algorithms I haven't tried and that could definitely improve the agent's performance, such as:
- Distributional Perspective on Reinforcement Learning (a.k.a. Categorical DQN). [Paper](https://arxiv.org/abs/1707.06887)
- Multi-Step Deep Reinforcement Learning. [Paper](https://arxiv.org/abs/1901.07510)
- Distributional Reinforcement Learning with Quantile Regression. [Paper](https://arxiv.org/abs/1710.10044)
- Hierarchical Deep Reinforcement Learning. [Paper](https://arxiv.org/abs/1604.06057)
- Rainbow: Combining Improvements in Deep Reinforcement Learning. [Paper](https://arxiv.org/abs/1710.02298)

We could (and should) additionally explore the effects of tunning the different hyperparameters and model architectures. This would definitely improve the performance, regardless the algorithm used. 

I'd be very curious to see the changes in performance of using only raw pixels as state input. This is a must experiement for another time.
