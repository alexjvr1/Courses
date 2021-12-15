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

dplyr has several functions that can help us combine datasets by row, by column, by intersecting datasets, and much more. 

See here for the cheatsheet: https://www.rstudio.com/wp-content/uploads/2015/02/data-wrangling-cheatsheet.pdf


```
lep <- bind_rows(pop1, pop2, pop3)

summary(lep)
```





Question: 

The data is not in tidy format. Can you name two things we need to change? 





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


```
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


##### Issue 2: Column headers are values, not variable names



Let’s take a closer look at all those column headers like G0.05, N0.2 and P.15.

- Limiting nutrient. This has six possible values: glucose (G), ammonium (N), sulfate (S), phosphate (P), uracil (U) or leucine (L).

- Growth rate: A number, ranging from .05 to .3. .05 means slow growth (the yeast were being starved hard of that nutrient) while .3 means fast growth. (Technically, this value measures the dilution rate from the chemostat).

- Expression level. These are the values currently stored in those columns, as measured by the microarray. (Note that the paper already did some pre-processing and normalization on these values, which we’re ignoring here).


The rules of tidy data specify that each variable forms one column, and this is not even remotely the case- we have 36 columns when we should have 3. That means our data is trapped in our column names. If you don’t see why this is a problem, consider: how would you put growth rate on the x-axis of a graph? How would you filter to look only the glucose condition?

Luckily, the tidyr package has a solution ready. The documentation for gather notes (emphasis mine):

Gather takes multiple columns and collapses into key-value pairs, duplicating all other columns as needed. You use gather() when you notice that you have columns that are not variables.

Hey, that’s us! So let’s apply gather as our next step:

```
cleaned_data <- original_data %>%
  separate(NAME, c("name", "BP", "MF", "systematic_name", "number"), sep = "\\|\\|") %>%
  mutate_each(funs(trimws), name:systematic_name) %>%
  select(-number, -GID, -YORF, -GWEIGHT) %>%
  gather(sample, expression, G0.05:U0.3)
```

Now when we view it, it looks like:

```
View(cleaned_data)
```

Notice that the dataset no longer consists of one-row-per-gene: it’s one-row-per-gene-per-sample. This has previously been called “melting” a dataset, or turning it into “long” format. But I like the term “gather”: it shows that we’re taking these 36 columns and pulling them together.

One last problem. That sample column really contains two variables, nutrient and rate. We already learned what to do when we have two variables in one column: use separate:


```
cleaned_data <- original_data %>%
  separate(NAME, c("name", "BP", "MF", "systematic_name", "number"), sep = "\\|\\|") %>%
  mutate_each(funs(trimws), name:systematic_name) %>%
  select(-number, -GID, -YORF, -GWEIGHT) %>%
  gather(sample, expression, G0.05:U0.3) %>%
  separate(sample, c("nutrient", "rate"), sep = 1, convert = TRUE)
```  

This time, instead of telling separate to split the strings based on a particular delimiter, we told it to separate it after the first character (that is, after G/P/S/N/L/U). We also told it convert = TRUE to tell it that it should notice the 0.05/0.1/etc value is a number and convert it.

Take a look at those six lines of code, a mini-sonnet of data cleaning. Doesn’t it read less like code and more like instructions? (“First we separated the NAME column into its five parts, and trimmed each. We selected out columns we didn’t need…”) That’s the beauty of the %>% operator and the dplyr/tidyr verbs.



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



#### Additional information

Read more: https://ggplot2.tidyverse.org






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


### More about lists

https://www.tutorialspoint.com/r/r_lists.htm

https://www.datamentor.io/r-programming/list/

### Troubleshooting

#### Common errors

The most common errors you'll run in to will be either 1) setting your path correctly and typos  (xx not found/does not exist error), or 2) syntax errors in R. 

Have a look at this post for some common error messages and solutions: https://github.com/noamross/zero-dependency-problems/issues/7


#### Finding help

Forums like StackExchange are very helpful when troubleshooting errors. The trick is to learn how to phrase your search to find the correct answers. The more specific your search, the more likely you are to find helpful answers. 


### Commenting on your code and keeping notes

Remember to add comments to all your code as you go. Delete code that didn't work, and/or make a note of things that you might have done wrong and how to correct it. It could also be useful to add in links to the solution (e.g. a helpful StackExchange post you found).

Keeping a digital lab book (e.g. on GitHub) is good practice and helpful for storing your projects and code. 


