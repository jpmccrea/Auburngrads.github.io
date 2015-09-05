---
layout: post
title: Data Processing with dplyr & tidyr
---

<style>
div {
    text-align: justify;
    text-justify: inter-word;
}
</style>

<br><br>
## <center>Introduction</center>
---

### <u>Analytic Process</u>
Analysts tend to follow 4 fundamental processes to turn data into understanding, knowledge & insight:

1. **Data manipulation**
2. Data visualization
3. Statistical analysis/modeling
4. Deployment of results

This tutorial will focus on **<u>data manipulation</u>**

<br>

### <u>Data Manipulation</u>

> *It is often said that 80% of data analysis is spent on the process of cleaning and preparing the data.* (Dasu and Johnson, 2003)

Well structured data serves two purposes:

1. Makes data suitable for software processing whether that be mathematical functions, visualization, etc.
2. Reveals information and insights

Hadley Wickham's paper on [Tidy Data](http://vita.had.co.nz/papers/tidy-data.html) provides a great explanation behind the concept of "tidy data"

<br>

<img src="TidyData.png", height="300px", width="800px" />

<br>


### <u>Why Use tidyr & dplyr</u>
- Although many fundamental data processing functions exist in R, they have been a bit convoluted to date and have lacked consistent coding and the ability to easily *flow* together &#8594; leads to difficult-to-read nested functions and/or *choppy* code.
- [R Studio](http://www.rstudio.com/) is driving a lot of new packages to collate data management tasks and better integrate them with other analysis activities &#8594; led by [Hadley Wickham](https://twitter.com/hadleywickham) & the R Studio [team](http://www.rstudio.com/about/) &#8594; [Garrett Grolemund](https://twitter.com/StatGarrett), [Winston Chang](https://twitter.com/winston_chang), [Yihui Xie](https://twitter.com/xieyihui) among others.
- As a result, a lot of data processing tasks are becoming packaged in more cohesive and consistent ways &#8594; leads to:
    - More efficient code
    - Easier to remember syntax
    - Easier to read syntax

<br>

#### <u>Packages Utilized</u>

```r
library(dplyr)
library(tidyr)
```

**<u>tidyr</u> and <u>dplyr</u> packages provide fundamental functions for <u>Cleaning, Processing, & Manipulating Data</u>**

* tidyr
    + <a href="#gather">**`gather()`**</a>
    + <a href="#spread">**`spread()`**</a>
    + <a href="#separate">**`separate()`**</a>
    + <a href="#unite">**`unite()`**</a>
* dplyr
    + <a href="#select">**`select()`**</b></a>
    + <a href="#filter">**`filter()`**</a>
    + <a href="#group">**`group_by()`**</a>
    + <a href="#summarise">**`summarise()`**</a>
    + <a href="#arrange">**`arrange()`**</a>
    + <a href="#join">**`join()`**</a>
    + <a href="#mutate">**`mutate()`**</a>

<br>

<a href="#">Go to top</a>

<br>
