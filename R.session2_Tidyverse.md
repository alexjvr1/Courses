# R Session 2 - Data wrangling and visualisation

Data will never be collected or arrive in exactly the right format to be analysed or visualised. The goal is to get data into a Tidy format so that they can be easily used as input for analyses or plots. 

What is Tidy data?  

1. Every column is a variable.

2. Every row is an observation.

3. Every cell is a single value.

Read more about tidy data: https://cran.r-project.org/web/packages/tidyr/vignettes/tidy-data.html


### Notes

This course is inspired by posts by David Robinson: http://varianceexplained.org/r/tidy-genomics/ 

and the DataCamp Introduction to TidyVerse course: https://campus.datacamp.com/courses/introduction-to-the-tidyverse

Useful Cheatsheets: https://www.rstudio.com/resources/cheatsheets/



## Part 1: Data wrangling: Turning un-tidy data into tidy data


### Objectives:

- Know how to clean data 

- Know how to change column names

- Know how to add and remove columns from data

- Know how to substitute data or text in columns

- How to combine datasets together

- How to group data

- How to filter data



#### 1. Import data and R packages


Tidyverse is a suite of R packages for wrangling and visualising data. They all use the same design philosophy, grammer and data structure. 

Today we'll be using *dplyr* and *ggplot2*, but you can find out more about the other packages here: https://www.tidyverse.org/packages/


```
#install.packages("tidyverse")
library(tidyverse)

```



#### Extra

Exercises to try: 



Some good tutorials: 



## Part 2: Reshape Data

### Objectives: 

- Know the difference between "Long" and "Wide" format

- Know how to transform data from Wide to Long




Filtering, merging, grouping, aggregating data - Introduction to common packages such as dplyr and data.table

Visualising data with ggplot





#### Extra

Exercises to try: 



Some good tutorials: 




## Part 3: Visualisation


### Objectives: 

- Know the different options needed to generate a plot with ggplot 

- Know how to draw a graph with points in ggplot

- Know how to draw a histogram in ggplot

- How to group within a plot in ggplot

- Create a pdf of your figure



#### Extra

Exercises to try: 



Some good tutorials: 





## Extra: 

### Troubleshooting

#### Common errors

The most common errors you'll run in to will be either 1) setting your path correctly, or 2) syntax errors in R. 

Have a look at this post for some common error messages and solutions: https://github.com/noamross/zero-dependency-problems/issues/7


#### Finding help

Forums like StackExchange are very helpful when troubleshooting errors. The trick is to learn how to phrase your search to find the correct answers. The more specific your search, the more likely you are to find helpful answers. 


### Commenting on your code and keeping notes

Remember to add comments to all your code as you go. Delete code that didn't work, and/or make a note of things that you might have done wrong and how to correct it. It could also be useful to add in links to the solution (e.g. a helpful StackExchange post you found).

Keeping a digital lab book (e.g. on GitHub) is good practice and helpful for storing your projects and code. 


