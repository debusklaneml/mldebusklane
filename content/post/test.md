+++
title = "tests"
date = 2018-12-01T00:00:00

# Authors. Comma separated list, e.g. `["Bob Smith", "David Jones"]`.
authors = ["Morgan DeBusk-Lane"]

# Publication type.
# Legend:
# 0 = Uncategorized
# 1 = Conference proceedings
# 2 = Journal
# 3 = Work in progress
# 4 = Technical report
# 5 = Book
# 6 = Book chapter
publication_types = ["2"]

# Publication name and optional abbreviated version.
#publication = "In *Research in Science Eduation*"
#publication_short = "In *PB&R*"

# Abstract and optional shortened version.
abstract = "Abstract goes here"
#abstract_short = "A mobile visual clothing search system is presented..."

# Featured image thumbnail (optional)
image_preview = ""

# Is this a selected publication? (true/false)
selected = false

# Projects (optional).
#   Associate this publication with one or more of your projects.
#   Simply enter the filename (excluding '.md') of your project file in `content/project/`.
#projects = ["example-external-project"]

# Links (optional).
url_pdf = ""
url_preprint = ""
url_code = ""
url_dataset = ""
url_project = ""
url_slides = ""
url_video = ""
url_poster = ""
url_source = ""

# Custom links (optional).
#   Uncomment line below to enable. For multiple links, use the form `[{...}, {...}, {...}]`.
#url_custom = [{name = "Custom Link", url = "http://example.org"}]

# Does the content use math formatting?
math = true

# Does the content use source code highlighting?
highlight = true

# Featured image
# Place your image in the `static/img/` folder and reference its filename below, e.g. `image = "example.jpg"`.
[header]
#image = "headers/bubbles-wide.jpg"
#caption = "My caption :smile:"

+++

Testing content here ```{r} testing ```

```{r echo=F, warning= F, message=F}
opts_chunk$set(message = FALSE, warning = FALSE, error = FALSE, tidy = FALSE, cache = FALSE,results = 'asis' )
```

---
<br/>
### Another Favorite from NYT
  
I think we all know the [data visualization team at NYT](http://blog.visual.ly/10-things-you-can-learn-from-the-new-york-times-data-visualizations/) is simply amazing.  Earlier this year in my post [d3 <- R with rCharts and slidify](http://timelyportfolio.blogspot.com/2013/04/d3-r-with-rcharts-and-slidify.html) I adapted and recreated the [512 Paths to the White House](http://www.nytimes.com/interactive/2012/11/02/us/politics/paths-to-the-white-house.html) to work with `R` data through [`rCharts`](http://rcharts.io/site).  Unfortunately, I was not creative enough to think of other data sets to plug into the visualization.  When Scott Murray tweeted, 

<br/>
<blockquote class="twitter-tweet"><p>Over at <a href="https://twitter.com/nytgraphics">@nytgraphics</a>, <a href="https://twitter.com/KevinQ">@KevinQ</a> and <a href="https://twitter.com/shancarter">@shancarter</a> really know how to wiggle a baseline. <a href="http://t.co/gS9gHrSLIu">http://t.co/gS9gHrSLIu</a></p>&mdash; Scott Murray (@alignedleft) <a href="https://twitter.com/alignedleft/statuses/349647895122804738">June 25, 2013</a></blockquote>
<script async src="http://platform.twitter.com/widgets.js" charset="utf-8"></script>
<br/>

I immediately knew the [Case Shiller Home Price Index visualization](http://www.nytimes.com/interactive/2011/05/31/business/economy/case-shiller-index.html) would be perfect for reuse with any cumulative growth time series data.   This is a bit of a hack of `rCharts` and should not be considered best practices, but it is a demonstration of the very flexible design of the package.  In the spirit of this [discussion](http://datastori.es/data-stories-23-inspiration-or-plagiarism/), I did not want to just copy entirely.  I was able to add a couple key innovations to the visualization:

- Generalize the d3 code a little more to adapt to the data
- Build in `R` with `rCharts` to make it reusable.


---
<br/>
### Reusable Version in rCharts
As I mentioned above, this visualization works well with any cumulative growth time series, so let's apply it to the `managers` dataset supplied by the `PerformanceAnalytics` package.

```{r echo = F, eval = F, results = 'asis'}
#I include this in case you want to create the original
p1 <- rCharts$new()
p1$setLib('libraries/widgets/nyt_home')
p1$setTemplate(script = "libraries/widgets/nyt_home/layouts/nyt_home.html")
p1$set(description = "The Standard & Poor's Case-Shiller Home Price Index for 20 major metropolitan areas is one of the most closely watched gauges of the housing market. The figures for April were released June 25. Figures shown here are not seasonally adjusted or adjusted for inflation.")
#get the data and convert to a format that we would expect from melted xts
#will be typical
#also original only uses a single value (val) and not other 
data <- read.csv("data/case-shiller-tiered2.csv", stringsAsFactors = F)
data.melt <- data.frame(
  format(as.Date(paste(data[,3],data[,4],"01",sep="-"),format = "%Y-%m-%d")),
  data[,c(1,2,5)]
)
colnames(data.melt) <- c("date",colnames(data.melt)[-1])
p1$set(data = data.melt)
p1$set(groups = "citycode")
#cat(noquote(p1$html()))
```

<h4>Get Data and Transform</h4>
```{r}
#get the data and convert to a format that we would expect from melted xts
#will be typical
#also original only uses a single value (val) and not other 
require(reshape2)
require(PerformanceAnalytics)
data(managers)
managers <- na.omit(managers)
managers.melt <- melt(
  data.frame( index( managers ), coredata(cumprod( managers+1 )*100 ) ),
  id.vars = 1
)
colnames(managers.melt) <- c("date", "manager","val")
managers.melt[,"date"] <- format(managers.melt[,"date"],format = "%Y-%m-%d")
```

<h4>Draw The Graph</h4>
```{r}
require(rCharts)
p2 <- rCharts$new()
p2$setLib('libraries/widgets/nyt_home')
p2$setTemplate(script = "libraries/widgets/nyt_home/layouts/nyt_home.html")
p2$set(
  description = "This data comes from the managers dataset included in the R package PerformanceAnalytics.",
  data = managers.melt,
  groups = "manager"
)
cat(noquote(p2$html()))
```
