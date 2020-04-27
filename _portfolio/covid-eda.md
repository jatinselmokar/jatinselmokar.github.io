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

```python
#Number for Countries affected over time

dfpivot = df.pivot_table(index = 'Country/Region', columns = 'Date', values = ['Confirmed'])

countrycount =  dfpivot.groupby("Country/Region").sum().apply(lambda x: x[x > 0].count(), axis =0)
countrycount = pd.DataFrame(countrycount)

countrycount.reset_index(inplace = True)
countrycount.columns = ["Metric", "Date", "Count"]


#Animated Line Chart
trace = go.Scatter(x=countrycount['Date'][0:2], y=countrycount['Count'][0:2],
                         mode = 'markers', line = dict(width = 2))
                         ['Count'][:k+1])],
                                        traces = [0,1],
                                        ) for k in range(1, len(countrycount) - 1)
                                  ]

                         layout = go.Layout(width = 600,
                                            height = 440,
                                            showlegend = False, hovermode = 'closest',
                                             updatemenus=[dict(type='buttons', showactive=False,
                                                         y=1.05,
                                                         x=1.15,
                                                         xanchor='right',
                                                         yanchor='top',
                                                         pad=dict(t=0, r=10),
                                                         buttons=[dict(label='Play',
                                                                       method='animate',
                                                                       args=[None,
                                                                             dict(frame=dict(duration=30,
                                                                                             redraw=False),
                                                                                  transition=dict(duration=0),
                                                                                  fromcurrent=True,
                                                                                  mode='immediate')])])]
                                            )

layout.update(xaxis =dict(range=[countrycount.Date[0], countrycount.Date[len(countrycount)-1]],
                        autorange=False, showgrid = True, showline = True,
                        showticklabels=True,
                        linecolor = 'rgb(204, 204, 204)',
                        linewidth = 2
                       ),
            yaxis =dict(range=[min(countrycount.Count)-10, max(countrycount.Count)+20],
                        autorange=False, showline = True,
                        showticklabels=True,
                        linecolor = 'rgb(204, 204, 204)',
                        linewidth = 2
                       ),
            title = "Number Of Countries Affected Over Time"
           );
fig = go.Figure(data=[trace], frames=frames, layout=layout)

fig.show()


import chart_studio.plotly as py
py.plot(fig, filename = 'countries-affected-over-time', auto_open=True)

```
*Note: Click on "Play" to see the animation.*
<iframe width="800" height="450" frameborder="0" scrolling="no" src="//plotly.com/~jatins/1.embed"></iframe>

## World Map - Confirmed Cases

```python
#Choropleth Map for Confirmed
pio.templates.default = "ggplot2"
fig = px.choropleth(dflatest, locations="Country/Region",
                    color=np.log10(dflatest["Confirmed"]),
                    hover_name="Country/Region",
                    hover_data=["Confirmed"],
                    color_continuous_scale=px.colors.sequential.Plasma,locationmode="country names")

fig.update_layout(title="Confirmed Cases Heat Map (Log Scale)")
fig.update_coloraxes(colorbar_title="Confirmed Cases(Log Scale)",colorscale="tealrose")
fig.update_geos(fitbounds="locations", visible=False, projection_type="orthographic", oceancolor = '#afd4db',  showocean = True)
# fig.to_image("Global Heat Map confirmed.png")
fig.show()


import plotly.io as pio
pio.write_html(figtrend, file="confirmedcasesworldmap.html", auto_open=True)
```

{% include confirmedcasesworldmap.html %}

## World Trend Over Time
{% include worldtrend.html %}

## World Confirmed Cases - Time Lapse
{% include worldconfirmedtimelapse.html %}

## US Confirmed Cases Heat Map
{% include usheatmap.html %}

## Top 10 States in the US
{% include top10states.html %}

## Top 10 States in the US (Dark Mode)
{% include top10states_darkmode.html %}

<!--
<p> Race chart of the trend - </p>

<div class="flourish-embed flourish-bar-chart-race" data-src="visualisation/1873703" data-url="https://flo.uri.sh/visualisation/1873703/embed"><script src="https://public.flourish.studio/resources/embed.js"></script></div> -->
