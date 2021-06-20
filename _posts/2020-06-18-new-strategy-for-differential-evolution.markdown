---
layout: post
title: "A New Strategy for Differential Evolution"
date: 2021-06-20 15:30:00
categories: research
tags: [differential-evolution, evolution-strategy, reinforcement-learning, python]
permalink: /research/a-new-strategy-for-differential-evolution
video: /assets/images/de_strategy/ackley_2.mp4
---

<video class="b-lazy" data-src="/assets/images/de_strategy/ackley_2.mp4" type="video/mp4" autoplay muted playsinline loop style="display: block; margin: auto; width: 100%;" ></video>
<div class="text-block">
<center>
<p>Ackley Function Optimized with DE Using The Introduced Strategy</p>
</center>
</div>

In this blog post I introduced a dynamically scaled strategy called `scaledbest1bin` which outperforms other Differential Evolution (DE) strategies in "[Test functions for optimization](https://en.wikipedia.org/wiki/Test_functions_for_optimization)".

# Dynamically Scaled Strategies

In these strategies, difference is calculated as scaled distances of 2 random individuals to the current individual. For scaling, rank transformed rewards are used similar to Salimans, Tim, et al., 2017. But rank transformed rewards are between -1.0 and 1.0 instead of -0.5 and 0.5, so that, if two randomly chosen individuals are the best and the worst, the difference will be the same as `best1bin` or `rand1bin` strategy.

$$diff = rank\\_transform(r0\\\_reward) * (r0 - current) + rank\\_transform(r1\\\_reward) * (r1 - current)$$
$$mutation\\\_vector = best + mutation\\\_factor * diff$$

Pseudo Python Code:
```python
# scaledbest1bin strategy
idxs = np.random.choice(np.delete(np.arange(POPULATION_SIZE), j), 2, replace=False)
current = population[j]
sub_rewards = rank_transformation(rewards)[idxs]
distances = population[idxs] - current
diff = sub_rewards @ distances
population[best_idx] + MUTATION_FACTOR * diff
cross = np.random.rand(2) <= CROSSOVER_PROBABILITY
new_candidate = current.copy()
new_candidate[cross] = mutation_vector[cross]
```

<div style="display:flex">
     <div style="flex:1;padding-right:5px;">
         <video class="b-lazy" data-src="/assets/images/de_strategy/ackley_2.mp4" type="video/mp4" autoplay muted playsinline loop style="display: block; margin: auto; width: 133%;" ></video>
         <div class="text-block">
         <center>
         <p>Ackley</p>
         </center>
         </div>
     </div>
     <div style="flex:1;padding-right:5px;">
         <video class="b-lazy" data-src="/assets/images/de_strategy/rastrigin_2.mp4" type="video/mp4" autoplay muted playsinline loop style="display: block; margin: auto; width: 133%;" ></video>
         <div class="text-block">
         <center>
         <p>Rastrigin</p>
         </center>
         </div>
     </div>
     <div style="flex:1;padding-right:5px;">
         <video class="b-lazy" data-src="/assets/images/de_strategy/schaffer.mp4" type="video/mp4" autoplay muted playsinline loop style="display: block; margin: auto; width: 133%;" ></video>
         <div class="text-block">
         <center>
         <p>Schaffer</p>
         </center>
         </div>
     </div>
</div>

# Plots
<div style="display:flex">
     <div style="flex:1">
         <center>
         <img class="b-lazy" src=data:image/png;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw== data-src="/assets/images/de_strategy/plot_ackley_0.7_0.5_128.png" style="display: block; margin: auto; width: 75%;"/>
         </center>
         <div class="text-block">
         <center>
         <p>Ackley Function, Mutation Factor: 0.7, Crossover Probability: 0.5, Population Size: 128</p>
         </center>
         </div>
     </div>
</div>

<div style="display:flex">
     <div style="flex:1">
         <center>
         <img class="b-lazy" src=data:image/png;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw== data-src="/assets/images/de_strategy/plot_rastrigin_0.7_0.5_128.png" style="display: block; margin: auto; width: 75%;"/>
         </center>
         <div class="text-block">
         <center>
         <p>Rastrigin Function, Mutation Factor: 0.7, Crossover Probability: 0.5, Population Size: 128</p>
         </center>
         </div>
     </div>
</div>

<div style="display:flex">
     <div style="flex:1">
         <center>
         <img class="b-lazy" src=data:image/png;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw== data-src="/assets/images/de_strategy/plot_schaffer_0.7_0.5_128.png" style="display: block; margin: auto; width: 75%;"/>
         </center>
         <div class="text-block">
         <center>
         <p>Schaffer Function, Mutation Factor: 0.7, Crossover Probability: 0.5, Population Size: 128</p>
         </center>
         </div>
     </div>
</div>

# Code
The code used to generate plots and animations: [https://github.com/goktug97/DEStrategy](https://github.com/goktug97/DEStrategy)

The strategies are also implemented in [https://github.com/goktug97/de-torch](https://github.com/goktug97/de-torch)

# References

1. Salimans, Tim, et al. “Evolution Strategies as a Scalable Alternative to Reinforcement Learning.” ArXiv:1703.03864 [Cs, Stat], Sept. 2017. arXiv.org, http://arxiv.org/abs/1703.03864.

# Citation

```
@article{karakasli2021destrategy,
  title   = "A New Strategy for Differential Evolution",
  author  = "Karakasli, Goktug",
  journal = "goktug97.github.io",
  year    = "2021",
  url     = "https://goktug97.github.io/research/a-new-strategy-for-differential-evolution"
}
```
