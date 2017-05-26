---
layout: post
title:  "Деление многочленов с остатком"
permalink: polynom-division
date:   2017-05-26 02:07:41 +0300
categories: algorithm
---

### Пререквизиты

* [Быстрое преобразование Фурье]({{ site.baseurl }}{% post_url 2017-04-14-fft %})
* [Обратный степенной ряд]({{ site.baseurl }}{% post_url 2017-04-21-fft-division %})

### Постановка задачи

Даны многочлены $A(x)$, $B(x)$, найти такие многочлены $Q(x)$, $R(x)$, что $A(x) = Q(x)B(x) + R(x)$, то есть поделить $A(x)$ на $B(x)$ с остатком.

Далее будем считать $deg(A)=n$, $deg(B)=m$, $n \ge m$.

### Тривиальный алгоритм

Школьный алгоритм деления многочленов столбиком работает за время $O(nm)$.

{% highlight cpp linenos %}
typedef vector<double> polynom;

const double eps = 1e-9;
bool Equal(double x, double y)
{
	return fabs(x - y) < eps;
}

pair<polynom, polynom> divide(polynom A, polynom B)
{
	int n = (int)A.size() - 1;
	int m = (int)B.size() - 1;
	polynom Q = polynom(n - m + 1);
	for (int i = n; i >= m; i--)
	{
		Q[i - m] = A[i] / B[m];
		for (int j = m; j >= 0; j--)
			A[i - m + j] -= B[j] * Q[i - m];
	}
	A.resize(m);
	while(A.size() > 1 && Equal(A.back(), 0))
		A.pop_back();
	return make_pair(Q, A);
}
{% endhighlight %}

### Алгоритм за $O(n \log n)$

Пусть $F(x) = \sum_{i=0}^{n} f_{i}x^{i}$. Мы будем использовать обозначение $\overline{F}(x) = \sum_{i=0}^{n} f_{n-i}x^{i}$, то есть $\overline{F}(x)$ &mdash; это многочлен, полученный разворотом вектора коэффициентов многочлена $F(x)$.

Понятно, что $deg(Q)=n-m$ и $deg(R)<m$. Мы будем считать, что $deg(R)=m-1$, то есть дополним его нулями при необходимости. $\overline{R}(x)$ мы будем понимать именно в этом смысле, разворачиваются ровно $m$ коэффициентов.

$$
A(x) = Q(x)B(x) + R(x) \Rightarrow x^{n}A(\frac{1}{x}) = x^{n} \left( Q(\frac{1}{x}) B(\frac{1}{x}) + R(\frac{1}{x}) \right) \Rightarrow \overline{A}(x) = \overline{Q}(x) \overline{B}(x) + x^{n+1-m} \overline{R}(x)
$$

Пусть теперь $D(x)$ &mdash; это формальный степенной ряд такой, что $\overline{B}(x) D(x) = 1$. Алгоритм поиска первых коэффициентов этого ряда описан в [соответствующей статье]({{ site.baseurl }}{% post_url 2017-04-21-fft-division %}). Тогда

$$
\overline{A}(x) D(x) = \overline{Q}(x) + x^{n+1-m} \overline{R}(x) D(x)
$$

Однако $\overline{Q}(x)$ имеет степень не больше $n-m$, поэтому первые $n+1-m$ коэффициентов $\overline{A}(x) D(x)$ и есть $\overline{Q}(x)$. Таким образом, для нахождения $\overline{Q}(x)$ достаточно вычислить первые $n+1-m$ коэффициент степенного ряда $D(x)$, а затем умножить его на $\overline{A}(x)$ с помощью FFT. Всё это можно сделать за время $O(n \log n)$.

Из $\overline{Q}(x)$ мы получаем $Q(x)$, а затем можно вычислить и $R(x)$ по формуле $R(x) = A(x) - Q(x)B(x)$, умножив $Q(x)$ на $B(x)$ с помощью FFT.

Итоговое время работы &mdash; $O(n \log n)$.

### Код

{% highlight cpp linenos %}
typedef vector<double> polynom;

polynom multiply(polynom A, polynom B);
polynom inverse(polynom F);

const double eps = 1e-9;
bool Equal(double x, double y)
{
	return fabs(x - y) < eps;
}

polynom divide(polynom A, polynom B)
{
	int n = (int)A.size() - 1;
	int m = (int)B.size() - 1;
	polynom A2 = A, B2 = B;
	reverse(A2.begin(), A2.end());
	reverse(B2.begin(), B2.end());
	while((int)B2.size() <= n - m)
		B2.push_back(0);
	polynom Q = multiply(A2, inverse(B2));
	Q.resize(n + 1 - m);
	reverse(Q.begin(), Q.end());
	polynom R = multiply(Q, B);
	for (int i = 0; i < (int)R.size(); i++)
		R[i] = A[i] - R[i];
	while(R.size() > 1 && Equal(R.back(), 0))
		R.pop_back();
	return make_pair(Q, R);
}
{% endhighlight %}