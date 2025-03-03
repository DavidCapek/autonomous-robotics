---
title: Localization and Factor graph
layout: default
has_children: true
nav_order: 7
author: Zuzana Jindrová
mathjax: true
---

# How to fuse almost anything: Localization and factor graphs
{: .no_toc }
This chapter discusses the estimation of the robot's position in a known environment. The first part is devoted to robot localization as a Maximum A Posteriori (MAP) estimation problem. The second part focuses on factor graphs as a convenient representation of the localization problem and their use for robot position estimation. 
{: .fs-5 .fw-300 }

<details open markdown="block">
<summary>
    Table of contents
</summary>
{: .text-delta }
1. TOC
{:toc}
</details>

# Localization problem
Localization of a robot is the process of determining its position in the environment. This process uses a probability distribution and Bayes' theorem to incrementally update the position estimate based on available measurements and the motion model.

## Bayes' theorem
Let $ A $ and $ B $ be a events and $ P(B) \neq 0 $. Then:

$$ P(A \mid B) = \frac{P(B \mid A) \cdot P(A)}{P(B)} $$ 

This equation says that the probability that a robot is in a certain state can be calculated based on measurements and a priori knowledge of the probability of that state.

### Example

Let's have a $N$ disease that 1% of the population has and a test that can detect this disease. For a sick person, the test comes out positive with a probability of 0.999 (the sensitivity of the test). In contrast, for a healthy person, the test comes out negative with a probability of 0.99 (the specificity of the test). Therefore, the test is not conclusive and the disease may not always be detected or a false alarm may occur. If a randomly selected person gives a positive test result, what is the probability that this person has the disease?

Víme:
* $ P(+ \mid N) = 0.999 $
* $ P(- \mid \neg N) = 0.99 $
* $ P(N) = 0.01 $

Řešení: 

$ P(+ \mid \neg N) = 1 - P(- \mid \neg N) = 1 - 0.99 = 0.01 $

$ P(\neg N) = 1 - P(N) = 1 - 0.01 = 0.99 $

$ P(+) = P(+ \mid N) \cdot P(N) + P(+ \mid \neg N) \cdot P(\neg N) = 0.999 \cdot 0.01 + 0.01 \cdot 0.99 \approx 0.02 $

$  $
$ P(N \mid +) = \frac{P(+ \mid N) \cdot P(N)}{P(+)} = \frac{0.999 \cdot 0.01}{0.02} \approx 0.502 $

## Probability measurement model
Measurements are generally affected by uncertainties, so various probability models such as the normal distribution are used. The probability density describes how probable a measurement is given the actual position of the robot. In general, the more measurements we have, the more accurate our position estimation will be.

## Localization problem definition
Robot localization is formally solved as a Maximum A Posteriori (MAP) estimation problem, which aims to find the most probable trajectory of the robot based on measurements and actions.
$$

x^* = \underset{x}{\operatorname{\argmax}} p(\mathbf{x} \mid \mathbf{z},\mathbf{u}) = \underset{x_1...x_t}{\operatorname{\argmax}} p(x_0...x_t \mid z_1...z_t, u_1,...u_t). 
$$

where:
* $ \mathbf{x^*_i} $ indicates the most probable states that the robot was in based on measurements and actions
* $ x_0, x_1, ..., x_t \in \mathbb{R}^n $ are the states of the robot. The state can represent, for example, temperature, position, battery voltage or robot orientation.
* $ u_1, u_2, ..., u_t \in \mathbb{R}^n $ are actions that lead to a change of states, i.e. $ x_t = x_{t-1} + u_t $
* $ z_1, z_2, ..., z_t \in \mathbb{R}^n $ are absolute or relative measurements to determine the state of the robot. These can be measurements from, for example, a GPS, lidar or thermometer.

<div align="center">
  <img src="{{ site.baseurl }}/assets/images/lec_02/localization_problem.png" width="750">
</div>

## Localization example: Absolute position measurements in wcf (world coordinate frame)
Consider a robot moving along an axis and having three absolute position measurements of 2 m, 3 m and 7 m ($z_1, z_2, z_3$). Using the MLE and Bayes' theorem, calculate the most probable position of the robot based on measurements such as:

