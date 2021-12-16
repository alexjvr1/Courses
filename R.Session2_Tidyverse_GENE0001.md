# R Session 2 - Data wrangling and visualisation

Data will never be collected or arrive in exactly the right format to be analysed or visualised. The goal is to get data into a tidy format so that they can be easily used as input for analyses or plots. 


### Objectives:

- Recognise un-tidy data 

- Create tidy data

- Combine data frames

- Know how to rearrange a data frame from wide to long format (gather and spread functions) 

- Know how to use dplyr to Select, Filter, Arrange, Mutate, and Summarise your data. 

- Know how to draw a basic plot using ggplot2



### Notes

This lecture should serve as starting point for learning about data wrangling and visualisation. Use the links in the "Extra" section below to learn more, and practice on your own datasets. 

Learn Tidyverse first and "Do powerful things quickly": http://varianceexplained.org/r/teach-tidyverse/


## Part 1: Data wrangling: Turning un-tidy data into tidy data


What are Tidy data?  

1. Every column is a variable.

2. Every row is an observation.

3. Every cell is a single value.

Read more about tidy data: https://cran.r-project.org/web/packages/tidyr/vignettes/tidy-data.html



#### 1. Import data and R packages


Tidyverse is a suite of R packages for wrangling and visualising data. They all use the same design philosophy, grammer and data structure. 

Today we'll be using *dplyr* and *ggplot2*, but you can find out more about the other packages here: https://www.tidyverse.org/packages/


```
#install.packages("tidyverse")
library(tidyverse)

```


##### Background: Genome-wide genetic diversity in butterfly species

Climate change and habitat destruction has resulted in sudden and extreme population size reductions in many insects (and other species). However, some species seem to be benefiting from climate change, with rapid range expansions and increased population sizes. 

We test if genetic diversity has changed in a buttefly species that has expanded its range in the UK since the 1980s. We generated whole genome sequence data for museum samples collected ~1900 (POP1), and modern samples from the same site (POP2), and the expanding range edge of the species (POP3). We estimated genetic diversity, and allele frequencies in 1Mb windows across the genome for each individual. 


We'll import the data, and see what we need to do to get the data in a tidy format. 
```
#import data. 

#Usually we'd do this - Read files in to a list: 
files <- list.files(pattern="GENE0001.txt") 
myfiles <- lapply(files, read.table, header=T)
##See more on lists in the "Extra section below"


##But, we want three separate files so that we can learn how to combine them using dplyr: 

pop1 <- read.table("POP1.GENE0001.txt", header=T)
pop2 <- read.table("POP2.GENE0001.txt", header=T)
pop3 <- read.table("POP3.GENE0001.txt", header=T)


#What do the data look like? 
dim(pop1)

head(pop1)

summary(pop1)

colnames(pop1)
colnames(pop2)
colnames(pop3)
```


Questions: 

Do the column names match?

How would we combine the data? 


Answer: 

dplyr has several functions that can help us combine datasets by row, by column, by intersecting datasets, and much more. 

See here for the cheatsheet: https://www.rstudio.com/wp-content/uploads/2015/02/data-wrangling-cheatsheet.pdf

```
lep <- bind_rows(pop1, pop2, pop3)

summary(lep)

head(lep)
```



Question: 

The data is not in tidy format. Can you name one thing we need to change? 




#### 2. Separate columns


ISSUE: Multiple variables are stored in a single column. 


```
lep$ChrRegion[1:3]

[1] LR761647.1_0_1000000       LR761647.1_1000000_2000000
[3] LR761647.1_2000000_3000000
56 Levels: LR761647.1_0_1000000 ... LR761649.1_9000000_10000000

```


This column contains three sets of data: 

- Chromosome name. eg. LR761647.1

- Start of window

- End of window

(Remember that diversity and nucleotide frequencies are estimated in 1Mb windows) 

Tidy format dictates that each column should contain a single variable, and each row a single observation. So we should split these: 

```
lep_cleaned <- lep %>%
  separate(ChrRegion, c("Chr", "start", "stop"), sep = "_")
```


We simply told separate what column we’re splitting, what the new column names should be, and what we were separating by. What does the data look like now?


The "%>%" is a pipe. This makes the preceding variable the input for the following commands. 

```
head(lep_cleaned)

```




#### 3. Change data from wide to long format

Our data are violating another tidy rule: Our dataframe reports nucleotide frequencies separately for all four nucleotides. What if we wanted a single column with all the nucleotide frequencies? 

Hint: see the gather() and spread() functions in tidyr

A possible solution: 
```
nuc_lep <- gather(lep, key="Nucleotide", "Frequency", pi.A., pi.C., pi.G., pi.T.)

head(nuc_lep)
```

And if we want to separate them again: 
```
head(spread(nuc_lep, Nucleotide, Frequency))

#Spread writes each unique Nucleotide:Frequency combination to a separate row.
```

#### *Bonus:* 

What if we want to rename the nucleotides in our "Nucleotide" column to remove the pi. part of the name?

```
nuc_lep$Nucleotide <- gsub("\." "" nuc_lep$Nucleotide)
nuc_lep$Nucleotide <- gsub("pi" "" nuc_lep$Nucleotide)

head(nuc_lep)
```


#### 4. Select or remove specific columns


select() lets you choose specific columns. 

This is very useful for subseting your data, or for removing columns. For example, now that we have extracted the nucleotide frequency information from our dataset, we don't need these columns in our dataset anymore. Let's remove them: 


