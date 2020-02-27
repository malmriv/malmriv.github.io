---
title: 'Tracking the coronavirus outbreak.'
date: 2020-02-27
permalink: /posts/2020/02/coronabot/
tags:
  - R
  - data visualization
  - public health
  - data mining
---

Watching the news has been very discouraging lately. Every major media outlet seems to be anxious to hoard clicks, views and tweets, which is especially dangerous in situations like the current one. The coronavirus outbreak seems to be being blown out of proportion in a similar way to what happened regarding the H1N1 virus not many years ago. But not everything is so bad when one looks at the data. The case fatality rate is low, almost everyone who gets sick recovers, and looking at the numbers it's hard to believe this could become the End of Humanity, as some people are frantically claiming.

So yesterday I put together a few lines of code and connected them to [a Twitter account](http://www.twitter.com/covidbot): the [coronabot](https://github.com/malmriv/coronabot). The bot can be visited in the first link, and the details and code can be found in the second link. It's basically an R script that reads live information about the outbreak, cleans and processes it to compute some encouraging bits of info. It tweets both in English and Spanish.
