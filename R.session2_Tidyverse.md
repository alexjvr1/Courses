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

- Know how to tidy data 

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


##### Background: gene expression in starvation

(Information from here: http://varianceexplained.org/r/tidy-genomics/)

Through the process of gene regulation, a cell can control which genes are transcribed from DNA to RNA- what we call being “expressed”. (If a gene is never turned into RNA, it may as well not be there at all). This provides a sort of “cellular switchboard” that can activate some systems and deactivate others, which can speed up or slow down growth, switch what nutrients are transported into or out of the cell, and respond to other stimuli. A gene expression microarray lets us measure how much of each gene is expressed in a particular condition. We can use this to figure out the function of a specific gene (based on when it turns on and off), or to get an overall picture of the cell’s activity.

Brauer 2008 used microarrays to test the effect of starvation and growth rate on baker’s yeast (*S. cerevisiae*, a popular model organism for studying molecular genomics because of its simplicity). Basically, if you give yeast plenty of nutrients (a rich media), except that you sharply restrict its supply of one nutrient, you can control the growth rate to whatever level you desire (we do this with a tool called a chemostat). For example, you could limit the yeast’s supply of glucose (sugar, which the cell metabolizes to get energy and carbon), of leucine (an essential amino acid), or of ammonium (a source of nitrogen).

"Starving" the yeast of these nutrients lets us find genes that:

- Raise or lower their activity in response to growth rate. 

- Growth-rate dependent expression patterns can tell us a lot about cell cycle control, and how the cell responds to stress.

- Respond differently when different nutrients are being limited. These genes may be involved in the transport or metabolism of those nutrients.


We'll import the data submitted by the authors, and see what we need to do to get the data in a tidy format. 
```
#import data
original_data <- read_delim("http://varianceexplained.org/files/Brauer2008_DataSet1.tds", delim = "\t")

#What do the data look like? 
dim(original_data)

head(original_data)

summary(original_data)

colnames(original_data)
```


Question: 

1) The data is not in tidy format. Can you name two things we need to change? 





#### 2. Tidy the data


##### Issue 1: Multiple variables are stored in a single column. 


```
original_data$NAME[1:3]

## [1] "SFB2       || ER to Golgi transport || molecular function unknown || YNL049C || 1082129"          
## [2] "|| biological process unknown || molecular function unknown || YNL095C || 1086222"                
## [3] "QRI7       || proteolysis and peptidolysis || metalloendopeptidase activity || YDL104C || 1085955"

```


The details of each of these fields isn’t annotated in the paper, but we can figure out most of it. It contains:

- Gene name e.g. SFB2. Note that not all genes have a name.

- Biological process e.g. “proteolysis and peptidolysis”

- Molecular function e.g. “metalloendopeptidase activity”

- Systematic ID e.g. YNL049C. Unlike a gene name, every gene in this dataset has a systematic ID.3

- Another ID number e.g. 1082129. I don’t know what this number means, and it’s not annotated in the paper. Oh, well.


Having all give of these in the same column is very inconvenient. For example, if I have another dataset with information about each gene, I can’t merge the two. Luckily, the tidyr package provides the separate function for exactly this case.

```
cleaned_data <- original_data %>%
  separate(NAME, c("name", "BP", "MF", "systematic_name", "number"), sep = "\\|\\|")
```


We simply told separate what column we’re splitting, what the new column names should be, and what we were separating by. What does the data look like now?


```
view(cleaned_data)

```

Just like that, we’ve separated one column into five.

Two more things. First, when we split by ||, we ended up with whitespace at the start and end of some of the columns, which is inconvenient:

```
head(cleaned_data$BP)
## [1] " ER to Golgi transport "        " biological process unknown "  
## [3] " proteolysis and peptidolysis " " mRNA polyadenylylation* "     
## [5] " vesicle fusion* "              " biological process unknown "
```


We’ll solve that with dplyr’s mutate_each, along with the built-in trimws (“trim whitespace”) function.



cleaned_data <- original_data %>%
  separate(NAME, c("name", "BP", "MF", "systematic_name", "number"), sep = "\\|\\|") %>%
  mutate_each(funs(trimws), name:systematic_name)
```

Finally, we don’t even know what the number column represents (if you can figure it out, let me know!) And while we’re at it, we’re not going to use the GID, YORF or GWEIGHT columns in this analysis either. We may as well drop them, which we can do with dplyr’s select.

```
cleaned_data <- original_data %>%
  separate(NAME, c("name", "BP", "MF", "systematic_name", "number"), sep = "\\|\\|") %>%
  mutate_each(funs(trimws), name:systematic_name) %>%
  select(-number, -GID, -YORF, -GWEIGHT)
  ```


##### Issue 2: 







#### 3. Change column names

Let's look at the column names again: 
```
colnames(original_data)

```

The column names are not very informative at the moment. To change one column name: 
```



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


