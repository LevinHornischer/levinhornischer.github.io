---
layout: distill
title: How Random Is the Random Graph?
description: Percolation Theory Meets Algorithmic Randomness
tags: AlgorithmicRandomness Percolation 
giscus_comments: false
date: 2024-09-01
featured: false
citation: true    
bibliography: 2024-09-01-random_graph.bib
 

toc:
  - name: The Random Graph 
  - name: Percolation
  - name: A Crash Course on Algorithmic Randomness
  - name: How random is the random graph?
  - name: Percolation Meets Algorithmic Randomness 
  - name: Next questions
  - name: Conclusion

tikzjax: true  
#Added {% include scripts/tikzjax.liquid %} to distill.liquid
    
---


The random graph is a wonderful object that is studied in graph theory, logic, and probability theory. It is constructed in a probabilistic way as follows. It has countably many vertices and its edges are determined by coin flips: for each (unordered) pair of vertices, you throw a coin; if it lands heads, you put an edge between the vertices, and if it lands tails, you don't. But why—you may ask—do we speak of 'the' random graph? Doesn't this construction yield many different graphs every time you run it? Yes, but it turns out that almost surely—i.e., with probability 1—the obtained graphs are isomorphic (we'll make this more precise below). 


In fact, the random graph is a special case of the models studied in percolation theory (again, below we'll see why). This area of math started with the seminal work of Broadbent and Hammersley in 1957 <d-cite key="Broadbent1957"></d-cite>. It describes, as they put it, ''physical phenomena in which a *fluid* spreads randomly through a *medium*''. Specifically, *percolation* describes those phenomena where the randomness resides in the medium and not in the fluid (like water permeating a porous stone), while in *diffusion* the randomness resides in the fluid and not in the medium (like particles dissolving in water). Beyond porous stones, this is also applied to disordered electrical networks, ferromagnetism, and epidemics. One reason for why this is exciting is that this provides a general model (among others) of how global and macro-level structure can arise from local and micro-level rules.


<div class="fake-img l-body-outset">
<p><em>This local-to-global idea also featured in the <a href="/blog/2024/no_free_lunch">last blog post</a>. It particularly interests me in the case of neural networks: how we might explain their behavior at a global and human-understandable level given only the local and low-level information about the parameters.</em></p>
</div> 


So, both the random graph specifically and percolation theory more generally describe certain random objects (graphs and media) and their properties (isomorphisms and fluid flow). This got me wondering: How does this connect to *the* theory of randomness, namely, algorithmic randomness? Surprisingly, I haven't yet seen these fields interact. (If you know about any references, I'd be very happy to hear about them.) In this post, I'd like to start exploring this a bit.




- [x] The definition of the random graph
- [X] A quick introduction to percolation
- [X] A crash course on algorithmic randomness
- [X] First ideas on how these topics interact
- [X] *Tl;dr*: Algorithmic randomness seems fruitful to analyze percolation 




## The Random Graph


### Tossing coins to build a graph

We start with a countable infinite collection $V$ of vertices which, for simplicity, we identify with the natural numbers $\mathbb{N}$:

<div class="fake-img l-body">
<script type="text/tikz">
\begin{tikzpicture}[dot/.style={draw,circle,minimum size=1.5mm,inner sep=0pt,outer sep=0pt,fill=lightgray}]
  
\node[dot, label={[font=\normalsize,text=gray]below:$0$}] at (0,0)  (v0) {};
\node[dot, label={[font=\normalsize,text=gray]below:$1$}] at (1,0)  (v1) {};
\node[dot, label={[font=\normalsize,text=gray]below:$2$}] at (2,0)  (v2) {};
\node[dot, label={[font=\normalsize,text=gray]below:$3$}] at (3,0)  (v3) {};
\node[dot, label={[font=\normalsize,text=gray]below:$4$}] at (4,0)  (v4) {};
\node[dot, label={[font=\normalsize,text=gray]below:$5$}] at (5,0)  (v5) {};
\node[] at (6,0)  (v6) {$\cdots$};
\end{tikzpicture}
</script>
</div>

The set $E$ of potential edges are all unordered pairs of vertices, i.e., the two-element subsets $\lbrace n, m \rbrace$ of $\mathbb{N}$. We now want to go through the potential edges, and for each edge $e \in E$ throw a coin to determine if it should become an actual edge or not. (Synonymous terminology is that two vertices that have an edge between them are *adjacent*.) 

So let's first fix the order in which we consider the edges, i.e., an enumeration of $E$. There are many enumerations, but a simple one is to go through $m$ and list all the $n < m$. So the order is 
$\lbrace 0, 1 \rbrace$, $\lbrace 0, 2 \rbrace$, $\lbrace 1, 2 \rbrace$, $\lbrace 0, 3 \rbrace$, $\lbrace 1, 3 \rbrace$, etc. For each edge, in this order, we then throw a coin and write $1$ if it lands heads and $0$ if it lands tails. For example, we might record these outcomes:


<div class="fake-img l-body-outset">
<table>
    <tr>
        <td>Edge</td>
        <td>$\lbrace 0,1 \rbrace$</td>
        <td>$\lbrace 0,2 \rbrace$</td>
        <td>$\lbrace 1,2 \rbrace$</td>
        <td>$\lbrace 0,3 \rbrace$</td>
        <td>$\lbrace 1,3 \rbrace$</td>
        <td>$\lbrace 2,3 \rbrace$</td>
        <td>$\lbrace 0,4 \rbrace$</td>
        <td>$\lbrace 1,4 \rbrace$</td>
        <td>$\lbrace 2,4 \rbrace$</td>
        <td>$\lbrace 3,4 \rbrace$</td>
        <td>$\lbrace 0,5 \rbrace$</td>
        <td>$\lbrace 1,5 \rbrace$</td>
        <td>$\lbrace 2,5 \rbrace$</td>
        <td>$\lbrace 3,5 \rbrace$</td>
        <td>$\lbrace 4,5 \rbrace$</td>
        <td>$\cdots$</td>
    </tr>
    <tr>
        <td>Number</td>
        <td>0</td>
        <td>1</td>
        <td>2</td>
        <td>3</td>
        <td>4</td>
        <td>5</td>
        <td>6</td>
        <td>7</td>
        <td>8</td>
        <td>9</td>
        <td>10</td>
        <td>11</td>
        <td>12</td>
        <td>13</td>
        <td>14</td>
        <td>$\cdots$</td>
    </tr>
    <tr>
        <td>Outcome</td>
        <td>1</td>
        <td>0</td>
        <td>0</td>
        <td>1</td>
        <td>0</td>
        <td>1</td>
        <td>1</td>
        <td>0</td>
        <td>1</td>
        <td>1</td>
        <td>0</td>
        <td>1</td>
        <td>0</td>
        <td>0</td>
        <td>1</td>
        <td>$\cdots$</td>
    </tr>
</table>
</div>


This then yields the following graph:

<div class="fake-img l-body">
<script type="text/tikz">
\begin{tikzpicture}[dot/.style={draw,circle,minimum size=1.5mm,inner sep=0pt,outer sep=0pt,fill=lightgray}]
  
\node[dot, label={[font=\normalsize,text=gray]below:$0$}] at (0,0)  (v0) {};
\node[dot, label={[font=\normalsize,text=gray]below:$1$}] at (1,0)  (v1) {};
\node[dot, label={[font=\normalsize,text=gray]below:$2$}] at (2,0)  (v2) {};
\node[dot, label={[font=\normalsize,text=gray]below:$3$}] at (3,0)  (v3) {};
\node[dot, label={[font=\normalsize,text=gray]below:$4$}] at (4,0)  (v4) {};
\node[dot, label={[font=\normalsize,text=gray]below:$5$}] at (5,0)  (v5) {};
\node[] at (6,0)  (v6) {$\cdots$};

\draw (v0) to [bend left] (v1);
\draw (v0) to [bend left] (v3);
\draw (v2) to [bend left] (v3);
\draw (v0) to [bend left] (v4);
\draw (v2) to [bend left] (v4);
\draw (v3) to [bend left] (v4);
\draw (v1) to [bend left] (v5);
\draw (v4) to [bend left] (v5);

\end{tikzpicture}
</script>
</div>

To introduce some notation, we write $2 = \lbrace 0, 1 \rbrace$ and $\omega = \mathbb{N}$. The set of two-element subsets of a set $S$ is often denoted $[S]_2$. Also, one writes $2^\omega$ for the set of functions from $\omega$ to $2$, which we can equivalently think of as infinite binary sequences. Hence each  outcome of coin tosses, like in the above table, is an element $x \in 2^\omega$. We write $\overline{\cdot} : E \to \omega$ for the enumeration function that assigns every potential edge $e$ its number $\overline{e}$.

Thus, we have the graph $(V,E) = (\mathbb{N} , [\mathbb{N}]_2)$ of all potential edges on the given vertices, and each outcome of coin tosses $x \in 2^\omega$ determines a subgraph $G_x = (V, E_x)$ of $(V,E)$. The graph $G_x$ has the same vertices but only selects the following set of edges as 'actual'
\begin{aligned}
E_x := \lbrace
e \in E : x ( \overline{e} ) = 1 
\rbrace.
\end{aligned}


### Almost surely getting isomorphic graphs

But why should the graphs constructed in this way be isomorphic with probability 1, and what does that even mean exactly?   

Graph isomorphism is readily defined: Two graphs $(V,E)$ and $(V',E')$ are *isomorphic* if there is a bijective function $f : V \to V'$ such that for all vertices $v,v' \in V$, they are adjacent in $(V,E)$ if and only if $f(v)$ and $f(v')$ are adjacent in $(V',E')$. We then also write $(V,E) \cong (V',E')$.

To make the probabilistic claim precise, we need to put a probability measure on the space $2^\omega = \prod_\omega 2$ of infinite coin toss outcomes. This is a standard thing to do: $2^\omega$ is known as the *Cantor space*. It is a 'space' because it naturally has a topology: namely, the product topology when viewing $2 = \lbrace 0 , 1 \rbrace$ as the discrete space with two points. This topology is generated by the *cylinder sets*, i.e., sets of the form $C = \prod_\omega C_n$ where only finitely many $C_{n_1} , \ldots , C_{n_k}$ are proper subsets of $2$ while the other $C_n$ are identical to $2$. Thus, the cylinder set $C$ describes the event 'for $i = 1 , \ldots , k$, the outcome of the $n_i$-th coin toss was in $C_{n_i}$'. If we use a fair coin, the probability measure for an individual coin toss is given by the Bernoulli measure: $\mu(\lbrace 1 \rbrace) = \frac{1}{2}$ and $\mu(\lbrace 0 \rbrace) = \frac{1}{2}$, and of course $\mu(\lbrace 0,1 \rbrace) = 1$ and $\mu(\emptyset) = 0$. So the probability measure of a cylinder set $C$ is $\prod_{i = 1}^k \mu (C_{n_i})$. If $C$ is non-empty, each $C_{n_i}$ is a singleton (since it is neither empty nor $2$), hence this number simply is $(\frac{1}{2})^k = 2^{-k}$. It now can be shown that this measure uniquely extends to a probability measure on the $\sigma$-algebra generated by the cylinder sets. This probability measure is known as the *uniform* or *Lebesgue* measure $\lambda$ on $2^\omega$. As we will see, this is also the standard setting for algorithmic randomness.


Now we can make precise the claim that the event that two coin toss outcomes $x$ and $y$ yield isomorphic graphs $G_x$ and $G_y$ has probability $1$. The straightforward way would be to say that the set $\lbrace (x,y) : G_x \cong G_y \rbrace$ has measure $1$ in the product space $2^\omega \times 2^\omega$. But we can do even better. We can show that there must be a specific graph $R$ such that with probability 1 the coin toss outcome $x$ yields a graph $G_x$ that is isomorphic to $R$.
  
**Theorem** (Erdős and Rényi 1963). There is a graph $R$ such that $\lambda ( \lbrace x \in 2^\omega : G_x \cong R \rbrace ) = 1$. 

Hence we can speak of *the* random graph $R$: Almost surely and up to isomorphism, our graph construction process always yields the same outcome. This may well seem counterintuitive: a random process that yields a predictable outcome (and we will come back to this). 

{% details Click for a proof sketch following Cameron <d-cite key="Cameron2013a"></d-cite>%}
The key idea is to consider the following graph-theoretic property:

* Whenever $X$ and $Y$ are two disjoint finite sets of vertices, there is a vertex which is neither in $X$ nor in $Y$ but which is adjacent to all vertices in $X$ and no vertices in $Y$.

Let's say a graph $(V,E)$ is *interpolatable* if it has this property.<d-footnote>Neither Cameron <d-cite key="Cameron2013a"></d-cite> nor Hodges <d-cite key="Hodges1993"></d-cite> give a name to this property, but Spencer <d-cite key="Spencer2001"></d-cite> does on pages 5--6 and calls it 'the Alice's restaurant property'. Spencer says Peter Winkler first used the term referring to <a href="https://www.youtube.com/watch?v=m57gzA2JCcM" target="_blank">this song</a> of Arlo Guthrie, with the chorus 'you can get anything you want at Alice's restaurant'. Since 'interpolatable' is a bit shorter, I'll stick with that.</d-footnote> Now the theorem will follow once we have the next two lemmas: 

1. $\lambda(\lbrace x \in 2^\omega : G_x \text{ is interpolatable} \rbrace) = 1$. This is because, for fixed $X,Y \subseteq V$, as we consider more and more vertices $n$ not in $X \cup Y$, it gets more and more likely that we find one $n$ for which we threw the coins in such a way that $n$ is adjacent to all of $X$ and none of $Y$. We give a precise argument for a stronger claim below.  
2. Any two countable interpolatable graphs are isomorphic. Here one gives a typical back-and-forth argument from model theory. The idea is this: Assume we already have constructed a partial isomorphism $f : G \to G'$ for vertices $v_0 , \ldots , v_{n-1}$ of $G$, and we want to extend it to include vertex $v_{n}$. We need to map $v_{n}$ in $G$ to a vertex in $G'$ with the same connectivity. To describe the connectivity of $v_n$ in $G$, divide $\lbrace v_0 , \ldots , v_{n-1} \rbrace$ into the subset $X$ of vertices adjacent to $v_n$ and the subset $Y$ of vertices not adjacent to $x$. By interpolatability, we can find a vertex $z$ in $G'$ with the same connectivity: i.e., that is adjacent to all vertices in $f(X)$ and no vertices in $f(Y)$. So we map $v_n$ to $z$ (in fact, we pick the $z$ with a minimal index in an enumeration of the vertices of $G'$). That was the 'forth'-direction. To get a bijection, i.e., to find a preimage of each vertex in $G'$, we apply the same idea in the 'back'-direction.

       
Now, by the first claim, since sets of full measure are nonempty, there in particular is an $x_0$ such that $G_{x_0}$ is interpolatable, and we set $R := G_{x_0}$. (Yes, this is a non-constructive existence proof, but we'll see Rado's neat explicit definition of an interpolatable graph below.) By the second claim, once $G_x$ is interpolatable, it is isomorphic to $R$, so

$$
1 = \lambda ( \lbrace x : G_x \text{ is interpolatable} \rbrace) \leq \lambda ( \lbrace x : G_x \cong R \rbrace ),
$$
as needed.
{% enddetails %}

This proof is the quick route to the result. But the common route is to construct the random graph $R$ as a Fraïssé limit (see, e.g., Hodges <d-cite key="Hodges1993"></d-cite> section 7.4). This yields many more stunning properties of the random graph: it is universal (every finite graph embeds into it), it is homogeneous (every isomorphism between finitely generated subgraphs extend to an automorphism), its theory is $\omega$-categorical and has quantifier elimination, etc. It also has a category-theoretic treatment: see e.g. [here](https://golem.ph.utexas.edu/category/2009/11/fraisse_limits.html) or [here](https://www.cambridge.org/core/journals/mathematical-structures-in-computer-science/article/abs/universal-domains-and-the-amalgamation-property/F11B604F6A928367A982F32C75E3C537) or [here](https://www.sciencedirect.com/science/article/pii/S0168007214000773?via%3Dihub). Fraïssé's construction is very beautiful, but something for another post.


### Bonus section: The strange logic of random graphs

To still give a flavor of the stunning results around the random graph, let me add one more theorem. Strictly speaking, it is not necessary for the remainder, but it answers the following natural question. Now we know that almost every graph has the property of being isomorphic to the random graph, but what about other properties? Couldn't there be properties that, say, half of the graphs have and the other half don't? Surprisingly, the theorem says: no, every property either is had by almost all graphs or by almost no graphs! (In probability theory, such statements are known as 0--1 laws.)

To be precise, though, we need to define what we mean by 'property'. A natural choice is: any property that can be formulated in the first-order language of graphs, i.e., the language with just one binary relation symbol $R$. The intuitive interpretation is that the variables of the language range over vertices and the relation symbol $R$ describes edge relations. Examples are:

* Being irreflexive: $\forall x (\neg Rxx)$ 
* Being symmetric: $\forall x \forall y (Rxy \to Ryx)$ 
* Having a loop of length 3: $\exists x \exists y \exists z ( \neg x = y \wedge \neg x = z \wedge \neg y = z \wedge Rxy \wedge Ryz \wedge Rzx)$.

If a graph $G$ has the property described by a sentence $\varphi$ of the first-order language of graphs, we write $G \models \varphi$. Then the theorem says:


**Theorem** (Fagin 1976 and Glebskii, Kogan, Liogonkii & Talanov 1969).
Let $\varphi$ be a sentence of the first-order language of graphs. Then 

$$
\lambda ( \lbrace x \in 2^\omega : G_x \models \varphi \rbrace )
$$

is either $0$ or $1$.<d-footnote>The theorem is usually formulated in terms of <em>asymptotically almost sure</em>: the ratio among the $n$-element graphs that have property $\varphi$ converges either to $0$ or to $1$ as $n$ goes to infinity. To avoid further terminology, I formulate the theorem here in terms of the product measure that we already have introduced. The difference in the proof is that then we do not need to refer to the compactness theorem anymore. I wonder if this has to do with the fact that compactness is built into the product measure by demanding that cylinder sets only have a non-trivial projection at finitely many indices (much like Tychonoff's theorem says that compactness is 'built into' the product topology).</d-footnote>

 


{% details Click for a proof sketch following Fagin/Spencer <d-cite key="Spencer2001"></d-cite>%}
We assume some basic first-order logic terminology. Let $L$ be the first-order language of graphs. 
Consider the $L$-theory $T$ which contains:

* the two sentences $\varphi_\mathrm{irr}$ and $\varphi_\mathrm{sym}$ from above expressing irreflexivity and symmetry, respectively, and
* for each $r,s \geq 0$ the sentence $\varphi_{r,s}$ which says that for all distinct $x_1, \ldots, x_r$ and $y_1, \ldots y_s$ there is a distinct $z$ such that $Rzx_i$ for all $1 \leq i \leq r$ and $\neg Rzy_j$ for all $1 \leq j \leq s$.

Hence, for any $L$-structure $G$, we have: $G \models T$ iff $G$ is an interpolatable graph.

We show that $T$ is a complete $L$-theory, i.e., for any $L$-sentence $\varphi$, either $T \models \varphi$ or $T \models \neg \varphi$. Indeed, otherwise there is a sentence $\varphi$ such that both $T_1 := T \cup \lbrace \neg \varphi \rbrace$ and $T_2 := T \cup \lbrace \varphi \rbrace$ have models. Note that any such model must be infinite because for any distinct elements $x_1 , \ldots, x_n$, there must be another element $z$ distinct from them. By downward Löwenheim-Skolem, we can also find countable models $G_1 \models T_1$ and $G_2 \models T_2$. But since these models are countable interpolatable graphs, they must be isomorphic, which contradicts that one makes true $\varphi$ and the other doesn't. (That's essentially the [Łoś–Vaught test](https://en.wikipedia.org/wiki/%C5%81o%C5%9B%E2%80%93Vaught_test).)

Now let's consider the given $L$-sentence $\varphi$. So either $T \models \varphi$ or $T \models \neg \varphi$. If $T \models \varphi$, then, for any graph $G$, we have the implications '$G$ interpolatable $\Leftrightarrow$ $G \models T$ $\Rightarrow$ $G \models \varphi$', so

$$
\begin{aligned}
1 &= \lambda ( \lbrace x : G_x \text{ interpolatable} \rbrace ) \\
&= \lambda ( \lbrace x : G_x \models T \rbrace ) \\
&\leq \lambda ( \lbrace x : G_x \models \varphi \rbrace ) 
\end{aligned}
$$

If $T \models \neg \varphi$, then the same reasoning, but replacing $\varphi$ by $\neg \varphi$, yields 
$\lambda ( \lbrace x : G_x \models \neg \varphi \rbrace ) = 1$, 
hence, for the complement, we have
$\lambda ( \lbrace x  : G_x \models \varphi \rbrace ) = 0$.
{% enddetails %}

For much more on this, see the book 'The Strange Logic of Random Graphs' by Spencer <d-cite key="Spencer2001"></d-cite>.




## Percolation


The general definition of a  *percolation model* is as a probability distribution on percolation configurations on an unordered graph $G = (V,E)$, where a percolation configuration $x$ is a function from $E$ to $\lbrace 0 , 1 \rbrace$ (see Duminil-Copin <d-cite key="DuminilCopin2018"></d-cite>). For an edge $e \in E$, if $x(e) = 1$ one says the edge is *open* in configuration $e$, and *closed* otherwise.  

Thus, our random graph setting is a percolation model:

* The graph $G$ has vertex set $V = \mathbb{N}$ with all possible edges, i.e., $E$ is the set of all two-elements subsets of $V$.  
* The percolation configurations $E \to \lbrace 0,1 \rbrace$ bijectively correspond, via our enumeration $E \to \omega$, to the elements of $2^\omega$.
* The probability distribution also is given via this bijective correspondence to the Lebesgue measure on Cantor space $2^\omega$. Thus, an edge is open with probability $p = \frac{1}{2}$.

But among the most studied percolation models is 'bond percolation on the square lattice' where

* The graph $G$ has as vertices pairs of integers, i.e., $V = \mathbb{Z}^2$, and there is an edge between $(a,b)$ and $(c,d)$ iff the (Euclidean) distance between them is $1$, i.e., $\vert a-c \vert + \vert b-d \vert = 1$.
* The configurations again are the functions $E \to 2$; which, by fixing an enumeration of $E$, again bijectively correspond to the elements of $2^\omega$. 
* The probability distribution is given with respect to the Bernoulli parameter $0 \leq p \leq 1$: an edge's probability of being open is $p$ and of being closed is $1 - p$, i.e., we don't necessarily use a fair coin. This again extends to a probability measure $\mu_p$ on all configurations $2^E$ by taking cylinder sets and the product measure of these Bernoulli measures. 

We can visualize this as follows: We draw the two-dimensional grid $\mathbb{Z}^2$, whose origin we mark by $o = (0,0)$. For each potential edge (i.e., vertices with distance $1$), we throw a coin with bias $p$ to determine if the edge is open (in which case we draw an edge) or closed (in which case we don't draw an edge). One possible outcome---i.e., one configuration---looks like this:

<script type="text/tikz">
\begin{tikzpicture}[dot/.style={draw,circle,minimum size=1.5mm,inner sep=0pt,outer sep=0pt,fill=lightgray}]
  
\node[dot] at (0,0)  (00) {};
\node[dot] at (1,0)  (10) {};
\node[dot] at (2,0)  (20) {};
\node[dot] at (3,0)  (30) {};
\node[dot] at (4,0)  (40) {};
\node[dot] at (5,0)  (50) {};
\node[dot] at (6,0)  (60) {};

\node[dot] at (0,1)  (01) {};
\node[dot] at (1,1)  (11) {};
\node[dot] at (2,1)  (21) {};
\node[dot] at (3,1)  (31) {};
\node[dot] at (4,1)  (41) {};
\node[dot] at (5,1)  (51) {};
\node[dot] at (6,1)  (61) {};

\node[dot] at (0,2)  (02) {};
\node[dot] at (1,2)  (12) {};
\node[dot] at (2,2)  (22) {};
\node[dot, label={[font=\normalsize,text=gray]30:$o$}] at (3,2)  (32) {};
\node[dot] at (4,2)  (42) {};
\node[dot] at (5,2)  (52) {};
\node[dot] at (6,2)  (62) {};

\node[dot] at (0,3)  (03) {};
\node[dot] at (1,3)  (13) {};
\node[dot] at (2,3)  (23) {};
\node[dot] at (3,3)  (33) {};
\node[dot] at (4,3)  (43) {};
\node[dot] at (5,3)  (53) {};
\node[dot] at (6,3)  (63) {};

\node[dot] at (0,4)  (04) {};
\node[dot] at (1,4)  (14) {};
\node[dot] at (2,4)  (24) {};
\node[dot] at (3,4)  (34) {};
\node[dot] at (4,4)  (44) {};
\node[dot] at (5,4)  (54) {};
\node[dot] at (6,4)  (64) {};

\node[] at (-1,2) {$\cdots$};
\node[] at (7,2) {$\cdots$};

\draw[-] (00)--(01);
\draw[-] (00)--(10);
\draw[-] (10)--(20);
\draw[-] (30)--(31);
\draw[-] (40)--(50);
\draw[-] (60)--(61);

\draw[-] (01)--(02);
\draw[-] (11)--(21);
\draw[-] (21)--(22);
\draw[-] (21)--(31);
\draw[-] (41)--(42);
\draw[-] (51)--(52);
\draw[-] (51)--(61);

\draw[-] (02)--(12);
\draw[-] (12)--(13);
\draw[-] (22)--(23);
\draw[-] (22)--(32);
\draw[-] (21)--(31);
\draw[-] (42)--(42);
\draw[-] (52)--(53);

\draw[-] (13)--(14);
\draw[-] (23)--(33);
\draw[-] (33)--(43);
\draw[-] (53)--(63);
\draw[-] (53)--(54);
\draw[-] (63)--(64);


\end{tikzpicture}
</script>

As mentioned in the introduction, one thinks of the subgraph of the grid that is determined by a configuration as describing a porous medium. An open edge describes a 'cavity' in the medium. Then one considers how a fluid spreads through it, i.e., which connected components there are in the configuration. (Hence, randomness resides in the medium and the fluid flows deterministically.) Thus, to understand the qualitative behavior of the medium, the natural question is to understand when there are large clusters. As Grimmett <d-cite key="Grimmett1999"></d-cite> puts it on page 1:

> Suppose we immerse a large porous stone in a bucket of water. What is the probability that the centre of the stone is wetted? [...] It is not unreasonable to postulate that the fine structure of the interior passageways of the stone is on a scale which is negligible when compared with the overall size of the stone. In such circumstances, the probability that a vertex near the centre of the stone is wetted by water permeating into the stone from its surface will behave rather similarly to the probability that this vertex is the endvertex of an infinite path of open edges in $\mathbb{Z}^2$. That is to say, the large-scale penetration of the stone by water is related to the existence of infinite connected clusters of open edges.


The exciting discovery about this model is that there is a clear *phase transition* when varying the parameter $p$. This varying of the parameter is when percolation theory gets really interesting. For small values of $p$ (i.e., close to $0$), almost surely all clusters are finite, while for large values of $p$ (i.e., close to $1$), almost surely there are infinite clusters. So there is a critical value $p_c$ such that for $p < p_c$ infinite clusters almost surely do not exist and for $p > p_c$ they almost surely exist. The celebrated theorem of Kesten shows that, in fact, $p_c = \frac{1}{2}$.

This is the sense mentioned in the introduction in which global, qualitative, macro-level behavior (existence of infinite clusters or permeability) arises from local, quantitative, micro-level rules (open edges in the configuration or porosity). For more on the topic, see, e.g., <d-cite key="DuminilCopin2017"></d-cite><d-cite key="Grimmett1999"></d-cite><d-cite key="Bollobas2006"></d-cite> or [this great introductory video](https://www.youtube.com/watch?v=a-767WnbaCQ).







## A Crash Course on Algorithmic Randomness

Let's now see how these random objects---the random graph or the porous stone---connect to the theory of algorithmic randomness. For a quick introduction to this exciting field, take a look at the introduction of the standard book by Downey and Hirschfeldt <d-cite key="Downey2010"></d-cite>. Here let's limit us to a very concise crash course. 
 
### Intuition
 
In the last 100 years or so, algorithmic randomness developed a rigorous formalization and investigation of the concept of randomness. It describes what it means that an individual mathematical object---like an infinite binary sequence---is random. After all, why is it that a sequence like
\begin{aligned}
1001110100101101010111010101\ldots
\end{aligned}
is (or at least appears) random while 
\begin{aligned}
1001001001001001001001001001\ldots
\end{aligned}
is not? There are several formal notions of randomness, with interesting relationships, but arguably the most well-known one is Martin-Löf randomness. It unites three intuitions about what it means for an infinite binary sequence to be random:

1. Unpredictability: We should not be able to predict the next bit in a random sequence when knowing all preceding bits.
2. Typicality: A random sequence should not have any rare properties. In other words, they should pass all effective statistical tests (we'll make this precise soon).
3. Incompressability: The initial segments of a random sequence are incompressible, i.e., there is no more effective method of producing them than just listing them. (The second, non-random sequence above is effectively generated by the rule 'repeat the string 100 forever'.) 

Here we will focus on formalizing the typicality intuition, which is also what Martin-Löf originally did. As mentioned, algorithmic randomness takes place---not exclusively, but primarily---in the setting of Cantor space $2^\omega$ (which we have seen before), i.e., the set of infinite binary sequences. The formal definition hence will be of the following form: 'an infinite binary sequence $x \in 2^\omega$ is Martin-Löf random iff ...'. To fill in the dots, we need a sequence of preliminary definitions.


### Definition

By 'sequence' we mean an infinite sequence and by 'string' we mean a finite sequence. If $\sigma$ is a binary string, $[\sigma]$ is defined as the set of all $x \in 2^\omega$ that extend $\sigma$. In other words, this is the cylinder set $\prod_n C_n$ with $C_n = \lbrace \sigma(n) \rbrace$ if $\sigma(n)$ is still defined and otherwise $C_n = 2$. If $A$ is a set of binary strings, we write $[A] := \bigcup_{\sigma \in A} [\sigma]$. 

The open sets of Cantor space are unions of cylinder sets, so $[A]$ is an open set. We now want to define when it is 'effectively open', i.e., when an algorithm can 'produce' $A$.

If $A$ is a set of binary strings, we say $A$ is *computably enumerable* if there is an algorithm that enumerates $A$. That is to say, as time progresses, the algorithm produces more and more outputs $a_0 , a_1, a_2, \ldots$ which are all elements of $A$ and every element of $A$ is outputted at least once. So, for a given string $\sigma$, if $\sigma$ is in $A$, it will eventually be outputted by the algorithm, but we don't know in advance how long we have to wait for this to happen. Hence we, for a given string, we cannot *decide* if it is in $A$. If we have such a stronger algorithm that decides membership in $A$ (i.e., given a string as input it produces $1$ or $0$ as output depending on whether the string is or is not in $A$), then we say $A$ is *computable*. But here we will only use the weaker notion of computable enumerability. 

We say that a sequence $A_0 , A_1, A_2, \ldots$ of sets of binary strings is *uniformly computably enumerable* if there is an algorithm that, when given the number $n$ as input, starts enumerating the set $A_n$. Hence not only is each $A_n$ computalby enumerable, but we have a single algorithm that can do all the enumerating. So, as usual, the 'uniform' indicates that we not only have an $\forall \exists$-quantifier, but an $\exists \forall$-quantifier: it is not just the case that for all $n$, there is an algorithm that enumerates $A_n$, but there is an algorithm such that, for all $n$, it enumerates $A_n$. 

Now we say that a set $U \subseteq 2^\omega$ is *effectively open* if there is a computably enumerable set $A$ of binary strings such that $U = [A]$. A sequence $U_0, U_1, U_2, \ldots$ of subsets of $2^\omega$ is *uniformly effectively open* if there is a uniformly computably enumerable sequence $A_0 , A_1, A_2, \ldots$ of sets of binary strings such that, for all $n$, we have $U_n = [A_n]$.

A *Martin-Löf test* is a uniformly effectively open sequence $(U_n)$ such that, for all $n$, we have $\lambda (U_n) \leq 2^{-n}$. With that, we can finally define:

**Definition** (Martin-Löf 1966). A sequence $x \in 2^\omega$ is *Martin-Löf random* if it passes every Martin-Löf test, i.e., for every Martin-Löf test $(U_n)$, we have $x \not\in \bigcap_n U_n$. 

How does this capture the typicality intuition? The idea is that a Martin-Löf test $(U_n)$ explicates the notion of a performable statistical test for outliers: If we have a given sample $x \in 2^\omega$, we test whether $x \in P := \bigcap_n U_n$, i.e., whether $x$ has the rare property $P$. In other words, we test whether $x$ is an outlier in that it has the rare or atypical property $P$. 

* That the test is performable is explicated as $(U_n)$ being uniformly effectively open: If $x$ has the rare property $P$, we will, for any $n$, eventually know that $x$ is in $U_n$, by checking whether our $x$ has as initial segment one of the binary strings generating $U_n$.<d-footnote>This is some form of falsifiability. Though, usually falsifiability is explicated as the property in question being a closed set.</d-footnote> 
* That the property $P$ is rare is explicated as it not only having probability $0$, but its probability converges to $0$ *quickly* or *effectively*, because the probability of the $U_n$ is bounded by $2^{-n}$. 

Of course, other explications of performability and rarity are possible, which may or may not lead to different notions of randomness. Here are two examples that we will use later on.

1. A *Solovay test* is defined just like a Martin-Löf test, but we only require $\sum_n \lambda (U_n) < \infty$ rather than $\lambda (U_n) \leq 2^{-n}$. It can be shown that this doesn't change the notion of randomness: $x$ is Martin-Löf random iff it passes every Solovay test.

2. A *Schnorr test* is also defined just like Martin-Löf test, but we require $\lambda (U_n) = 2^{-n}$ rather than only $\lambda (U_n) \leq 2^{-n}$. (Or, rather, we require $(\lambda (U_n))$ to be uniformly computable, but this yields the same notion of randomness, so we skip the definition of uniform computability of reals.) This does, however, change the notion of randomness: If we define $x \in 2^\omega$ to be *Schnorr random* if $x$ passes all Schnorr tests, then being Martin-Löf random clearly implies being Schnorr random, but the converse need not be true.  

As a final comment, the typicality intuition offers an important perspective on the field of algorithmic randomness. While probability theory 'only' provides qualitative results of the form that almost surely a given object has a desired property, algorithmic randomness provides a further quantitative analysis of such results by describing 'how typical' the object $x$ has to be for it to have the desired property. For more on this, see, e.g., <d-cite key="Rute2019"></d-cite>.






## How random is the random graph?

Now that we have the tools of algorithmic randomness available, we can revisit the random graph. Conveniently, it already 'lives' in the same setting as algorithmic randomness, i.e., Cantor space. So we can straightforwardly investigate how random it is. 


### Schnorr Randomness is Sufficient
 
 
To recall, we considered the graph $G = (V,E)$ whose vertices are the natural numbers and which has all unordered edges. A sequence $x \in 2^\omega$ of coin toss outcomes selects a subgraph $G_x$ of $G$ by selecting an edge $e$ iff the $\overline{e}$-th coin toss was $1$, where $\overline{\cdot} : E \to \omega$ is our fixed (computable) enumeration of edges.


**Theorem**. Let $x \in 2^\omega$ be Schnorr random. Then the graph $G_x$ is isomorphic to the random graph.


{% details Click for the proof %}
We show that $G_x$ is interpolatable. So let $X = \lbrace i_0, \ldots , i_{r-1} \rbrace$ and $Y = \lbrace j_0, \ldots , j_{s-1} \rbrace$ be two finite disjoint subsets of $\mathbb{N}$. We need to find a vertex $k$ not in $X \cup Y$ that is adjacent to all vertices in $X$ and no vertices in $Y$. 
So let's say *$x$ fits $X$ and $Y$ at $k$* if $k \not\in X \cup Y$ and

1. for all $i \in X$, we have $x ( \overline{\lbrace i , k \rbrace } ) = 1$, and
2. for all $j \in Y$, we have $x ( \overline{\lbrace j , k \rbrace } ) = 0$.

We will define an appropriate Schnorr test $(U_n)$ such that $x \not\in \bigcup_n U_n$ implies the existence of the desired vertex $k$.

To define the Schnorr test, let $N := \max ( X \cup Y ) + 1$. For $n \geq 0$, define $V_n$ as the set of those $x \in 2^\omega$ for which there is no $k \in \lbrace N , \ldots ,  N + n \rbrace$ such that $x$ fit $X$ and $Y$ at $k$.
Set
\begin{aligned}
q 
:= 
\frac{2^{r + s} - 1}{2^{r + s}},
\end{aligned}
and define $f : \mathbb{N} \to \mathbb{N}$ by mapping $n$ to the smallest $m$ such that $q^m \leq 2^{-n}$.
Finally, define, for $n \geq 0$, 
\begin{aligned}
U_n 
:=
V_{f(n)}.
\end{aligned} 

Show that $(U_n)$ is a Schnorr test will finish the proof: Then, since $x$ is Schnorr random, we have $x \not\in \bigcap_n U_n$, so there is $n$ such that $x \not\in V_{f(n)}$. Hence there is $k \in \{ N , \ldots , N + f(n)\}$ such that $x$ fits $X$ and $Y$ at $k$, as needed.


As useful notation, for $k \in \mathbb{N} \setminus (X \cup Y)$ and $\sigma$ a binary string of length $r + s$, write $W(k,\sigma)$ for the set of those $x \in 2^\omega$ such that 

* $x( \overline{ \lbrace i_0 , k \rbrace } ) = \sigma (0)$, 
* $\ldots$,
* $x( \overline{ \lbrace i_{r-1} , k \rbrace} ) = \sigma (r-1)$,
* $x( \overline{ \lbrace j_0 , k \rbrace} ) = \sigma (r)$,
* $\ldots$, and 
* $x(\overline{ \lbrace j_{s-1} , k \rbrace }) = \sigma (r+s-1)$.

Thus, $x \in 2^\omega$ fits $X$ and $Y$ at $k \in \mathbb{N} \setminus (X \cup Y)$ precisely if $x \in W(k,\rho)$ where

$$
\begin{aligned}
\rho 
= 
\underbrace{1 \ldots 1}_{r \text{ times}}
\underbrace{0 \ldots 0}_{s \text{ times}}.
\end{aligned}
$$

Hence, writing $S$ for the set of all binary strings of length $r + s$ except for $\rho$, we have

$$
\begin{aligned}
V_n
= 
\bigcup_{(\sigma_0 , \ldots , \sigma_{n-1}) \in S^n }
\bigcap_{k = N}^{N + n}
W(k,\sigma_k).
\end{aligned}

Since the set $W(k,\sigma_k)$ is a cylinder set, also the intersection $\bigcap_{k = N}^{N + n} W(k,\sigma_k)$ is. So $V_n$ even is a finite union of cylinder sets, which we can compute uniformly in $n$, so $(V_n)$ is uniformly effectively open.  
 
Turning to the measures, the cylinder set $W(k,\sigma_k)$ places a constraint at $r + s$ many positions, and the intersection $\bigcap_{k = N}^{N + n} W(k,\sigma_k)$ at $n (r + s)$ many positions. So the intersection has measure $(2^{-(r + s)})^n$. Since these intersections are disjoint for different elements of $S^n$, we have
\begin{aligned}
\lambda (V_n) 
=
\sum_{S^n}
(2^{-(r + s)})^n
=
|S|^n  (2^{r + s})^{-n}
=
\frac{(2^{r+s} - 1)^n}{(2^{r + s})^{n}}
= q^n.
\end{aligned}

Finally, since $f$ is computable, the $U_n$'s also are uniformly effectively open, and their measure $\lambda(U_n) = q^{f(n)} \leq 2^{-n}$ is uniformly computable. So $(U_n)$ indeed is a Schnorr test.
{% enddetails %}


Thus, Schnorr randomness (and in particular Martin-Löf randomness) is enough typicality to ensure that the selected graph is the random graph. But as we'll see next, we cannot have the converse.


### But Randomness is Not Necessary

As came up in conversation with Michiel van Lambalgen, there is no hope that this is a characterization of Schnorr randomness, or *any* randomness notion, because there can be computable $x$ (which hence are not random in any sense) such that $G_x$ is random. 

This is because the random graph is characterized by being a countable interpolatable graph, but there also are completely deterministic ways of building such a graph. Indeed, Rado <d-cite key="Rado1964"></d-cite> constructed the random graph as follows: The set of vertices is $\mathbb{N}$ and there is an edge between $i$ and $j$ iff the $i$-th bit of $j$, when written as a binary number, is $1$.
Thus, consider the $x \in 2^\omega$ defined by $x(n) = 1$ iff for the pair $\lbrace i,j \rbrace$ with $\overline{\lbrace i,j \rbrace} = n$ we have that the $i$-th bit of $j$ in binary is $1$. Then $G_x$ is Rado's random graph but $x$ is computable. 

Ultimately this reflects what we said right after introducing the random graph: even though the process generating this graph is random (hence Schnorr randomness is sufficient), the graph itself is deterministic (hence it is not necessary). 


## Percolation Meets Algorithmic Randomness

So far, we considered one percolation model (namely, the setting of the random graph) and one almost sure property of the percolation configurations (namely, being the random graph). We saw that if the percolation configuration was typical (in the sense of being Schnorr random), then it has the almost sure property, but the converse needn't be the case.

Now it is only natural to consider other percolation models with their configurations, as well as other almost sure properties, and maybe also other notions of typicality/randomness. An obvious choice is the 'bond percolation on the square lattice' model and the property of having (or avoiding) an infinite cluster---since we have seen that they play a central role in percolation theory. 

I haven't worked this out yet, but I imagine it could go like this. Let's consider the subcritical regime, i.e., a bond percolation on the square lattice with bias $p < p_c = \frac{1}{2}$. Instead of infinite clusters, we can also consider the closely related property of the origin $o$ being connected to infinity, i.e., there being an infinite simple path starting at $o$. (This property has positive probability iff the property of there being an infinite cluster has probability 1.) 
A classic result, reviewed by Duminil-Copin <d-cite key="DuminilCopin2017"></d-cite> on page 3, is that in this subcritical regime, there is a constant $c > 0$ such that the probability of $o$ being connected to distance $n$ is $\leq \mathrm{exp}(-cn)$. This lends itself to a Solovay test $(U_n)$ with $U_n$ containing those $x$ where in $G_x$ the origin is connected to distance $n$. This suggests to conjecture that if $x$ is Martin-Löf random, then $G_x$ in the subcritical regime does not have any infinite clusters. What about the converse here? 

And what about the supercritical regime and the property of having an infinite cluster? Duminil-Copin <d-cite key="DuminilCopin2017"></d-cite> highly praises a classic result of [Burton and Keane](https://link.springer.com/article/10.1007/BF01217735) showing that then this cluster is unique. The argument uses ergodic theory, and [Bienvenu, Day, Mezhirov and Shen](https://link.springer.com/chapter/10.1007/978-3-642-13962-8_6) provide ergodic-type characterizations of algorithmic randomness.



These questions about applying algorithmic randomness to percolation seem very interesting because it is not just any old 'almost sure' result that is being analyzed. It is a result about a statistical process where many intuitions about randomness play a key role. After all, randomly selecting a subsequence of a given sequence was central to von Mises' first formalization and analysis of randomness. Whether these intuitions can be pumped to be added to the trinity of 'unpredictable', 'typical', and 'incompressible' remains to be seen.
 




## Next questions


1. Extend the above first ideas and analyze 'almost sure' results in percolation theory in terms of algorithmic randomness: which notion of randomness they require and maybe even characterize. 

2. What changes if we built the random graph or the bond percolation on the square lattice when each edge is determined by different, but still independent Bernoulli processes, e.g., coins with different biases. Then [Kakutani's theorem](https://www.jstor.org/stable/1969123?origin=crossref) (the one in measure theory, not the fixed point theorem) applies. It provides a simple criterion for when two product measures are mutually singular or equivalent (see section 10, page 222 of the linked paper). This theorem also plays a crucial role in van Lambalgen's proof of Ville's theorem in algorithmic randomness (<d-cite key="Lambalgen1987"></d-cite>, theorem 4.6.1).

3. Modal logic can be seen as the logic of (directed) graphs: How does it connect to percolation theory?   

4. We've seen that percolation is in a sense dual to diffusion: the medium is random rather than the fluid. Now, diffusion famously inspired machine learning. Can percolation also be fruitfully applied to neural networks? After all, activation deterministically flows through the network which was built using a stochastic process (namely, stochastic gradient descent).




## Conclusion

We introduced the random graph and percolation theory in order to see how they can be studied from the point of view of algorithmic randomness. This posed more questions than we saw answers, but algorithmic randomness did seem fruitful to analyze percolation. 




