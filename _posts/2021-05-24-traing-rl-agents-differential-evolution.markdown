---
layout: post
title:  "Training Neural Networks with Differential Evolution for Reinforcement Learning Tasks"
date:   2021-05-23 23:30:00
categories: research
tags: [differential-evolution, evolution-strategy, reinforcement-learning, python]
---

<div>
<center>
<img src="/assets/images/de/lunarlander.gif" width=100%>
</center>
</div>

Differential Evolution is an evolutionary method that evolves a population to optimize the given loss function. In each generation of differential evolution, a candidate population is created and parents are replaced with better performing children. To create this candidate population, parents are crossovered with mutation vectors that are
calculated with random individuals from the population. There are different strategies in differential evolution which affects how the mutation vector is calculated. For example, in `best1_bin` strategy, the mutation vector is calculated as $$\text{best\_individual} + \text{differential\_weight} * (\text{random\_individual}\_1 - \text{random\_individual}\_2)$$ Differential weight is also called mutation factor in the literature. For all experiments, `best1_bin` strategy is used.
For the recombination(crossover) part, each weight in parent is replaced with a mutated weight with a probability. This probability is called crossover probability in the literature.

Python pseudocode for creating a candidate population with `best1_bin` strategy;

```python
for p in population:
    p1, p2 = random.choice(population, 2)
    diff = p1.weights - p2.weights
    mutation_vector = population[best_id].weights + mutation_factor * diff
    mask = np.random.rand(len(mutation_vector)) <= crossover_probability 
    p.weights[mask] = mutation_vector[mask]
```

You can read more about it from [https://en.wikipedia.org/wiki/Differential\_evolution](https://en.wikipedia.org/wiki/Differential\_evolution) .

When I learn a new optimization algorithm, I always try it with reinforcement learning tasks to see how well it performs in this area of AI. Recently, I implemented a [library for Natural Evolution Strategies](https://github.com/goktug97/nes-torch) and since evolutionary algorithms are very similar in terms of implementation, it was really easy to modify the nes-torch to implement a [differential evolution library](https://github.com/goktug97/de-torch).

In all experiments the `LunarLander-v2` environment from OpenAI `gym` library is used. My first experiment was training a feedforward network that has no hidden layers. Also, in every experiment I used population size of `256` and varied only the mutation factor and the crossover probability hyperparameters which are `0.5` and `0.7` respectively for this experiment.

<div style="display:flex">
     <div style="flex:1">
         <center>
         <img src="/assets/images/de/highproblowdim.png" width=50%>
         </center>
         <div class="text-block">
         <center>
         <p>LunarLander-v2, No Hidden Layer, Mutation Factor: 0.5, Crossover Probability: 0.7</p>
         </center>
         </div>
     </div>
</div>

Above graph shows that differential evolution can quickly get good rewards in reinforcement learning tasks even with a really small network. Unfortunately, with the same settings, when the network size is increased, the results are not as good as expected.

<div style="display:flex">
     <div style="flex:1">
         <center>
         <img src="/assets/images/de/highprobhighdim.png" width=50%>
         </center>
         <div class="text-block">
         <center>
         <p>1 Hidden Layer with Size of 16, Mutation Factor: 0.5, Crossover Probability: 0.7</p>
         </center>
         </div>
     </div>
</div>

I found out that lower crossover probabilities work much better with high dimensionality. I tried really low values such as 0.05 and it trained very well.

<div style="display:flex">
     <div style="flex:1">
         <center>
         <img src="/assets/images/de/lowprobhighdim.png" width=50%>
         </center>
         <div class="text-block">
         <center>
         <p>1 Hidden Layer with Size of 16, Mutation Factor: 0.5, Crossover Probability: 0.05</p>
         </center>
         </div>
     </div>
</div>

So to calculate crossover probability based on network size, I come up with this formula;
$$ \frac{1}{\log(\text{size}) * \sqrt{2\pi}} * e^{-\frac{1}{2}} $$

The problem is, even though DE achieved really high rewards during training, trained agents didn't perform well during evaluation. I thought some sort of regularization of the weights might help agents to generalize better. So I applied L2 decay to mutated individuals to make them generalize better. While it helped a little bit, they were still not as good as NES (Natural Evolution Strategy) trained agents. So I combined two algorithms together, using DE to quickly explore good regions and then fine tuning the trained agents using NES. This is a good strategy because DE also performs very well in sparse reward environments such as `MountainCar-v0`.

<div style="display:flex">
     <div style="flex:1;padding-right:5px;">
         <img src="/assets/images/de/finetune.png">
         <div class="text-block">
         <center>
         <p>50 Generations DE and 50 Generations NES Fine Tuning</p>
         </center>
         </div>
     </div>
     <div style="flex:1;padding-left:5px;">
         <img src="/assets/images/de/only_de.png">
         <div class="text-block">
         <center>
         <p>100 Generations DE</p>
         </center>
         </div>
     </div>
</div>
<div class="text-block">
<center>
<p>No Hidden Layer, Mutation Factor: U(0.7, 1.0), Crossover Probability: 0.0675 (Calculated with the formula)</p>
</center>
</div>
</div>

`U(0.7, 1.0)` means, in each generation the mutation factor is uniformly sampled between `0.7` and `1.0`. While the both versions performed very well during training, only the fine tuned version performed reasonable during evaluation.

```
Fine Tuned Evaluation Reward: 260.96171919546
Only DE Evaluation Reward: 69.39153938133028
```

The GIF in the intro is the fine tuned policy.

In conclusion, I think differential evolution has a potentional in reinforcement learning tasks and when combined with natural evolution strategies it is a really good algorithm.

# Code
[https://github.com/goktug97/de-torch/blob/master/examples/finetuning.py](https://github.com/goktug97/de-torch/blob/master/examples/finetuning.py)


# References
- [https://en.wikipedia.org/wiki/Differential_evolution](https://en.wikipedia.org/wiki/Differential_evolution)
- [https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.differential_evolution.html](https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.differential_evolution.html)

# Citation

```
@article{karakasli2021de,
  title   = "Training Neural Networks with Differential Evolution for Reinforcement Learning Tasks",
  author  = "Karakasli, Goktug",
  journal = "goktug97.github.io",
  year    = "2021",
  url     = "https://goktug97.github.io/research/2021/05/23/traing-rl-agents-differential-evolution.html"
}
```