```
#Using a pipe
lep_cleaned2 <- lep_cleaned %>% select(-pi.A., -pi.G., -pi.T., -pi.C.)
head(lep_cleaned2)

#Or without a pipe
lep_cleaned2 <- select(lep_cleaned, -pi.A., -pi.G., -pi.T., -pi.C.)
head(lep_cleaned2)
```

But select is even more useful when we use helper functions like starts_with(), ends_with(), and contains(). 

For example, let's find all the nucleotide frequency columns: 
```
head(lep_cleaned %>% select(contains("pi")))

#OR
head(select(lep_cleaned, contains("pi")))
```

Or only the Chr and midpos columns
```
head(lep_cleaned %>% select(Chr, midpos))

#OR
head(select(lep_cleaned, Chr, midpos))

```

These are quite versatile tools for subsetting the data by column. 





#### 5. Subset or select by row

filter() allows you to select data by row. You can use any valid arguments regarding row values or content. 

e.g. Let's choose only rows from one of the populations: 

```
unique(lep_cleaned$Pop)
lep_cleaned %>% filter(Pop=="MODC")
```

Or we might be interested in data that have been sequenced at > 2X: 
```
summary(lep_cleaned$depth)

summary(lep_cleaned %>% filter(depth>2))

```

Or we can filter using multiple variables: 

e.g. let's find all the museum (MUS) samples that have >2X coverage
```
lep_cleaned %>% filter(Pop=="MUS" && depth>2)

```

No data! So now we start to know something about our dataset... 



#### 6. Add new columns with mutate()

mutate() allows you to add columns to your data based on data already available to you. 

e.g. We have the start and end bp position of each genomic window. How would we calculate the midpoint of the window and add this as a new column. (Hint, this has already been done and the column is called "midpos)"

```
head(mutate(lep_cleaned, midpos=start+(stop-start)/2))

```

Oh no! We've run into an error: 
```
Error: Problem with `mutate()` column `midpos.new`.
ℹ `midpos.new = start - stop`.
✖ non-numeric argument to binary operator
```

This suggests that R is interpreting the numbers as characters: 
```
str(lep_cleaned)

```

We can easily fix this: 
```
lep_cleaned$stop <- as.numeric(lep_cleaned$stop)
lep_cleaned$start <- as.numeric(lep_cleaned$start)

#And recalculate the midpos
head(mutate(lep_cleaned, midpos=start+(stop-start)/2))
```



#### 7. summarise data

summarise() calculates summary statistics per column. 

e.g. we can calculate the mean sequencing depth like this: 
```
summarise(lep_cleaned, mean(depth))
```


But we've just seen that at least one of the populations (MUS) has an extremely low sequencing depth. So we really need to see the depth per population. 

We can use the group_by() function to group data together within summarise to look at any particular subset of the data. e.g.
```
summarise(group_by(lep_cleaned,Pop), mean(depth))

# A tibble: 3 × 2
  Pop   `mean(depth)`
  <fct>         <dbl>
1 MUS            1.07
2 MODC           4.53
3 MODE           4.60

```

Question: What is the mean theta between populations?




## Part 2: Data Visualisation



### Objectives: 

- Know the different options needed to generate a plot with ggplot 

- Know how to draw a graph with points in ggplot

- Know how to draw a histogram in ggplot

- How to group within a plot in ggplot

- Create a pdf of your figure



#### Basic plot: 









## Extra: 

### More information on dplyr and a bit on tidyverse

There are lots of really useful free resources available to learn more about dplyr and other tidyverse packages. Here are a few: 

Data Carpentry has a whole section on analysing data in R. Here is the dplyr section: https://datacarpentry.org/R-genomics/04-dplyr.html

A very clean and simple intro to dplyr from Sean Anderson: https://seananderson.ca/2014/09/13/dplyr-intro/

David Robinson has a couple of posts on why it is important to start using tidyverse as early as possible when learning R: http://varianceexplained.org/r/tidy-genomics/ 

A free course from DataCamp - Introduction to TidyVerse: https://campus.datacamp.com/courses/introduction-to-the-tidyverse

I can't overstate how useful these R cheatsheets are: https://www.rstudio.com/resources/cheatsheets/

### More on joining datasets together

https://www.guru99.com/r-dplyr-tutorial.html


### More information on ggplot

Another great post from Data Carpentry: https://datacarpentry.org/semester-biology/materials/ggplot/

A nicely written example using genomic data: https://genviz.org/module-02-r/0002/03/01/introToggplot2/ 

And a second genomics example: https://ucsdlib.github.io/workshops/ggplot.html

Tidyverse: https://ggplot2.tidyverse.org


### More about lists

https://www.tutorialspoint.com/r/r_lists.htm

https://www.datamentor.io/r-programming/list/


### More about pipes: %>%

https://www.datacamp.com/community/tutorials/pipe-r-tutorial


### Troubleshooting

#### Common errors

The most common errors you'll run in to will be either 1) setting your path correctly and typos  (xx not found/does not exist error), or 2) syntax errors in R. 

Have a look at this post for some common error messages and solutions: https://github.com/noamross/zero-dependency-problems/issues/7


#### Finding help

Forums like StackExchange are very helpful when troubleshooting errors. The trick is to learn how to phrase your search to find the correct answers. The more specific your search, the more likely you are to find helpful answers. 


### Commenting on your code and keeping notes

Remember to add comments to all your code as you go. Delete code that didn't work, and/or make a note of things that you might have done wrong and how to correct it. It could also be useful to add in links to the solution (e.g. a helpful StackExchange post you found).

Keeping a digital lab book (e.g. on GitHub) is good practice and helpful for storing your projects and code. 


