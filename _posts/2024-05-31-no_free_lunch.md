---
layout: distill
title: The No-Free-Lunch Theorem and Sheaf-Contextuality
description: How the impossibility of a universal learner can be seen as the non-existence of global sections of a 'learner' presheaf
tags: MachineLearning CategoryTheory 
giscus_comments: false
date: 2024-05-31
featured: false
citation: true    
bibliography: 2024-05-30-no_free_lunch.bib
 

toc:
  - name: Statistical learning theory in a nutshell
  - name: The No-Free-Lunch theorem
  - name: Sheaf-theoretic Contextuality
  - name: The Learner Presheaf
  - name: Contextuality of PAC-learning
  - name: What's next? Cohomology, Compactness, Category
  - name: Conclusion

tikzjax: true  
#Added {% include scripts/tikzjax.liquid %} to distill.liquid
    
---


The *No-Free-Lunch theorem* is a fundamental limitative result in the theory of machine learning. It says that there cannot be a universal learner. (We discuss the precise statement below.) I recently covered it [my course](https://levinhornischer.github.io/FoundAI/), and it got me wondering whether there are any links to another topic known as *contextuality*, i.e., situations where we have a family of data which is *locally consistent, but globally inconsistent* <d-cite key="Abramsky2015"></d-cite>. Examples include quantum mechanics (for which this was first developed), but also databases, constraint satisfaction, logical paradoxes, and others. 

In this post, I explore whether the impossibility of a universal learner can be seen as an instance of contextuality. The idea is:
 
* while we can have various learners that work correctly on various *parts* of the input space (locally consistent), 
* we don't have a single learner that works correctly on *all* of the input space (globally inconsistent). 

We make this precise by defining what we will call the *learner presheaf* and by providing a locally consistent family of learners (i.e., compatible sections of the presheaf) that cannot be globally consistent (i.e., the presheaf does not have global sections). 

I'm not aware of work in this direction<d-footnote>Closest seems to come <d-cite key="Bowles2023"></d-cite>, but they don't use sheaves. If you know of some work, I'd be very happy to hear about it.</d-footnote>, but it promises to make applicable the powerful tools of contextuality to machine learning.

- [x] A quick introduction to statistical learning theory as the classic theory of machine learning
- [x] A statement of the No-Free-Lunch theorem saying that a universal learner doesn't exist
- [x] An introduction to sheaf-theoretic contextuality
- [x] How to apply sheaf-theoretic contextuality to machine learning
- [x] Make precise the main idea: learners on the different parts of the input space that cannot be glued to a learner on the whole input space
- [x] Some intersting open questions



## Statistical learning theory in a nutshell

Statistical learning theory was developed as the theory of machine learning <d-cite key="ShalevShwartz2014"></d-cite>. It studies when a learner can make correct predictions for future inputs after having seen finitely many past inputs together with their correct prediction (aka supervised learning). 

<div class="fake-img l-body-outset">
<p><em>Just a quick comment on the broader context. There is some disconnect between theory and practice: this theory does not fully account for the practices of modern deep learning. For further reading on this, see, e.g., <d-cite key="Berner2022"></d-cite> or <d-cite key="Belkin2021"></d-cite>. But this topic, together with the many other approaches to a theory of machine learning, is a something for another occasion. Here we will continue and see what the classic theory has to say.</em></p>
</div>



Let's illustrate such a learner with an example. The learner gets a dataset $S$ consisting of, say, 100,000 pairs $(x,y)$ with a pixel image $x$ of an animal and the label $y$ which is $1$ or $0$ depending on whether the depicted animal is a dog or not. Based on that, the learner has to come up with a general prediction rule $h$ that takes as input an image $x$ (whether already seen or new) and produces as output a label $h(x) = y$ describing whether the learner takes this image to depict a dog. If our learner is a neural network, it updates its weights according to the backpropagation algorithm as it goes through batches of the dataset. Whatever the setting of weights it arrives at after this training process, it will compute a function $h$ mapping inputs $x$ to outputs $h(x)$.<d-footnote>Technically, the network maps $x$ to logits, which then can be turned into probabilities, and the output is taken to be $1$ in a given dimension once the corresponding probability exceeds $0.5$.</d-footnote>

Typically, the learner is biased—or, positively formulated, has prior knowledge—and considers only certain functions from inputs to outputs to be potential prediction rules. (This helps avoiding overfitting to the training data, since with fewer prediction rules there are fewer options that fit the data too well.) Our neural network, for example, will only consider those functions that can be realized with some setting of weights.<d-footnote>By the universal approximation theorems the functions that are realized by neural networks can, however, approximate any given continuous function (e.g., from inputs to logits) arbitrarily well.</d-footnote> The set of functions that the learner considers is called the *hypothesis class* or its *inductive bias* (i.e., its bias when doing what, outside of math, is called induction: namely, choosing a general rule after seeing finitely many examples).



This already motivates the formal framework of statistical learning theory. (We use the following common notation: If $M$ is a set, $M^*$ is the set of all finite sequences of elements from $M$. If $X$ and $Y$ are sets, then $Y^X$ is the set of all functions from $X$ to $Y$.)

* We have a set $X$ (the *domain set* or *input space*). 

In our example, $X = \mathbb{R}^n$, i.e., we describe our pixel images as vectors $(x_0, x_1 , \ldots , x_{n-1})$ where $x_i$ is the value of the $i$-th pixel (with $n$ pixels in total).

* We have a set $Y$ of labels (the *label set* or *output space*). Often, $Y = \lbrace 0,1\rbrace$, in which case we also write $Y = 2$.

In our example, we also have $Y = 2$.
 
* We have a set $H \subseteq Y^X$ (the *hypothesis class* of the learner). 

In our example, this was the set of functions $h : X \to Y$ that the neural network can realize with some setting of its weights.

* We have a function $A : (X \times Y)^* \to H$ (the *learner*), which maps a *dataset* $S$ (i.e., a finite sequence of input-output pairs) to a *prediction rule* $h = A(S)$. 

In our example, $A(S)$ is the function that the neural network realizes with the weights obtained after initializing the weights and then doing, say, 10 repetitions (aka epochs) of going through batches of $S$ and updating the weights with backpropagation (we ignore that this is not a fully deterministic process).

The last definition we need is what it means for a learner to be 'good', i.e., to produce correct prediction rules. Statistical learning theory accounts for the fact that the learners operate in a statistical and noisy environment. So rather than saying that a learner will surely be completely correct, we focus on when a learner will *p*robably be *a*pproximately *c*orrect---aka a PAC learner. For such a learner, there should only be a small probability of $\delta$ that the learner comes up with a prediction rule with more than $\epsilon$-much error.


More precisely, for any confidence parameter $\delta$ and accuracy parameter $\epsilon$, there should be a number of samples $m$, such that if the learner takes a set $S$ of $m$-many input-output samples of the world (which is governed by a probability distribution $P$ that the learner does not know), then, with probability at least $1 - \delta$ over the choice of samples, the learner's prediction $A(S)$ has an error of less than $\epsilon$, provided there is a correct hypothesis in the first place. Here the error of a function $h : X \to Y$ is measured as the probability that the function does not produce the right label:  

$$
L_P (h) := P \big( \{
(x,y) \in X \times Y 
:
h(x) \neq y
\} \big).
$$

This is also called the *true error* (which is inaccessible to the learner) to contrast it with the *empirical error* (which is accessible to the learner), that is defined as difference of prediction relative to a given dataset $S$:

$$
L_S (h) := \frac{1}{|S|} \big\lbrace
(x,y) \in S : h(x) \neq y 
\big\rbrace. 
$$


Now, the formal definition of PAC-learnability reads as follows. (We write $\mathsf{Prob}(X)$ for the set of probability distributions on the set $X$; and for simplicity we ignore questions of measurability, i.e., we don't specify the $\sigma$-algebras on the sets that we work with.) 

* We say $A$ *PAC-learns* $(X,Y,H)$ if $A$ is a function from $(X \times Y)^*$ to $H$ such that, for all $\epsilon, \delta \in (0,1)$, there is $m \in \mathbb{N}$ such that, for all $P \in \mathsf{Prob}(X \times Y)$, if there is $h \in H$ such that $L_P (h) = 0$, then 

  $$  
  P^m \big( \{ 
  (x,y) \in (X \times Y)^m 
  :
  L_P( A(S) ) \geq \epsilon
  \} \big)
  < \delta. 
  $$

With this framework of statistical learning theory, we can move to one of its fundamental results.




## The No-Free-Lunch theorem

For a given domain set $X$ and label set $Y$, there are many learners: they can vary in their hypothesis class (i.e., their inductive bias), and even if they agree on that, they can still generalize to different prediction rules from the very same dataset that they saw. So wouldn't it be nice if there is a *universal* learner: one that, on any task, performs as well as any other learner? If so, we would get a 'free lunch' since we wouldn't need to design, for each task, a learner specific to this task, but could always just blindly use the universal learner instead. 

At first sight we might hope for a positive answer: After all, there are universal Turing machines that can simulate any other Turing machine. So maybe we can build a universal learner that simulates other learners, collects their prediction rules, and aggregates those into a combined prediction rule, hoping for a [wisdom of the crowd](https://en.wikipedia.org/wiki/Jury_theorem) effect. 
But—as spoiled by the section title (sorry!)—the No-Free-Lunch theorem says that such a universal learner cannot exist. In other words, for every learner, there is a task on which it fails, even though another learner succeeds.<d-footnote>For a discussion of interpretations of the No-Free-Lunch theorem—connecting to the philosophical problem of induction—, see <d-cite key="Sterkenburg2021"></d-cite></d-footnote>

Formally, the No-Free-Lunch theorem is stated as follows (in the version of <d-cite key="ShalevShwartz2014"></d-cite> on page 61):

* Let $X$ be an infinite set, let $Y = 2$, and $H = 2^X$. Then no learner PAC-learns $(X,Y,H)$. In fact, slightly stronger, for any $A : (X \times 2)^* \to H$, if we choose $\epsilon = \frac{1}{8}$ and $\delta = \frac{1}{7}$, then, for any $m \in \mathbb{N}$, there is $P \in \mathsf{Prob}(X \times Y)$ such that there is $h \in H$ with $L_P (h) = 0$ but 

  $$  
  P^m \big( \{ 
  S \in (X \times Y)^m 
  :
  L_P( A(S) ) \geq \epsilon
  \} \big)
  \geq \delta. 
  $$
  
  (Note that we can hence think of $P$ as a probability distribution $D$ on $X$ together with a deterministic labeling function $h : X \to Y$.)

So learner $A$ fails on this task while the highly biased learner with hypothesis class $\lbrace h \rbrace$ succeeds.







## Sheaf-theoretic Contextuality

Now let's move to what initially seems like a completely different topic: contextuality. As already mentioned, this concerns situations where we have locally consistent data that is globally inconsistent<d-cite key="Abramsky2015"></d-cite>. Famously, this occurs in quantum mechanics. But the sheaf-theoretic formalization of this phenomenon, which was first developed by Abramsky and Brandenburger<d-cite key="Abramsky2011"></d-cite>, was then shown to also describe other, 'classical' phenomena like databases or constraint satisfaction.


Let's illustrate this with the standard example, depicted on the right.
We have two agents, and of course we call them Alice and Bob. 
Both sit at opposite sides of a box, inside of which is a mystery system. Both have two buttons in front of them and a screen. Now, at each round of the experiment, they each can press one of their two buttons and their screen will read out either 0 or 1, indicating whether the underlying system currently has the property that is measured by the respective button. They do this many rounds and record the following statistics:

1. Whenever Alice chose the first button ($a_1$) and Bob also chose the first button ($b_1$), then in $\frac{1}{2}$ of the cases Alice's screen showed $1$ and Bob's showed $0$, and in the other $\frac{1}{2}$ of the cases Alice's screen showed $0$ and Bob's $1$. So it was never observed that their screens showed the same readings. In the figure, we hence have a bright blue line from the $1$ above $a_1$ to the $0$ above $b_1$, and another one from $0$ above $a_1$ to the $1$ above $b_1$. 
2. Whenever Alice chose the second button ($a_2$) and Bob the first ($b_1$), then in $\frac{3}{8}$ of the cases both screens showed $1$ (hence a reasonably bright blue line from the $1$ above $b_1$ to the $1$ above $a_2$), in $\frac{3}{8}$ of the cases both screens showed $0$ (hence a reasonably bright blue line between the $0$'s), in $\frac{1}{8}$ of the cases Alice's screen showed $0$ while Bob's showed $1$ (hence a faint blue line from the $1$ above $b_1$ to the $0$ above $a_2$), and in the remaining $\frac{1}{8}$ of the cases Alice's screen showed $1$ while Bob's showed $0$ (hence the faint blue line form $0$ to $1$).
3. Whenever Alice chose the first button ($a_1$) and Bob the second ($b_2$), they get the same distribution of screen readings as in 2.
4. Whenever Alice chose the second button ($a_1$) and Bob also the second ($b_2$), they get the reversed distribution: in $\frac{1}{8}$ of the cases both screens showed $0$, in $\frac{1}{8}$ of the cases both screens showed $0$, in $\frac{3}{8}$ of the cases Alice's screen showed $1$ while Bob's showed $0$, and in $\frac{3}{8}$ of the cases Alice's screen showed $0$ while Bob's showed $1$.


<div class="fake-img l-gutter">
<script type="text/tikz">
\begin{tikzpicture}[dot/.style={draw,circle,minimum size=1.5mm,inner sep=0pt,outer sep=0pt,fill=lightgray}]
  
\node[dot, label={[font=\normalsize,text=gray]left:$a_1$}] at (0,0)  (a) {};
\node[dot, label={[font=\normalsize,text=gray]right:$b_1$}] at (2,0)  (b) {};
\node[dot, label={[font=\normalsize,text=gray]right:$a_2$}] at (3,1)  (c) {};
\node[dot, label={[font=\normalsize,text=gray]left:$b_2$}] at (1,1)  (d) {};

\draw[-, lightgray] (a)--(b);
\draw[-, lightgray] (b)--(c);
\draw[-, lightgray] (c)--(d);
\draw[-, lightgray] (d)--(a);

\node[dot, label={[font=\small,text=gray]left:$0$}] at (0,3)  (a0) {};
\node[dot, label={[font=\small,text=gray]right:$0$}] at (2,3)  (b0) {};
\node[dot, label={[font=\small,text=gray]right:$0$}] at (3,4)  (c0) {};
\node[dot, label={[font=\small,text=gray]left:$0$}] at (1,4)  (d0) {};

\node[dot, label={[font=\small,text=gray]left:$1$}] at (0,4.2)  (a1) {};
\node[dot, label={[font=\small,text=gray]right:$1$}] at (2,4.2)  (b1) {};
\node[dot, label={[font=\small,text=gray]right:$1$}] at (3,5.2)  (c1) {};
\node[dot, label={[font=\small,text=gray]left:$1$}] at (1,5.2)  (d1) {};

\draw[dashed, lightgray] (a)--(a1);
\draw[dashed, lightgray] (b)--(b1);
\draw[dashed, lightgray] (c)--(c1);
\draw[dashed, lightgray] (d)--(d1);

\draw[cyan!100, thick] (a0)--(b1);
\draw[cyan!100, thick] (a1)--(b0);

\draw[cyan!75, thick] (b0)--(c0);
\draw[cyan!75, thick] (b1)--(c1);
\draw[cyan!25, thick] (b0)--(c1);
\draw[cyan!25, thick] (b1)--(c0);


\draw[cyan!25, thick] (c0)--(d0);
\draw[cyan!25, thick] (c1)--(d1);
\draw[cyan!75, thick] (c0)--(d1);
\draw[cyan!75, thick] (c1)--(d0);

\draw[cyan!75, thick] (d0)--(a0);
\draw[cyan!75, thick] (d1)--(a1);
\draw[cyan!25, thick] (d0)--(a1);
\draw[cyan!25, thick] (d1)--(a0);

\end{tikzpicture}
</script>
<p>The <em>Bell table</em></p>
</div> 


We now wonder: what do these observations say about our mystery system? (We cannot dismiss the example as imaginary, because it is physically realizable according to quantum mechanics <d-cite key="Abramsky2015"></d-cite>.)

Naturally, we would say that, at any given time, the system is in some state, $s$, and this state determines the outcomes of each of the measurements $a_1, a_2, b_1, b_2$ if they are performed on the system in this state. There is some probability distribution on the set of all states of the system, describing how likely it is to find the system in a given state (and of course this distribution is independent of whatever Alice and Bob do). So the measurements measure objective properties of the system, which obtain independently of which measurements we perform; and the distributions over the measurement outcomes come from the distribution over system states. 

But from our observations, we must conclude that our system cannot be of such a form! No probability measure on the state space of our system can give rise to the observed distributions on the measurements. 

We can say this more precisely with the 'locally consistent but globally inconsistent' terminology. We considered above four choices of measurements: 
$C_1 = \lbrace a_1, b_1 \rbrace$, 
$C_2 = \lbrace a_2, b_1 \rbrace$,
$C_3 = \lbrace a_2, b_2 \rbrace$, and
$C_4 = \lbrace a_1, b_2 \rbrace$.
For each $C_i$, we got a probability distribution $D_i$ on the measurements in $C_i$ (so $D_i$ is a probability measure on the set of the potential outcomes $\lbrace (0,0), (0,1), (1,0), (1,1) \rbrace$). One can check that these distributions are compatible in the sense that two distributions $D_i$ and $D_j$ have the same [marginals](https://en.wikipedia.org/wiki/Marginal_distribution) on their overlap $C_i \cap C_j$.
Thus, we have a *locally consistent* family $\lbrace D_i \rbrace$ of observation data.
However, this family of data is not *globally consistent*, because there cannot be a distribution $D$ on all measurements $\lbrace a_1 , a_2 , b_1 , b_2 \rbrace$ such that each $D_i$ is a marginalization of $D$ (i.e., $D$ is the joint distribution).


{% details Click here for a delightfully simple proof presented in <d-cite key="Abramsky2015"></d-cite>%}
Consider the state space of our system with its probability measure $P$ giving rise to the observed distributions on the measurements. Consider the following events and their associated logical formula:

| ---                                          | ---                                        |
| $E_1$: $a_1$ and $b_1$ yield distinct values | $\varphi_1 = a_1 \leftrightarrow \neg b_1$ |
| $E_2$: $b_1$ and $a_2$ yield distinct values | $\varphi_2 = b_1 \leftrightarrow \neg a_2$ |
| $E_3$: $a_2$ and $b_2$ yield the same values | $\varphi_3 = a_2 \leftrightarrow b_2$      |
| $E_4$: $b_2$ and $a_1$ yield distinct values | $\varphi_4 = b_2 \leftrightarrow \neg a_1$ |

Looking at the observation statistics, we have
$P(E_1) = \frac{1}{2} + \frac{1}{2} = 1$ and, for $i = 2,3,4$, we have 
$P(E_i) = \frac{3}{8} + \frac{3}{8} = \frac{3}{4}$, so 
$\sum P (E_i) = 1 + 3 \frac{3}{4} = 3.25$.
On the other hand, basic logical reasoning shows that the events cannot occur simultaneously: otherwise we get the contradiction

$$
a_1 
\overset{\varphi_1}{\leftrightarrow} 
\neg b_1 
\overset{\varphi_2}{\leftrightarrow}
a_2
\overset{\varphi_3}{\leftrightarrow}
b_2
\overset{\varphi_4}{\leftrightarrow}
\neg a_1.
$$

So if one of the events obtains, one of the remaining ones cannot obtain, so

$$ P(E_4) \leq 
P( \bigcup_{i = 1}^3 E_i^c ) \leq
\sum_{i = 1}^3 P( E_i^c) = 
\sum_{i = 1}^3 1 - P( E_1).
$$

By bringing all $P$'s to the left side, we get 

$$
\sum_{i = 1}^4 P (E_i) \leq 3
$$

contradicting our observation that 
$\sum_{i = 1}^4 P (E_i) = 3.25$.
{% enddetails %}
Abramsky <d-cite key="Abramsky2017"></d-cite> (on page 9) analyzes this situation thus:

> quantum mechanics predicts correlations which exceed those which can be achieved by any classical mechanism. This is the content of Bell's theorem [...] and in many ways the starting point for the whole field of quantum information.




So our system cannot be of the form we initially thought and *somehow* the properties of our system depend on the choice of measurements that we perform, i.e., the measurement context. How exactly is formalized with the sheaf-theoretic approach to contextuality. This goes as follows (e.g. <d-cite key="Abramsky2017"></d-cite>). 

* We have a set $X$ of *variables* (aka measurements, features, observations, etc.). 

In our example, $X = \lbrace a_1 , a_2 , b_1 , b_2 \rbrace$.

* We have a set $\mathcal{C} \subseteq \mathcal{P} (X)$ of *compatible contexts*, i.e., those sets of measurements that can be performed at a given time in an experiment. We require that $\mathcal{C}$ covers $X$ (i.e., every element of $X$ is in some context in $\mathcal{C}$), so every variable we consider can also be measured in some context (otherwise don't consider it).

In our example, $\mathcal{C} = \big\lbrace
\lbrace a_1 , b_1 \rbrace,
\lbrace a_1 , b_2 \rbrace,
\lbrace a_2 , b_1 \rbrace,
\lbrace a_2 , b_2 \rbrace
\big\rbrace$.

* We have a function $P$ that assigns to every context $C \subseteq X$, whether compatible or not, the set of possible data yielded by the measurements in $C$.

In our example, $P(C) = \mathsf{Prob} (2^C)$, where we think of $2^C$ as the set of possible outcomes when measuring the variables in $C$. So for compatible contexts $C$, the set $P(C)$ includes the 'local' distributions that we have observed. For contexts $C$ outside $\mathcal{C}$, the set $P(C)$ contains distributions that we might not directly observe, but which may be constrained by the observed distributions. In particular, we saw that $P(X)$ contains any potential 'global' distribution. 
To talk about this transition between the local and the global, we also need the following.

* For each $C \subseteq D$ in $\mathcal{P}(X)$, we have a function 

  $$\rho^D_C : P(D) \to P(C),$$
  
  called the *restriction map*. If $d \in P(D)$, we also write $d \restriction C := \rho^D_C(d)$. We require that $\rho^C_C$ is the identity function on $P(C)$, and if $C \subseteq D \subseteq E$, then $\rho^D_C \circ \rho^E_D = \rho^E_C$.

In our example, $\rho^D_C : P(D) \to P(C)$ takes a distribution $d$ on $2^D$ to the distribution $d \restriction C$ on $2^C$ obtained by marginalizing on the variables in $C$.

* An element $d \in P(C)$ is called a (local) *section* and an element $d \in P(X)$ is called a *global section*. A family $\lbrace d_C \in P(C) : C \in \mathcal{C} \rbrace$ is called *locally consistent* (or *compatible*) if, for all $C,D \in \mathcal{C}$, we have

  $$d_C \restriction C \cap D = d_D \restriction C \cap D.$$

  The family is called *globally consistent* if there is a global section $d \in P(X)$ such that, for all $C \in \mathcal{C}$, we have $d \restriction C = d_C$. A locally consistent family is called *contextual* if it is not globally consistent (i.e., doesn't have a global section).


In the language of category-theory, this means that $P : \mathcal{P}(X)^\mathsf{op} \to \mathsf{Set}$ is a *presheaf*, where $\mathcal{P}(X)^\mathsf{op}$ is the poset category of the powerset with reverse inclusion.<d-footnote> This suggests that we can have a more general category of contexts rather than the powerset of variables (e.g., with several inclusion morphisms), but this setting is enough for most purposes.</d-footnote> In the language of topology, we think of $X$ as a space equipped with the discrete topology $\mathcal{P}(X)$, and we think of $\mathcal{C}$ as an open cover.






 

## The Learner Presheaf


It is high time to bring together the two strands: how we can view statistical learning theory through the lens of contextuality. So assume we consider a machine learning setting, i.e., a typically infinite domain set $X$ (e.g., pixel images), the label set $Y = 2$ (i.e., binary classification), and a learning architecture $H \subseteq 2^X$. For $C \subseteq X$, we define the notation 

$$
H_C := \{ h \restriction C : h \in H \}.
$$

(As usual, $h \restriction C$ is the function from $C$ to $Y$ mapping $x \in C$ to $h(x) \in Y$.)

We will consider the domain $X$ as our set of variables, and our compatible contexts are all finite subsets of $X$, i.e., $\mathcal{C} := \mathcal{P}_\mathsf{fin} (X)$. After all, any actual learning context deals with finitely many inputs $x \in X$ that are, e.g., used in training data or also in test data to check a suggested prediction rule of a learner.  

<div class="fake-img l-body-outset">
<p><em> Now, the trick to get a presheaf is that we don't just view $x \in X$ as an object that should be labeled. We also view $x \in X$ as a variable that we measure in terms of how it figures in learners that try to label the object $x$.</em></p>
</div>

Concretely, we define the *learner presheaf* $P_H$ as follows:

* For $C \subseteq X$, define 

  $$
  P_H(C)
  :=
  \big\{ 
  A : A \text{ PAC-learns } (C,Y,H_C)
  \big\}, 
  $$

  so each $A$ in $P(C)$ is a function $(C \times Y)^* \to H_C$.

* If $D \supseteq C$, define the restriction map $\rho^D_C : P_H(D) \to P_H(C)$ by 

  $$
  A \mapsto [ S \mapsto A(S) \restriction C ],
  $$

  so the learner $A$ in $P_H(D)$ is mapped to the learner $\rho^D_C (A)$ in $P_H(C)$, which is defined by mapping $S$ to $A(S) \restriction C$. 
  
{% details Click for a proof that this is well-defined and compositional %}
For well-definedness, we need to check that, for a learner $A$ in $P(D)$, the function $B := \rho^D_C (A)$ is in $P_H(C)$, i.e., PAC-learns $(C,Y,H_C)$.

First, $B$ is of the right type: it maps an element 

$$ S \in (C \times Y)^* \subseteq (D \times Y)^* $$ 

to $h_S := A(S) \restriction C$, which is in $H_C$ because $A(S)$ is in $H_D$, so there is a function $h \in H$ with $A(S) = h \restriction D$, so, since $C \subseteq D$, also $A(S) \restriction C = h \restriction C$.

Second, to show that $B$ PAC-learns $(C,Y,H_C)$, let $\epsilon, \delta \in (0,1)$. Since $A$ PAC-learns $(D,Y,H_D)$, there is $m \in \mathbb{N}$ such that, for all $P \in \mathsf{Prob}(D \times Y)$, if there is $h \in H_D$ such that $L_P (h) = 0$, then 

$$  
P^m \big( \{ 
S \in (D \times Y)^m 
:
L_P( A(S) ) \geq \epsilon
\} \big)
< \delta. 
$$

So, choosing the very same $m$, it suffices to show that, for $Q \in \mathsf{Prob}(C \times Y)$ such that there is $g \in H_C$ with $L_Q (g) = 0$, we have

$$
Q^m \big( \{ 
S \in (C \times Y)^m 
:
L_Q( B(S) ) \geq \epsilon
\} \big)
< \delta. 
$$

Define the measure $P$ on $D \times Y$ by taking the distribution of $Q$ on $C \times Y$ and putting 0 weight on the rest of $D \times Y$ (i.e., $P(Z) := Q (Z \cap (C \times Y))$). 
Note that, for $f : D \to Y$, we have

$$\begin{aligned}
L_P(f) &= 
P \big( (x,y) \in D \times Y : f(x) \neq y \big) \\ &= 
Q \big( (x,y) \in C \times Y : f(x) \neq y \big) \\ &=  
L_Q(f \restriction C).
\end{aligned}$$

Now, since $g \in H_C$, there is $\overline{h} \in H$ with $g = \overline{h} \restriction C$. Define $h := \overline{h} \restriction D \in H_D$. Since $g = h \restriction C$, we have

$$L_P(h) = L_Q(h \restriction C) = L_Q (g) = 0.$$

So, by assumption, we have

$$  
P^m \big( \{ 
S \in (D \times Y)^m 
:
L_P( A(S) ) \geq \epsilon
\} \big)
< \delta. 
$$

Hence, since $L_Q( B(S) ) = L_Q( A(S) \restriction C ) = L_P( A(S) )$, we have


$$
\begin{aligned}
&
Q^m \big( \{ 
S \in (C \times Y)^m 
:
L_Q( B(S) ) \geq \epsilon
\} \big) \\ =& 
Q^m \big( \{ 
S \in (C \times Y)^m 
:
L_P( A(S) ) \geq \epsilon
\} \big) \\ =&
P^m \big( \{ 
S \in (D \times Y)^m 
:
L_P( A(S) ) \geq \epsilon
\} \big) < \delta,
\end{aligned}
$$

as needed.






For compositionality, we have that $\rho^C_C$ is the identity function on $P_H(C)$, since it maps $A$ to the mapping $S \mapsto A(S) \restriction C = A(S)$. And if $C \subseteq D \subseteq E$, then $\rho^D_C \circ \rho^E_D = \rho^E_C$ because, for $A \in P_H(E)$ and $S \in (C \times Y)^*$, we have:

$$ \begin{aligned}
\rho^D_C \circ \rho^E_D (A) (S) &= 
\rho^D_C \big(  [ S' \mapsto A(S') \restriction D ] \big) (S) \\ & =
[ S' \mapsto A(S') \restriction D ] (S) \restriction C \\ &=
( A(S) \restriction D ) \restriction C \\ &=
A(S) \restriction C \\ &=
\rho^E_C (A) (S),
\end{aligned}
$$
as needed.
{% enddetails %}



## Contextuality of PAC-learning

Given the learner presheaf, do we get contextuality, i.e., local consistency but global inconsistency? Yes, as we'll see now.

### Global inconsistency

The No-Free-Lunch theorem immediately implies that, for $X$ infinite and $H = 2^X$, we do not have *any* global sections, i.e., $P_H(X) = \emptyset$. So we cannot have globally consistent data.

In fact, we can be more precise. The *fundamental theorem of PAC learning* says that there is a PAC-learner for $(X,Y,H)$ iff $H$ has finite VC-dimension (see <d-cite key="ShalevShwartz2014"></d-cite>, sec. 6.4). The VC-dimension measures the complexity of a hypothesis class $H \subseteq Y^X$, and is defined as follows. 
First, given $C \subseteq X$, we say $H$ *shatters* $C$ if $H_C = Y^C$.
Then the *VC-dimension* of $H$ is the maximal size of a finite set $C \subseteq X$ that can be shattered by $H$. If $H$ shatters sets of arbitrarily large size, it has infinite VC-dimension.
Hence

* For a hypothesis class $H \subseteq Y^X$, the learner presheaf $P_H$ has a global section iff 

  $$ \lbrace \vert C \vert : C \in \mathcal{C} \text{ with } H_C = Y^C  \rbrace $$ 
  
  is bounded.

Thus, we also get a 'simple combinatorial condition on the cover' characterizing the existence of global sections, like Abramsky and Brandenburger get in the quantum mechanical case (<d-cite key="Abramsky2011"></d-cite>, page 21).


### Local consistency

How would locally consistent data look like? It is a family of PAC learners, one for each finite subsets $C$ of $X$,

$$\lbrace A_C \in P_H (C) : C \in \mathcal{C} \rbrace$$

such that they agree on their overlaps, i.e., for all finite subsets $C$ and $D$ of $X$, we have that, for all training sets $S \in ( (C \cap D) \times 2)^*$, the prediction rules outputted by the learner $A_C$ and $A_D$ coincide on $C \cap D$, i.e.,

$$A_C (S) \restriction C \cap D = A_D (S) \restriction C \cap D $$

So these PAC learners work on different finite areas of the inputs space, and they are all consistent with each other in the sense that they agree in their prediction on overlapping areas.

But can such a family of learner exist? After all, if it does, shouldn't we be able to glue all those local learners together into a global learner, which, however, is impossible once $H$ has infinite VC-dimension?

Nonetheless, here is the plan to construct such a locally consistent family of learners:

1. First, fix an enumeration $C_0 , C_1, C_2, \ldots$ of $\mathcal{C}$ (without any repeats). This of course only works if $X$ is countable, which we assume for now. (If $X$ is uncountable, issues of measurability enter; then we naturally work with a standard Borel space $X$, and choose a countable generating basis $\mathcal{C}$ instead of all finite subsets.) 
Define $D_n := \bigcup_{i = 0}^n C_i$.

2. As an intermediate step, we recursively define, for each $n$, a learner $B_n : (D_n \times 2)^* \to H_{D_n}$ such that 

   - $B_n$ minimizes the empirical risk, i.e., for all $S \in (D_n \times 2)^*$ and $h \in H_{D_n}$, we have $L_S(B_n(S)) \leq L_S(h)$, and
   - $B_n$ is backwards compatible, i.e., for all $m \leq n$, we have $ B_m = B_n \restriction D_m $.

3. Finally, we define, for each $C \in \mathcal{C}$, the learner $A_C := B_n \restriction C_n$, where $n$ is the unique index such that $C = C_n$. It will follow that the $A_C$'s are PAC learners and satisfy the local consistency condition.

{% details Click here for how to complete these steps %}
Let's first check that, once we do step 2, we really get step 3. 
First, to check that each $A_C$ is indeed a PAC-learner, we use the basic fact from statistical learning theory that empirical risk minimizers with a finite hypothesis class are PAC learners for this hypothesis class (see <d-cite key="ShalevShwartz2014"></d-cite>, sec. 4.2). 
To check local consistency, given $C$ and $D$ with, say, $C = C_i$ and $D = C_j$ and $i \leq j$, we have

$$ 
\begin{aligned}
A_C \restriction C \cap D
&=
(B_i \restriction C_i) \restriction C_i \cap C_j
\\
&=
B_i \restriction C_i \cap C_j
\\
&=
(B_j \restriction D_i) \restriction C_i \cap C_j
\\
&=
B_j \restriction C_i \cap C_j
\\
&= 
(B_j \restriction C_j) \restriction C_i \cap C_j
\\
&= 
A_D \restriction C \cap D. 
\end{aligned}
$$



So it remains to do step 2. We construct the desired $B_n$ by recursion. As useful notation, for a function $f : X \to \mathbb{R}$, define

$$ \mathrm{argmin}_{x \in X} f(x) := \lbrace 
x \in X : \forall x' \in X . f(x) \leq f(x') 
\rbrace. $$ 

* For $n = 0$: Given $S \in (D_0 \times 2)^{*}$, choose as $B_0 (S)$ any element of 
  
  $$\mathrm{argmin}_{h \in H_{D_0}} L_S (h),$$
   
  which is nonempty since $H_{D_0}$ is finite.

* For $n + 1$: Given $S \in ( D_{n+1} \times 2)^*$, define the prediction rule $h_S = B_{n+1} (S) \in H_{D_{n+1}}$ as follows:

  * Case 1: $S \not\in ( D_n \times 2)^*$. Then pick any 
    
    $$ h_S \in \mathrm{argmin}_{h \in H_{D_{n+1}}} L_S (h). $$

  * Case 2: $S \in ( D_n \times 2)^*$. Then $B_n (S) \in H_{D_n}$, so there is $\overline{h} \in H$ with $\overline{h} \restriction D_n = B_n (S)$, and we pick $h_S := \overline{h} \restriction D_{n+1}$.


Let's verify that the $B_n$'s have the desired properties, i.e., empirical risk minimization and backwards compatibility. For $n = 0$ this holds by construction. So assume it holds for $B_n$, and we show it for $B_{n+1}$.

We first show backwards compatibility. Let $m \leq n + 1$ and show $ B_m = B_{n+1} \restriction D_m $. Let's first consider the main case where $m = n$. So let $S \in ( D_n \times 2)^{*}$ and show 
$B_n (S) = B_{n+1} (S) \restriction D_n$.
Hence we are in case 2 of the definition of $B_{n+1} (S)$ and, with the notation of this case, we have

$$
B_n (S) =
\overline{h} \restriction D_n =
(\overline{h} \restriction D_{n+1} ) \restriction D_n =
B_{n+1} (S) \restriction D_n.
$$

For the other cases, if $m = n+1$, the claim is trivial, and if $m < n$, then, using the induction hypothesis, 

$$
B_{n+1} \restriction D_m = 
( B_{n+1} \restriction D_n ) \restriction D_m = 
B_n \restriction D_m = 
B_m.
$$


Finally, we check empirical risk minimization. Given $S \in ( D_{n+1} \times 2)^{*}$, we show that 
$h_S = B_{n+1} (S)$ is in 

$$ \mathrm{argmin}_{h \in H_{D_{n+1}}} L_S (h). $$

This holds by construction in case 1, so let's check it in case 2 where $S \in ( D_n \times 2)^*$. To do so, note that for any $C \supseteq \lbrace x : \exists y . (x,y) \in S \rbrace$ we have

$$ L_S (h) = 
\frac{1}{|S|} \big\lbrace
(x,y) \in S : h(x) \neq y 
\big\rbrace =
L_S (h \restriction C).
$$  

Hence, for any $h' \in H_{D_{n+1}}$, we have, since $B_n$ is an empirical risk minimizer by induction hypothesis,

$$\begin{aligned}
L_S (B_{n+1}) 
&= 
L_S (B_{n+1} \restriction D_n) = L_S ( B_n ) 
\\
&\leq 
L_S ( h' \restriction D_n ) =
L_S ( h' ).
\end{aligned}
$$
{% enddetails %}

So we see that PAC-learnability really exhibits contextuality: we can have locally consistent learners covering the whole input space but which cannot be glued together to a single, globally consistent learner. 





## What's next? Cohomology, Compactness, Category


The obvious next step is to apply the rich and general theory of sheaf-theoretic contextuality to the theory of machine learning! Specifically, the next step, like in <d-cite key="Abramsky2015"></d-cite> (especially section 5), is to understand the obstructions we face when wanting to extend a local section to a global one. Here, these are the obstructions that are at play in the No-Free-Lunch theorem when trying to extend a learner working on a local part of the input space to working on the whole input space. The tools of *cohomology* (specifically Čech cohomology) are used for this. But that's for another occasion.


A central theorem of logic is the *compactness* theorem: if every finite subset of a theory has a model, then the whole theory has a model. The contextuality that we have found here describes a sense in which learning is *not* compact: every finite subset of the input space has a PAC learner, but the whole input space does not have a PAC learner. This should be related to discussions about the (non-) compactness of learning (<d-cite key="Asilis2024"></d-cite>).

 
More generally, it would be interesting to see how our discussion connects to other category-theoretic approaches to learners (<d-cite key="Spivak2022"></d-cite>) and machine learning more generally (<d-cite key="Shiebler2021"></d-cite> and [here](https://github.com/bgavran/Category_Theory_Machine_Learning)).




## Conclusion

Sorry, this post got rather long. But let's quickly recap what we've seen. 

We introduced statistical learning theory as the classic theory of machine learning, and we stated the No-Free-Lunch theorem saying that a universal learner cannot exist. Then we introduced sheaf-theoretic contextuality, aiming to apply it to statistical learning theory. 

We defined the learner presheaf and provided a locally consistent family of sections/learners that cannot be globally consistent. This makes precise the starting idea of rethinking the No-Free-Lunch theorem: That we can have learners on the different parts of the input space that cannot be glued to a learner on the whole input space. We ended with some natural questions of where to go from here.




