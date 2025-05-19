# COWSolver

## 1. Setting up the orders

Imagine during one batch you collect:

* A list of **bids** (buys):

![image](https://github.com/user-attachments/assets/56a34998-21bc-4f65-8a38-224c5868ae74)

  where $p_i$ is the limit price of bid $i$, and $q_i$ its quantity.

* A list of **asks** (sells):

![image](https://github.com/user-attachments/assets/6df47ef9-df08-4e50-b468-728be5615163)

  where $p_j$ is the limit price of ask $j$, and $q_j$ its quantity.

> **Notation note:**
>
> * Parentheses $(p_i, q_i)$ just bundle the two numbers together.
> * Curly braces $\{\dots\}$ denote a set (or list) of those pairs.

---

## 2. Building the supply & demand curves

We turn those discrete orders into two **step-functions**:

1. **Demand** $D(p)$: total shares buyers want at **or above** price $p$.

![image](https://github.com/user-attachments/assets/5ce28eb8-a028-45ee-b05c-fc980969ef4d)

2. **Supply** $S(p)$: total shares sellers want at **or below** price $p$.

![image](https://github.com/user-attachments/assets/4cf18f75-00f1-445a-a67e-fafa61093091)


Here $\mathbf{1}\{\dots\}$ is the **indicator function**—it’s 1 if the condition inside the braces is true, and 0 otherwise. So, for example, if bid $i$ has $p_i=50$ and we evaluate $\mathbf{1}\{50 \ge 48\}$, that’s 1; but $\mathbf{1}\{50 \ge 52\}$ is 0.

Graphically, $D(p)$ is a **non-increasing** step curve (as price goes up, fewer buyers remain), and $S(p)$ is **non-decreasing**.

---

## 3. Finding the clearing price

We look for the price $p^*$ that **maximizes the traded volume**, i.e.

![image](https://github.com/user-attachments/assets/96d33767-1850-4542-99f1-e919f0be2707)

* $\min\bigl(D(p),\,S(p)\bigr)$ is the number of shares we *can* match at price $p$ (you need both a buyer and a seller).
* We scan across all possible prices $p$ (typically the union of all bid/ask prices) and pick the one that makes this $\min$ as large as possible.

> **Notation note:**
>
> * $\arg\max$ means “the value of $p$ that makes the thing after it as big as possible.”
> * $\min(x,y)$ means the smaller of $x$ and $y$.

If there are ties (two prices give the same max volume), exchanges often break them by, say, choosing the **midpoint** between the highest price that cleared on the sell side and the lowest that cleared on the buy side.

---

## 4. Calculating how much actually trades

Once $p^*$ is set:

1. **Total executable volume**

V* = min(D(p*), S(p*))

2. **Who trades?**

   * **Buyers at or above** $p^*$:
![image](https://github.com/user-attachments/assets/7c9389b5-a4af-4369-8e88-81892d7470c9)

   * **Sellers at or below** $p^*$:
![image](https://github.com/user-attachments/assets/6aa39281-8e47-468d-9617-c6f672e43c8c)


3. **Pro-rata allocation** (if one side is oversubscribed):

   * Suppose total buy-demand at $\ge p* is D(p*)$, but D(p*) > S(p*). Then we only have S(p*) shares to allocate among buyers. Each buyer $i$ gets

![image](https://github.com/user-attachments/assets/03445c9a-b2c7-4494-b5fe-ed01047941f0)

   * Symmetrically for sellers if S(p*) > D(p*).

4. **Result**

   * Every matched buyer and seller trades at the **same price** $p^*$.
   * Partially filled orders carry their leftover $q_i - q'_i$ into the next batch.

---

## 5. A tiny numerical example

| Order | Side | $p$ | $q$ |
| :---: | :--: | :-: | :-: |
|   1   |  Buy |  10 | 100 |
|   2   |  Buy |  9  | 200 |
|   A   | Sell |  8  | 150 |
|   B   | Sell |  10 | 100 |

1. **Demand**

   * At $p=10$: $D(10)=100$
   * At $p=9$: $D(9)=100+200=300$
2. **Supply**

   * At $p=8$: $S(8)=150$
   * At $p=9$: $S(9)=150$ (no new sells ≤9)
   * At $p=10$: $S(10)=150+100=250$
3. **Compute $\min(D,S)$**

   * $p=10:\;\min(100,250)=100$
   * $p=9:\;\min(300,150)=150$
   * $p=8:\;\min(300,150)=150$
     The max is 150, so p*=8 or 9 (tie). Suppose we pick p*=9.
4. **Everything at ≥9 on bids** (orders 1 & 2) and ≤9 on asks (just A) trades:

   * Buyers total $D(9)=300$, sellers $S(9)=150$.
   * That means sellers are the scarce side: each share they offered gets fully sold, buyers get pro-rata:

     * Order 1 gets $100 × (150/300)=50$ shares
     * Order 2 gets $200 × (150/300)=100$ shares

All at price \$9, and you’ve just cleared 150 shares in one go.
* **Step functions** $D(p),\,S(p)$ built by summation
* **Argmax/min** to pick clearing price
* **Pro-rata** fractions to split scarce volume

--

---
id: the-problem
sidebar_position: 1
---

# Optimization problem

In this section, we describe all the different components of the optimization problem that needs to be solved within each batch.

## User orders

Suppose that there are $$\{1,2,...k\}$$ tokens. From a high-level perspective, we can define a user order as an _acceptance set_ $$S \subset \mathbb R^k$$ specifying the trades a user is willing to accept (where negative entries of a vector represent tokens sold, while positive entries represent tokens bought). So, for example, if $$k=2$$ and $$\begin{bmatrix} x \\-y \end{bmatrix}\in S$$ then a user is happy to receive _x_ units of token 1 in exchange for _y_ units of token 2.

:::note

This is from the user's perspective, and is therefore net of fees.

:::

We also assume that $$0 \in S$$ that is, when submitting an order a user accepts that the order may not be filled. Also, to each order $$S$$ we define _surplus_function_ $$U_S:S\rightarrow \mathbb R$$, measuring "how good" a trade is from the point of view of the user who submitted order _S_. By definition $$U_S(0)=0$$.

Practically speaking, CoW Protocol allows only some types of orders, which we can think of as constraints on the set _S_ that a user can submit. One such constraint is that only pairwise swaps are allowed, that is, all vectors in $$S$$  have zeros in $$k-2$$ dimensions. Furthermore, each order must fit within one of the categories we now discuss. To simplify notation, when discussing these categories, we assume that $$k=2$$.

### Limit Sell Orders

A _limit sell order_ specifies a maximum sell amount of a given token _Y_ > 0, a buy token _b_, and a limit price $$\pi$$, that corresponds to the worst-case exchange rate that the user is willing to settle for. They can be fill-or-kill whenever the executed sell amount must be Y (or nothing). They can be partially fillable if the executed sell amount can be smaller or equal to Y.  Formally, if _x_ denotes the (proposed) buy amount and _y_ denotes the (proposed) sell amount of the order, a fill-or-kill limit sell order has the form

$$S=\left\{\begin{bmatrix} x \\-y \end{bmatrix}~~s.t. ~~\frac{y}{\pi}\leq x \hbox{ and } y\in\{0,Y\} \right\},$$

and a partially-fillable sell order has the form

$$S= \left \{ \begin{bmatrix} x \\-y \end{bmatrix} ~~s.t. ~~\frac{y}{\pi} \leq x \hbox{ and } y \in [0,Y] \right \}.$$

In both cases, the surplus function is defined as

$$U(x,-y)= x-y / \pi$$,

i.e., it is the additional amount of buy tokens received by the user relative to the case in which they trade at the limit price, and is naturally expressed in units of the buy token.

A final observation is that orders can be valid over multiple batches. For a fill-or-kill, this means that an order that is not filled remains valid for a certain period (specified by the user). For a partially-fillable order, this also means that only a fraction of it may be executed in any given batch.

### Limit Buy Orders

A _limit buy order_ is specified by a maximum buy amount _X_ > 0 and a limit price $$\pi$$ corresponding to the worst-case exchange rate the user is willing to settle for. With _x_ denoting the buy amount and _y_ denoting the sell amount of the order, fill-or-kill limit buy orders have the form

$$S = \left\{\begin{bmatrix} x \\-y \end{bmatrix}~~s.t.~~ y \leq x \cdot \pi \hbox{ and } x \in\{0, X\} \right\}$$

while partially-fillable limit buy orders have the form

$$S = \left\{\begin{bmatrix} x \\-y \end{bmatrix}~~s.t.~~ y \leq x \cdot \pi \hbox{ and } x \in[0, X] \right\}.$$

Again, the surplus function is defined as

$$U(\{x,-y\})= x \cdot \pi - y$$.

Also here, orders can be executed over multiple batches.

## Protocol Fees

Each user order may have an associated fee paid to the protocol. At a high level, these fees can be represented by a function that, for a given order $$S$$ maps all possible trades to a non-negative vector of tokens, that is $$f_S:S \rightarrow \mathbb R^k_+$$   with $$f_S(0)=0$$.

:::note

Solvers are also expected to charge a fee to cover the cost of executing an order. We discuss such fees later in the context of solvers' optimal bidding, but we do not account for them here as they are not part of the protocol.

:::

## Solution

Solvers propose solutions to the protocol, where a solution is a set of trades to execute. Formally, suppose there are $$I$$ users and _J_ liquidity sources. A solution is a list of trades $$\{o_1, o_2, ...o_I, l_1, l_2, ..., l_J\}$$ one per user and one per liquidity source such that:

* **Incentive compatibility and feasibility**: the trades respect the user and liquidity sources, that is, $$o_i\in S_i~~\forall i\leq I$$  and $$l_j \in L_j~~\forall j\leq J$$.
* **Sufficient endowments**:  each user should have enough sell tokens in their wallet. Note that the protocol already performs this check at order creation. However, a user could move funds between order creation and execution or create multiple orders pledging the same sell amount multiple times. Hence, each solver should  also check that users' endowments are sufficient to execute the proposed solution.
* **Uniform clearing prices**: all users must face the same prices. Importantly, this constraint is defined at the moment when the swap occurs. So, for example, suppose user _i_ receives _x_ units of token 1 in exchange for _y_ units of token 2 and that the protocol takes a fee in the sell token $$f_2$$. Define $$p_{1,2}=\frac{y-f_2}{x}$$ as the price at which the swap occurs. Uniform clearing prices means that $$p_{1,2}$$ is the same for all users swapping token 1 and token 2. Furthermore, prices must be consistent, in the sense that for any three tokens 1, 2, and 3, if $$p_{1,2},~ p_{2,3}, ~p_{1,3}$$ are well-defined, then it must be that $$p_{1,2}\cdot p_{2,3}=p_{1,3}$$. Note that this implies that prices can be expressed with respect to a common numéraire, giving rise to a uniform price clearing vector $$p$$.
* [**Social consensus rules**](competition-rules): These are a set of principles that solvers should follow, which were voted by CIPs.

:::caution

At CoW DAO's discretion, systematic violation of these rules may lead to penalizing or slashing of the offending solver.

:::


From the protocol viewpoint, each solution that satisfies the above constraints has a _score_ that is given, roughly speaking, by the total surplus generated and the fees paid to the protocol, all aggregated and denominated in some numéraire. More specifically, the score of a solution is equal to the sum of scores of the orders the solution proposes to execute, where the score of an order $$o$$ is defined as:

* $$o$$ is a sell order: $$\mathrm{score}(o) = (U(o)+ f(o)) \cdot p(b)$$, where $$p(b)$$ is an externally provided price of the buy token relative to a numéraire.
* $$o$$ is a buy order:  $$\mathrm{score}(o) = (U(o)+ f(o)) \cdot p(b) / \pi$$, where $$p(b)$$ is an externally provided price of the buy token relative to a numéraire and $$\pi$$ is the limit price of the order.

We stress that in the above definition of score, we have assumed that potential protocol fees associated with a trade are always captured in the surplus token of the order. In case there is need in the future for a more general protocol fee, the above definition of score will need to be reworked.

Finally, solvers compete for the right to settle a batch by participating in an auction, aiming to implement the solution that generates the largest possible score. The [solver that wins the auction is rewarded](rewards) by the protocol.
