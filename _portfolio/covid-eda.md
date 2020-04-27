---
title: "Exploratory Data Analysis - COVID-19"
excerpt: "Visualization of COVID19 Data Using Plotly Python Package"
header:
  overlay_image: /assets/images/COVIDHeader.jpg
  caption: "Photo credit: [**Unsplash**](https://unsplash.com)"
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
COVID-19, a novel form of coronavirus family, was first identified in Wuhan, China in December 2019. From then on, it swiftly spread to other countries to the extent that the World Health Organization (WHO) declared it a **pandemic** on March 11.

Since inception, the infamous virus has caused major disruptions in almost every aspect of life and continues to affect millions all around the world.

## Countries Impacted Over Time
Although the rate of impact seemed constant in early February, the curve had reached an unquestionable inflection point towards the end of the month.

*Note: Click on "Play" to see the animation.*
<iframe width="800" height="450" frameborder="0" scrolling="no" src="//plotly.com/~jatins/1.embed"></iframe>

## World Map - Confirmed Cases

Interactive world map built using the plotly's choropleth feature!

{% include confirmedcasesworldmap.html %}

## World Trend Over Time

{% include worldtrend.html %}

## World Confirmed Cases - Time Lapse

{% include worldconfirmedtimelapse.html %}

## US Confirmed Cases Heat Map

{% include usheatmap.html %}

## Top 10 States in the US

{% include top10states_darkmode.html %}


For more visualizations and the python code, please visit [GitHub!]('https://github.com/jatinselmokar/COVID-19-Exploratory-Data-Analysis-Using-PyPlot')
<!--
<p> Race chart of the trend - </p>

<div class="flourish-embed flourish-bar-chart-race" data-src="visualisation/1873703" data-url="https://flo.uri.sh/visualisation/1873703/embed"><script src="https://public.flourish.studio/resources/embed.js"></script></div> -->