$$ x^* = \argmax_x p(x \mid z_1, z_2, z_3) = \argmax_{x} \frac{p(z_1, z_2, z_3 \mid x) \cdot p(x)}{p(z_1, z_2, z_3)}. $$

The term $p(z_1, z_2, z_3) $ can be omitted from the calculation since it does not depend on the search position $x$ and only scales the probability. Similarly, the term $p(x) $ can be omitted since it is a uniform distribution and does not affect the function $\argmax$. The resulting formula can also be rewritten using the logarithm:

$$ x* = \arg\max_x \left( \prod_i p(z_i \mid x) \right) = \argmin_x \sum_i -\log p(z_i \mid x).$$

The following video shows the calculation process for the discrete probability distribution $p(z \mid x) $. The robot's position is most likely at 3 m.

<div align="center">
  <video src="{{ site.baseurl }}/assets/videos/lec_02/animation_robot.mp4" width="640" autoplay loop controls muted></video>
</div>

<div align="center">
  <video src="{{ site.baseurl }}/autonomous-robotics/assets/videos/lec_02/animation_robot.mp4" width="640" autoplay loop controls muted></video>
</div>

<div align="center">
  <img src="{{ site.baseurl }}/assets/images/lec_02/animation_robot.gif" width="750">
</div>

Similarly, this example can be solved for the continuous probability distribution $p(z_i \mid x)$. Specifically, for the normal distribution $ p(z_i \mid x) = \mathcal{N}(z_i;x, \sigma^2) $

$$ \begin{align*}
x^* &= \argmax_x p(x \mid z_1, z_2, z_3) = \argmax_x \left( \prod_i p(z_i \mid x) \right)= \\
&= \argmax_x \left( \prod_i \mathcal{N}(z_i;x, \sigma^2) \right) = \argmax_x \prod_i K \cdot \exp \left(  -\frac{\|z_i-x\|_2^2}{\sigma^2} \right) =  \\ 
&= \argmin_x \sum_i \|z_i-x\|_2^2 = \frac{\sum_i z_i}{N}. 
\end{align*}
$$

The following figure shows that the function takes its minimum in this example at point $4$ m.
<div align="center">
  <img src="{{ site.baseurl }}/assets/images/lec_02/continuous_example.png" width="750">
</div>

## Multivariate gaussian
In case we are trying to estimate the robot's position in space, we need to use a multivariate Gaussian probability distribution. Multivariate Gaussian is defined as follows:

$$
p(x) = \mathcal{N}(x; \mu, \Sigma) = \frac{exp(-\frac{1}{2}(x-\mu)^T\Sigma^{-1}(x-\mu))}{\sqrt{(2\pi)^n\det(\Sigma)}},
$$

where 
* $ x \in \mathcal{R}^n $ is real n-dimensional random column vector
* $ \mu \in \mathcal{R}^n $ is real n-dimensional mean 
* $ \Sigma \in \mathcal{R}^{n \times n} $ symmetric positive definite covariance matrix.
 
<div align="center">
  <img src="{{ site.baseurl }}/assets/images/lec_02/gauss.png" width="350">
</div>


Logarithm of Gaussian  is quadratic form:

$$
\log(\mathcal{N}(x;\mu, \Sigma)) = -\frac{1}{2}(x-\mu)^T\Sigma^{-1}(x-\mu) + C.
$$


## Localization example: Relative position measurements in rcf (robot coordinate frame)
In this case, the relative position of the robot is, for example, the distance from an obstacle or marker whose position we know. The solution will be similar to the previous example, except that here we convert the position of the wall into robot coordinate system ($m-x$) and use it as a parameter of the normal distribution of measurements $p(z_i \mid x, m)= \mathcal{N}(z_i;m-x, \sigma^2)$. 
The calculation then looks like this:

$$
\begin{align*}
x^* &= \argmax_x p(x \mid z_1, z_2, z_3, m) = \argmax_x \left( \prod_i p(z_i \mid x,m) \right) = \\ 
&= \argmax_x \left( \prod_i \mathcal{N}(z_i;m-x, \sigma^2) \right) = \argmax_x \prod_i K \cdot \exp \left(  -\frac{\|(m-x)-z_i\|_2^2}{\sigma^2} \right) = \\ 
&=\argmin_x \sum_i \|m-z_i-x\|_2^2 = \frac{\sum_i m-z_i}{N}. 
\end{align*}
$$

