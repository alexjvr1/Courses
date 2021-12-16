# R Session 2 - Data wrangling and visualisation

Data will never be collected or arrive in exactly the right format to be analysed or visualised. The goal is to get data into a tidy format so that they can be easily used as input for analyses or plots. 


### Objectives:

- Recognise un-tidy data 

- Create tidy data

- Combine data frames

- Know what the five main verbs are in dplyr (Select, Filter, Arrange, Mutate, Summarise)

- Know how to use the five dplyr verbs to manipulate your data

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




#### 2. Tidy the data


##### Issue: Multiple variables are stored in a single column. 


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

These data are now in a tidy format. 


#### *Extra:* 

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
```


#### 3. Add or remove columns







#### 4. Change column names

Let's look at the column names again: 
```
colnames(original_data)

```

The column names are not very informative at the moment. To change one column name: 
```



```



#### 5. Substitute text in a column





#### 6. Combine datasets with dplyr





#### 7. Filter data










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


