---
layout: post
title:  "Сумма по всем подмаскам"
permalink: subset-sums
date:   2017-05-01 02:07:41 +0300
categories: algorithm
---

### Постановка задачи

Дан массив $a[2^{n}]$, посчитать массив $b[2^{n}]$, где $b[mask] = \sum\limits_{submask \subseteq mask} a[submask]$

### Тривиальное решение

Можно просто для каждой маски перебрать все её подмаски. При правильной организации решения это будет $O(3^{n})$.

{% highlight cpp linenos %}
const int N = (1 << 18) + 3;
int a[N], b[N];
int n;

void brute(int pos, int mask, int submask)
{
	if (pos == n)
	{
		b[mask] += a[submask];
		return;
	}
	brute(pos + 1, mask, submask);
	brute(pos + 1, mask | (1 << pos), submask);
	brute(pos + 1, mask | (1 << pos), submask | (1 << pos));
}

void solve()
{
	brute(0, 0, 0);
}
{% endhighlight %}

Или нерекурсивно:

{% highlight cpp linenos %}
const int N = (1 << 18) + 3;
int a[N], b[N];
int n;

void solve()
{
	for (int mask = 0; mask < (1 << n); mask++)
	{
		b[mask] = a[0];
		// generate all submasks of mask (except for empty)
		for (int submask = mask; submask > 0; submask = (submask - 1) & mask)
			b[mask] += a[submask];
	}
}
{% endhighlight %}

### Решение за $O(2^{n}n)$

Задачу можно переформулировать так: есть $n$-мерный массив $2 \times 2 \times \ldots \times 2$, мы хотим посчитать в нём префиксные суммы. Связь с исходной задачей такая: $i$-я координата &mdash; это $i$-й бит маски. Несложно понять, что это действительно одна и та же задача.

Решение в новой формулировке достаточно простое: нужно просуммировать сначала по первой координате, потом результаты по второй координате и так далее.

Эту идею можно оформить в виде изящной ДП: $dp[k][mask]$ &mdash; сумма по всем $submask$ таким, что в первых $k$ битах это подмаска $mask$, а в остальных $n-k$ битах она полностью совпадает с $mask$. Тогда $a = dp[0]$ и $b = dp[n]$. При переходе от $k$-го слоя к $(k+1)$-му нужно посмотреть на $k$-й бит маски, и если он равен $1$, то прибавить значение из маски без этого бита.

{% highlight cpp linenos %}
const int K = 21;
const int N = (1 << K) + 3;
int a[N], b[N];
int dp[K + 1][N];
int n;

void solve()
{
	for (int mask = 0; mask < (1 << n); mask++)
		dp[0][mask] = a[mask];
	for (int k = 0; k < n; k++)
		for (int mask = 0; mask < (1 << n); mask++)
		{
			dp[k + 1][mask] = dp[k][mask];
			if ((mask >> k) & 1)
				dp[k + 1][mask] += dp[k][mask ^ (1 << k)];
		}
	for (int mask = 0; mask < (1 << n); mask++)
		b[mask] = dp[n][mask];
}
{% endhighlight %}

Разумеется, для экономии памяти можно хранить только 2 слоя динамики.

А если исходный массив сохранять не нужно, то всё это можно делать вообще на месте, используя $O(1)$ дополнтельной памяти.

{% highlight cpp linenos %}
const int K = 21;
const int N = (1 << K) + 3;
int a[N];
int n;

void solve()
{
	for (int k = 0; k < n; k++)
		for (int mask = 0; mask < (1 << n); mask++)
			if ((mask >> k) & 1)
				a[mask] += a[mask ^ (1 << k)];
}
{% endhighlight %}

### Обратная задача

По массиву $b$ восстановить массив $a$.

Воспользуемся принципом включений-исключений. Докажем по индукции, что

$$
a[z] = \sum\limits_{x \subseteq z} \left( -1 \right) ^{ | z | - | x |} b[x]
$$

$$
\begin{multline}
a[z] = b[z] - \sum\limits_{y \subsetneq z} a[y] = b[z] - \sum\limits_{y \subsetneq z} \sum\limits_{x \subseteq y} \left( -1 \right) ^{| y | - | x |} b[x] = \\
b[z] - \sum\limits_{x \subsetneq z} \sum\limits_{x \subseteq y \subsetneq z} \left( -1 \right) ^{| y | - | x |} b[x] = b[z] - \sum\limits_{x \subsetneq z} \left( b[x] \sum\limits_{y \subsetneq \left( z \oplus x \right)} \left( -1 \right) ^{| y |} \right) = \\
b[z] - \sum\limits_{x \subsetneq z} \left( b[x] \left( -1 \right) ^{| z \oplus x |} \left( -1 \right) \right) = \sum\limits_{x \subseteq z} \left( -1 \right) ^{| z | - | x |} b[x]
\end{multline}
$$

Здесь везде $\| mask \|$ &mdash; это количество включенных битов в $mask$.

Заметим теперь, что в нашей динамике нетривиальные переходы &mdash; это выключение одного бита в маске. Наша формула с ПВИ говорит, что при выключении одного бита мы должны поменять знак. Таким образом, обратная задача решается точно так же, как прямая, только при выключении бита нужно вычитать значение динамики, а не прибавлять.

Для краткости, код я приведу только для полной версии.

{% highlight cpp linenos %}
const int K = 21;
const int N = (1 << K) + 3;
int a[N], b[N];
int dp[K + 1][N];
int n;

void solve()
{
	for (int mask = 0; mask < (1 << n); mask++)
		dp[0][mask] = b[mask];
	for (int k = 0; k < n; k++)
		for (int mask = 0; mask < (1 << n); mask++)
		{
			dp[k + 1][mask] = dp[k][mask];
			if ((mask >> k) & 1)
				dp[k + 1][mask] -= dp[k][mask ^ (1 << k)];
		}
	for (int mask = 0; mask < (1 << n); mask++)
		a[mask] = dp[n][mask];
}
{% endhighlight %}