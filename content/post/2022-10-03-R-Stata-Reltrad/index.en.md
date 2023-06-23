---
title: 'Creating the Reltrad Variable in the General Social Survey Using R'
categories:
- R
- Measuring religion
tags: Religion
author: 'David Eagle, PhD'
slug: 'Creating Reltrad'
date: '2018-11-06T07:32:00-08:00'
lastmod: '2022-10-03T07:32:00-08:00'
output: html_document
bibliography: ["/Users/dee4/Zotero/My Library.bib"]
---

Haven’t you always wanted an easy to access R script to create the famous reltrad religious categorizaton in the General Social Survey? Well, today is your lucky day.The measure of religious affiliation described in Steensland et al. (2000), otherwise known as *reltrad*, remains the most popular way to categorize people religiously in the United States. Early development of the measure was done by Corwin Smidt, Bud Kellstedt and James Guth during their annual seminars on measuring religion offered at Calvin College.

The *reltrad* crowd continues to think their measure is pretty good (Woodberry et al. 2012). A *reltrad* for African Americans has been developed - this is required reading for those who use *reltrad* in their research (Shelton and Cobb 2017).

## Code Conundrums

Up until now, the code to create the reltrad variable from the General Social Survey is scattered about. A PDF of Stata code for reltrad is available from Lifeway Research. They have R code, too. [Ryan Burge has code up on Github](https://github.com/ryanburge/reltrad)

### A Better *Reltrad* Repository

I am providing [**my repository**](https://thebigbird/R_Stata_Reltrad), which I think is the best out there This has a Stata .do file and an R function. Note that I do incorporate the corrections suggested by Stetzer and Burge (2016) . Using Kieran Healy’s gssr package, the sample code loads the gss datafile, dumps the oversamples, and then calls the function I’ve written (now mostly in dplyr) to create a *reltrad* variable.

Again, [github.com/thebigbird/R_Stata_Reltrad](https://github.com/thebigbird/R_Stata_Reltrad).

### An Example

After the code is run, the results can be quickly plotted The following plot looks at how the religious composition of the US has changed over time.

``` r
#Create a Figure for GSS attendanc= over time
#Plot the proportions over time
source("https://raw.githubusercontent.com/thebigbird/R_Stata_Reltrad/master/ReltradGSS.R")
#See my github for the code to make these data

#Cut by generation
gss = gss %>% 
  mutate(year = as.numeric(as.character(year))) %>%
  mutate(birthyr = year - age) %>%
  mutate(age_cat = cut(birthyr, c(0,1928,1945,1965,1981,1997,2020)))

levels(gss$age_cat) = c("Great","Silent","Boomer","GenX","Millenial","GenZ")

library(srvyr)
#Calculate weighted proportions
#Create survey object using svyr
gss_svy <- gss %>% as_survey_design(1, weight = wtssall,
                                    variables = c(year, reltrad, age_cat))

#Now calculate proportions
out = gss_svy %>% 
  group_by(year, reltrad) %>%
  summarize(prop = survey_mean(na.rm=T, proportion = T)) %>%
  drop_na()

#A nice set o' colors
scFill = scale_color_manual(values = 
                              c("#1B9E77", 
                                "#DDC849",
                                "#7570b3", 
                                "#a6761d",
                                "#e7298a",
                                "#1f78b4",
                                "#a9a9a9",
                                "#cc99ff"))

plotout = ggplot(data = out, 
                 aes(x=year, 
                     y=prop,
                     color = reltrad,
                     group = reltrad)) +
  geom_ribbon(aes(ymin = prop - 1.96*prop_se,
                  ymax = prop + 1.96*prop_se),
                  color = "white",
                  alpha = .2, fill = "grey") +
  geom_point() +
  geom_line() +
  ylab("Proportion of US Population Identifying As...") +
  xlab("") +
  labs(title = "Religious Affiliation in the United States",
       subtitle = "General Social Survey",
       caption = "David Eagle / Data: https://gss.norc.org/",
       color = "Religious Tradition")+
  theme_minimal() + scFill +
  theme(axis.text.x = element_text(angle = 70, hjust = 1)) +
  ggtitle("Religious Affiliation in the United States 1972-2018")
#ggsave(plotout,"Output/reltradGSS.png", width = 12, height=5, dpi="retina")

plotout
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-1-1.png" width="960" />

## References

<div id="refs" class="references csl-bib-body hanging-indent">

<div id="ref-sheltonBlackReltradMeasuring2017" class="csl-entry">

Shelton, Jason E., and Ryon J. Cobb. 2017. “Black Reltrad: Measuring Religious Diversity and Commonality Among African Americans.” *Journal for the Scientific Study of Religion* 56 (4): 737–64. <https://doi.org/gfgw25>.

</div>

<div id="ref-steenslandMeasureAmericanReligion2000" class="csl-entry">

Steensland, B., L. D. Robinson, W. B. Wilcox, J. Z. Park, M. D. Regnerus, and R. D. Woodberry. 2000. “The Measure of American Religion: Toward Improving the State of the Art.” *Social Forces* 79 (1): 291–318. <https://doi.org/db9hrh>.

</div>

<div id="ref-stetzerReltradCodingProblems2016" class="csl-entry">

Stetzer, Ed, and Ryan P. Burge. 2016. “Reltrad Coding Problems and a New Repository.” *Politics and Religion* 9 (01): 187–90. <https://doi.org/gfkc5x>.

</div>

<div id="ref-woodberryMeasureAmericanReligious2012" class="csl-entry">

Woodberry, R. D., J. Z. Park, L. A. Kellstedt, M. D. Regnerus, and B. Steensland. 2012. “The Measure of American Religious Traditions: Theoretical and Measurement Considerations.” *Social Forces* 91 (1): 65–73. <https://doi.org/10.1093/sf/sos121>.

</div>

</div>