<div align="center">
  <img src="{{ site.baseurl }}/assets/images/lec_02/relative_measurement_example.png" width="650">
</div>

## SLAM example: Realtive position measurment in rcf
In case we do not know the position of the robot or the position of the object to which we are measuring the relative position of the robot, we minimize the previous equation through two variables $x$ and $m$.

$$
\begin{align*}
x^* &= \argmax_{x,m} p(x,m \mid z_1, z_2, z_3) = \argmax_{x,m} \left( \prod_i p(z_i \mid x,m) \right) = \\
&= \argmax_{x,m} \left( \prod_i \mathcal{N}(z_i;m-x, \sigma^2) \right) = \argmax_{x,m} \prod_i K \cdot \exp \left(  -\frac{\|(m-x)-z_i\|_2^2}{\sigma^2} \right) = \\
&= \argmin_{x,m} \sum_i \|m-z_i-x\|_2^2 = \frac{\sum_i m-z_i}{N}. 
\end{align*}
$$

One possible solution is the $(3, 9)$ solution from the previous example. Other solutions are all combinations of robot and wall positions that differ by $6$ m, so for example $(1, 7), (2, 8), (4, 10), ... $. 
For one unique solution, we need to have at least one absolute solution in addition, for example.


## Moving robot
Consider that the robot can move between measurements. So if we have two absolute measurements $z_1$ and $z_2$ these measurements are taken at time $t = 1$ and $t = 2$. At time $t = 1$ the robot is in state $x_1$ and at time $t = 2$ the robot is in state $x_2$.
So we are trying to estimate the robot's trajectory. This gives us two normal distributions, one for each position $x$. However, since there is a connection between the robot's positions in time for example the constraints on its velocity we add a relative measure $z_{12}$ of the distance $x_2-x_1$. Thus, the following computation tries to estimate the position of $x_i$ and $x_2-x_1$ so as to maximize the probability of measuring $z_i$ and $z_{12} of the normal distribution belonging to a given state.

$$
\begin{align*}
p(z_1 \mid x_1) &= \mathcal{N}(z_1; x_1, \sigma^2)\\
p(z_2 \mid x_2) &= \mathcal{N}(z_2; x_2, \sigma^2)\\
p(z_{12} \mid x_1, x_2) &= \mathcal{N}(z_{12}; x_2-x_1, \sigma^2)\\
\end{align*}
$$


$$
\begin{align*}
(x_1^*, x_2^*) &= \argmax_{x_1,x_2} p(x_1, x_2 \mid z_1, z_2, z_{12}) = \argmax_{x_1,x_2} p(z_1 \mid x_1) \cdot p(z_2 \mid x_2) \cdot p(z_{12} \mid x_1, x_2)= \\
&=\argmin_{x_1,x_2} ((z_1 - x_1)^2 +(z_2 - x_2)^2 + (z_{12} - (x_2-x_1))^2) = (1.5, 3.5)
\end{align*}
$$

<div align="center">
  <img src="{{ site.baseurl }}/assets/images/lec_02/movement_example.png" width="750">
</div>

Specifically, for the figure above, we try to maximize the probability of measuring $z_1$, $z_2$ and $z_{12}$ by the normal distribution of the corresponding color.
Thus, the optimal solution is in the situation where $(z_1-x_1) = (z_2-x_2) = (z_{12} -(x_2 - x_1)) = 0.5$. 

Since we know that the robot should have performed the action/movement $u_2$ to reach the state $x_2$ from the state $x_1$, we can use this information in estimating its position. Using the motion propability $p(x_2 \mid x_1, u_2) = \mathcal{N}(x_2;x_1+u_2, \sigma^2)$, which expresses that if the robot was at point $x_1$ and had to perform the motion action $u_2$ it will be at time $t=2$ with probability of normal distribution around point $x_2$. Using $u_2 = 2.5$ as the action and thus supporting the relative measurement $z_{1,2}$, the optimal solution will not change much from the previous one. The calculation with motion propability looks as follows:

