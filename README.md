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
