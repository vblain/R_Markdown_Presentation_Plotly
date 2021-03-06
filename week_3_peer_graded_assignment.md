<style>
  
iframe {
  position: absolute;
  top: 50%; 
  left: 50%;
  -webkit-transform: translateX(-50%) translateY(-50%);
  transform: translateX(-50%) translateY(-50%);
  min-width: 100vw; 
  min-height: 100vh; 
  z-index: -1000; 
  overflow: hidden;
}
</style>

week 3 peer graded assignment
========================================================
author: Vincent Blain
date: 04/15/2021
autosize: true

Developing Data Products - Week 3 Peer-graded Assignment
========================================================

Create a web page presentation using R Markdown that features a plot created with Plotly. Host your webpage on either GitHub Pages, RPubs, or NeoCities. Your webpage must contain the date that you created the document, and it must contain a plot created with Plotly. We would love to see you show off your creativity! 



Setup data for 3D Surface using Plotly and built in mtcars dataset
========================================================


```r
data <- as.matrix(mtcars)

p <- plot_ly(x=colnames(data), y=rownames(data), z = data, type = "surface") %>% layout(margin = list(l=120))

htmlwidgets::saveWidget(as_widget(p), "p.html")
```

Display Plot
========================================================

<iframe src="./p.html" width=100% height=100% allowtransparency="true"> </iframe>


Setup data for user interaction - 1
========================================================

Add Package "Tigris" to be able to get US State/County FIPS


```r
library(tigris)
library(dplyr)
```

Load midwest dataset

```r
mh <- midwest
```

Load FIPS data, remove the string " County" from the county column


```r
fips <- fips_codes
fips <- data.frame(lapply(fips, function(x) { gsub(" COUNTY", "", toupper(x)) }))
```

Setup data for user interaction - 2
========================================================

Using DPLYR join the Midwest Dataset to Fips to create a dataset that contains the full FIPS code to make easier to generate a map based on FIPS. Use Stringr to remove all white space from the FullFips column to prevent errors.


```r
library(dplyr)
library(stringr)

mhfips <- mh %>% inner_join(fips, by = c("state" = "state", "county" = "county"))

mhfips$fullfips <- as.character(
  str_replace_all(
    paste(mhfips$state_code, mhfips$county_code), 
    regex("\\s*"), ""))

library(rjson)

url <- 'https://raw.githubusercontent.com/plotly/datasets/master/geojson-counties-fips.json'

counties <- rjson::fromJSON(file=url)
```

Setup data for user interaction - 3
========================================================


```r
mhfips$hover <- with(mhfips, paste(state, '<br>', 
                           county, '<br>', 
                           "white", popwhite, 
                           "black", popblack
                           ))

# give state boundaries a white border
l <- list(color = toRGB("white"), width = 2)
```

Setup data for user interaction - 4
========================================================


```r
# specify some map projection/options
g <- list(
  scope = 'usa',
  projection = list(type = 'albers usa'),
  showlakes = TRUE,
  lakecolor = toRGB('white')
)

fig <- plot_ly()
```

Setup data for user interaction - 5
========================================================


```r
fig <- fig %>% add_trace(type="choropleth", z = mhfips$poptotal, 
    geojson=counties, text = mhfips$hover, locations = mhfips$fullfips,
    colorscale="Viridis", marker=list(line=list(width=0))
  )
fig <- fig %>% colorbar(title = "Population per county")
fig <- fig %>% layout(title = 'Midwest US State Dataset<br>(Hover for breakdown)', geo = g)

htmlwidgets::saveWidget(as_widget(fig), "fig.html")
```

Map Interaction
========================================================

<iframe src="./fig.html" width=100% height=100% allowtransparency="true"> </iframe>
