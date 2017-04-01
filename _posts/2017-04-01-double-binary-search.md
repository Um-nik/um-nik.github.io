---
layout: post
title:  "Двоичный поиск для double"
permalink: double-binsearch
date:   2017-04-01 22:07:41 +0300
categories: algorithm
---
Suppose we want to solve a problem by doing binary search on answer. Then the answer will be checked against jury's answer by absolute or relative error (one of them should be smaller then \\( \varepsilon \\)). For simplicity we will assume that our answer is always greater than \\( 1 \\) and smaller than \\( B \\). Because of that, we will always use relative error rather than absolute.

Suppose we have made \\( n \\) iterations of our binary search &mdash; what information do we have now? I state that we know that real answer is lying in some segment \\( [x_{i}, x_{i+1}] \\), where \\( 1 = x_{0} < x_{1} < \ldots < x_{i} < \ldots < x_{2^{n}} = B \\). And what is great &mdash; we can choose all \\( x_{i} \\) except for \\( x_{0} \\) and \\( x_{2^{n}} \\).

Now, for simplicity, we will also assume that we will answer \\( x_{i+1} \\) for segment \\( [x_{i}, x_{i+1}] \\) and the real answer was \\( x_{i} \\) &mdash; it is the worst case for us. It is obvious that we will not do that in real life, any other answer would be better, but you will get the idea.

So, what is our relative error? It is \\( \frac{x_{i+1}-x_{i}}{x_{i}} = \frac{\Delta x_{i}}{x_{i}} \\). Worst case for us is when relative error is maximal. It is logical to make them equal &mdash; exactly what we do by binary search with absolute errors. \\( \frac{\Delta x_{i}}{x_{i}} = \frac{\Delta x_{j}}{x_{j}} \\). We can assume that \\( \Delta x_{i} \ll x_{i} \\) so \\( \frac{\Delta x_{i}}{x_{i}} \approx \Delta \ln x_{i} \\). Now we have \\( \Delta \ln x_{0} = \Delta \ln x_{1} = \ldots = \Delta \ln x_{2^{n}-1} \\), but \\( \Delta \ln x_{0} + \Delta \ln x_{1} + \ldots + \Delta \ln x_{2^{n}-1} = \ln B \\), so \\( \Delta \ln x_{i} = \frac{\ln B}{2^{n}} \\). How large should be \\( n \\) to get error less than \\( \varepsilon \\)? \\( n = \log_{2}(\ln B \cdot \varepsilon^{-1}) \\). Much smaller than \\( \log_{2}(B \cdot \varepsilon^{-1}) \\).

How to write such binary search? We want to choose \\( m \\) in such a way that \\( \ln m - \ln l = \ln r - \ln m \\) or simply \\( m = \sqrt{l \cdot r} \\).

Now I want to deal with some assumptions I made.

How to choose answer in the end? Again, \\( \sqrt{l \cdot r} \\) (it is basically the same as dividing the segment in binary search).

What to do if the answer can be smaller than \\( 1 \\)? Try \\( 1 \\); if answer is smaller than \\( 1 \\) &mdash; use standard binary search (because absolute error smaller than relative); is answer is bigger than \\( 1 \\) &mdash; use the binary search above.