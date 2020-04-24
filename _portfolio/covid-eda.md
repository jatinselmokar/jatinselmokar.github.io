---
title: "Exploratory Data Analysis - COVID-19"
excerpt: "Visualization of COVID19 data using plotly python package"
author_profile: true
categories:
  - EDA
tags:
  - COVID19
  - EDA
  - Visualization
  - plotly
classes: wide
---
## The Beginning...
COVID-19, a novel form of coronavirus family, was first identified in Wuhan, China in December 2019.  It swiftly spread to other countries to the extent that the World Health Organization (WHO) declared it a **pandemic** on March 11. Since inception, the infamous virus has caused major disruptions in almost every aspect of life and continues to affect millions all around the world.

## Countries Impacted Over Time
Although the rate of impact seemed constant in early February, the curve had reached an unquestionable inflection point towards the end of the month.

*Note: Click on "Play" to see the animation.*
<iframe width="800" height="450" frameborder="0" scrolling="no" src="//plotly.com/~jatins/1.embed"></iframe>


{% capture fig_img %}
![Foo]({{ '/assets/images/WorldTrend.jpg' | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>World Trend</figcaption>
</figure>


{% capture fig_img %}
![Foo]({{ '/assets/images/calmap_confirmed.jpg' | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>Calmap for confirmed</figcaption>
</figure>



{% capture fig_img %}
![Foo]({{ '/assets/images/calmap_deaths.jpg' | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>Calmap for deaths</figcaption>
</figure>




{% capture fig_img %}
![Foo]({{ '/assets/images/Top5USstates.jpg' | relative_url }})
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>Top 5 US States.</figcaption>
</figure>

<p> Race chart of the trend - </p>

<div class="flourish-embed flourish-bar-chart-race" data-src="visualisation/1873703" data-url="https://flo.uri.sh/visualisation/1873703/embed"><script src="https://public.flourish.studio/resources/embed.js"></script></div>