$$
\begin{align*}
(x_1^*, x_2^*) &= \argmax_{x_1,x_2} p(x_1, x_2 \mid z_1, z_2, z_{12}, u_2) = \\
&= \argmax_{x_1,x_2} p(z_1 \mid x_1) \cdot p(z_2 \mid x_2) \cdot p(z_{12} \mid x_1, x_2) \cdot p(x_2 \mid x_1, u_2) = \\
&=\argmin_{x_1,x_2} ((z_1 - x_1)^2 +(z_2 - x_2)^2 + (z_{12} - (x_2-x_1))^2) + (x_2 -(x_1 + u_2))^2 \approx (1.4, 3.6).
\end{align*}
$$

## Linear model and non- linear model for moving robot
If we include, for example, rotation in the robot's motion, a non-linear trajectory is created. In the case of nonlinear motion, we use the nonlinear functions $h()$ and $g()$ to describe the probability model.  

|| Linear model | Non-linear model|
|:--------------|:--------------:|-----------------:|
| Absolute measurement probability| $p(z_1 \mid x_1) = \mathcal{N}(z_1;x_1, \sigma_1^2)$   |   $p(z_1 \mid x_1) = \mathcal{N}(z_1;h(x_1), \sigma_1^2)$
| Absolute measurement probability| $p(z_2 \mid x_2) = \mathcal{N}(z_2;x_2, \sigma_2^2)$   |   $p(z_2 \mid x_2) = \mathcal{N}(z_2;h(x_2), \sigma_2^2)$
| Relative measurement probability| $p(z_{12} \mid x_1, x_2) = \mathcal{N}(z_{12};x_2-x_1, \sigma_{12}^2)$   |   $p(z_{12} \mid x_1, x_2) = \mathcal{N}(z_{12};h(x_1, x_2), \sigma_{12}^2)$
| Motion probability| $p(x_2 \mid x_1, u_2) = \mathcal{N}(x_2;x_1+u_2, \sigma^2)$   |   $p(x_2 \mid x_1, u_2) = \mathcal{N}(x_2;g(x_1, u_2), \sigma^2)$

# Factor graph
Factor graph is bipartite graph $\mathcal{G} = \{\mathcal{U},\mathcal{V},\mathcal{E}\}$ witch two types of nodes: factors $\Phi_i \in \mathcal{U}$ and variables $x_j \in \mathcal{V}$. 

<div align="center">
  <img src="{{ site.baseurl }}/assets/images/lec_02/factor_graph.png" width="750">
</div>

And with edges $e_{ij} \in \mathcal{E}$ whitch are always between factor nodes and variable nodes. Podle počtu nodes, s kterými faktor spojen je nazýván jako unární, binární, ternární, ... 

<div align="center">
  <img src="{{ site.baseurl }}/assets/images/lec_02/edges_factor_graph.png" width="750">
</div>

Factor graph is convenient visualization of (sparse) problem structure and umožňuje simply formulovat MAP estimation in negative log space. 

$$
\begin{align*}
x_0^*, ... x_t^* = \argmax_{x_0, ... x_t} \prod_i \Phi_i(X_i) = \argmin_{x_0, ... x_t} \sum_i -log(\Phi_i(X_i))
\end{align*}
$$

If factors are linear there are closed-form solution available like least square or Kalman filtr method. 

## Example of factor graph
### SLAM example
An example with three relative measurements of the robot's distance from an obstacle with unknown robot and wall positions can be drawn as a factor graph with two nodes (robot position and wall position) and three factors representing the three measurements depending on both robot position and wall position. The factors are binary since they depend on two parameters.

<div align="center">
  <img src="{{ site.baseurl }}/assets/images/lec_02/factor_graph_example_2.png" width="750">
</div>


###  GPS localization example
The localization of the robot based on three absolute measurements of its position can be represented by a factor graph with one node (robot position) and three unary fractions corresponding to the three measurements.

<div align="center">
  <img src="{{ site.baseurl }}/assets/images/lec_02/factor_graph_example_1.png" width="750">
</div>
