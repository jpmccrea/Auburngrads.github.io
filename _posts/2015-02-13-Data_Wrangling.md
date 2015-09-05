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

<br>
<font size="4">The original post for this tutorial can be accessed at
[http://rpubs.com/bradleyboehmke/data_wrangling](http://rpubs.com/bradleyboehmke/data_wrangling)
</font>
<br>

---
## <center>Introduction</center>
---

### <u>Analytic Process</u>
Analysts tend to follow 4 fundamental processes to turn data into understanding, knowledge & insight:

1. **Data manipulation**
2. Data visualization
3. Statistical analysis/modeling
4. Deployment of results

This tutorial will focus on **<u>data manipulation</u>**

<br><br>

### <u>Data Manipulation</u>

> It is often said that 80% of data analysis is spent on the process of cleaning and preparing the data. (Dasu and Johnson, 2003)

Well structured data serves two purposes:

1. Makes data suitable for software processing whether that be mathematical functions, visualization, etc.
2. Reveals information and insights

Hadley Wickham's paper on [Tidy Data](http://vita.had.co.nz/papers/tidy-data.html) provides a great explanation behind the concept of "tidy data"

<br>

<img src="TidyData.png", height="300px", width="800px" />

<br>
<br>



