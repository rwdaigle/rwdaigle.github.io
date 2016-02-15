---
layout: post
title:  "Mocks and Explicit Contracts in Practice with an Elixir Slackbot"
date:   2016-02-15
---

I gave a brief talk at the [Triangle |> Elixir group](http://triangle-elixir.github.io) last week where I discussed my experience putting into practice Josè Valim's advice on how to [properly mock components](http://blog.plataformatec.com.br/2015/10/mocks-and-explicit-contracts/) that integrate with a remote service. While the concepts are not Elixir-specific, I used a Phonenix Slackbot project as an example, which you [can see here](https://github.com/rwdaigle/spreedly-docs-slackbot).

<iframe width="420" height="315" src="https://www.youtube.com/embed/NDfB3bVnvnA" frameborder="0" allowfullscreen></iframe>

The summary is - I think it worked out really well! There is no silver bullet when it comes to testing an integration to a remote service - at some point you have to test that the actual calls are being sent and processed correctly. However, this pattern isolates that process to a single, well-defined, and interchangeable component and using tags to explicitly invoke or ignore the remote tests works really well in practice.

If you want to view how the remote component was modeled, see the [`Search` modules here](https://github.com/rwdaigle/spreedly-docs-slackbot/tree/f3d7d9f108a2da2ae629ce68dd5f1351f3993ec1/web/models/search) and the [defining Behavior here](https://github.com/rwdaigle/spreedly-docs-slackbot/blob/f3d7d9f108a2da2ae629ce68dd5f1351f3993ec1/web/models/search.ex).
