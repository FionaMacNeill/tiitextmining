## Text Mining the Anonymous Marking Audit Trail

### Author: Fiona MacNeill | Date: 06/06/2019

## Learning Technologies Scenario:
You have access to data from the Anonymous Marking Audit Excel file which can you download from the Turnitin administrative account. You would like to find out if there are any patterns in this text-based data and whether there are any misconceptions about anonymous marking.


## About this tutorial
This data was simulated by Fiona MacNeill on 6 May 2019. This tutorial takes you from the original Microsoft Excel file, all the way to a finished word cloud. Tweaking will be needed to get the text-based data how you want it, but the good news is that the parameters that you need are in this tutorial. Certain sections are commented out as you may or may not need them (just remove the "#" in order to try them out). 

Turnitin allows you to export the Anonymous Marking Audit Trail data if you have access to the administrative account and for information about how to do that, please take a look at the tutorial from help.turnitin.com: <https://help.turnitin.com/feedback-studio/moodle/direct-v2/administrator/anonymous-marking/viewing-an-anonymous-marking-audit-trail.htm?Highlight=turn%20off%20anonymous%20marking> || [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/).

I made use of the following tutorial from STHDA for learning text mining skills and I owe the authors a debt of immense gratitude: <http://www.sthda.com/english/wiki/text-mining-and-word-cloud-fundamentals-in-r-5-simple-steps-you-should-know>

## Getting started

If you are new to using RStudio you will need to get setup first by...

1. Installing R from the R Archive:

    i. Pick a mirror close-by (geographically speaking), e.g in UK – [University of Bristol](https://www.stats.bris.ac.uk/R/), [Imperial College London](https://cran.ma.imperial.ac.uk/)

    ii. Mac tip: make sure that you check the MD5 hash and SHA hash match. You can do this quickly and easily in terminal. As shown in this video: <https://youtu.be/HHdrIlHS2-4>

    iii. [Mac only] XQuartz Install – information about this is provided at R Archive above.

2. Install RStudio: again do check the MD5. You can get the installer here – <https://www.rstudio.com/products/rstudio/download/#download>


## 1. Install the packages that you need

There are a number of packages that you will need. The first four packages below are only for importing, extracting and transforming the data from the original Excel spreadsheet. You can do this more simply by copying the "Reasons" column into a text editor like TextWrangler and then saving it as a .txt file. Using this option would allow you to skip step 3, but there is something to be said for understanding how to transform your data.

```{r setup, eval=FALSE, include=FALSE}

install.packages("readxl") # for reading a native MS Excel file
install.packages("dplyr") # for extracting the data from the MS Excel file
install.packages("tm") # for text mining
install.packages("SnowballC") # for text clean up
install.packages("wordcloud") # word-cloud generator 
install.packages("RColorBrewer") # colours
install_formats() # supports the use of rio
install.packages("rio") # for exporting columns as text files

```

## 2. Load the libraries for the packages that you have installed

Now that you have installed the requisite packages you need to load the libraries so that they are available in your R environment. If you copy the complete directory, *'wordcloud_tutorial'* to your computer and open the RMarkdown file *'wordcloud_tutorial.Rmd'* from within it, you should not need to change the file path information in Steps 1-4.

```{r turnitin data setup - load your libraries and convert the data}

library("tm")
library("SnowballC")
library("wordcloud")
library("RColorBrewer")
library("ggplot2")
library("readxl")
library("dplyr")
library("rio")

# Copy the directory to your computer
reasons.all <- read_excel("wordcloud_tutorial_files/reasons_tutorial.xlsx")

# Find out about the columns in the spreadsheet
names(reasons.all) 

```
## 3. Pull the "Reasons" column data and export it as text (.txt)

We want to pull the "Reason" column and convert it to a new data frame with a single variable and then export it as a text file so that we can create our text corpus ([What is a text corpus? - Wikipedia](https://en.wikipedia.org/wiki/Text_corpus)).

**Tip**:
You can take a look at the text file itself prior to completing step 4 and remove anything odd (such as strange characters that are hard to remove automatically) with a "find and replace" option in your text editor. It is worth going back to this if you see odd patterns in your data in the summary provided by 'inspect' at the end of step 5. Having made changes and saved your text file you can then re-run chunks 4-8 with your cleansed text file.

**Side note**: if you wanted to extract only unique reasons so that you have a concise list as a separate data.frame then you can quickly use the distinct option from the dplyr package - remove "#" to use it. This will keep only 'distinct' data.


```{r pull out the "reason" variable and convert it to text}

reason <- pull(reasons.all, var="Reason") %>% data.frame %>% export(file = "wordcloud_tutorial_files/reasons_demo.txt")

#reason.individual <- distinct(reasons.all, Reason)
#reason.individual

```

## 4. Create the text corpus 

This is using the base tools.

```{r create a corpus from the text file}

filePath <- "wordcloud_tutorial_files/reasons_demo.txt"

# Read the lines in the text document.
reason <- readLines(filePath) 

# Create a text corpus based on our document - we are telling our existing variable 'reason' to become this new corpus.
reason <- Corpus(VectorSource(reason)) 

```

## 5. Clean up the text in your text corpus

You will need to do quite a bit of clean up on the text, particularly if you are working on several years worth of data. Thankfully the tm library is here to help and you can use the various transformations that it offers to complete clean-up tasks in-bulk!

```{r cleaning up characters and text}

# Change to lower case
reason <- tm_map(reason, content_transformer(tolower))

# Remove numbers
reason <- tm_map(reason, removeNumbers)

# Remove english common stopwords
#reason <- tm_map(reason, removeWords, stopwords("en"))

# Remove punctuation
reason <- tm_map(reason, removePunctuation)

# Eliminate extra white spaces
reason <- tm_map(reason, stripWhitespace)

# Using inspect will allow you to see how the text-based data has changed based on your transformations
inspect(reason)

```
```{r further clean-up}

# Remove some words that don't make sense 
reason <- tm_map(reason, removeWords, c("marking", "provide", "anonymous", "set", "need", "poss", "possible", "submission", "students", "student's", "the", "for", "this", "not", "module", "name", "with", "&quot;"))

```

## 6. When you are ready create the TermDocumentMatrix

This is about creating table or matrix containing the count information for each word. 

**a).** Create the TermDocumentMatrix which is essentially applies a list of controls for manipulating your text corpus. Delete "#" before *inspect(dtm)* to see what happens.

**b).** "m <- as.matrix(dtm)" converts your TermDocumentMatrix into an actual matrix. As in a table type thing with counts per word. Delete "#" before *View(m)* to see what happens. 

**c).** In this step we are now sorting our matrix from highest to lowest. Delete "#" before *View(v)* to see what happens.

**d).** Now we want to create a fresh data.frame with the data that we have mangled so that we can visualise it. Delete "#" before *View(d)* to see what happens.


```{r create the Term Document Matrix}

# create the term matrix based on your corpus
# Step a).
dtm <- TermDocumentMatrix(reason)
#inspect(dtm)

# Step b).
m <- as.matrix(dtm)
#View(m)

# Step c).
v <- sort(rowSums(m), decreasing = TRUE)
#View(v)

# Step d).
d <- data.frame(word = names(v), freq=v)
#View(d)

# This is just for information and returns the top 30 terms in your new data.frame for your reference
head(d, 30)
```


## 7. Check if your word cloud is working

I would draw your attention to the min.freq and max.words below. So the min.freq being 1 in this case means that we are including words that are mentioned once. In a larger dataset you are going to want to set this threshold much higher; I recommend a minimum of 10. The max.words option restricts the number of words included in your cloud. You definitely need to use this if you are working with several years-worth of data. Also, do not worry if the preview below shows an error that saying that content cannot be fit on the page, the final export file in step 8 will display all the word cloud content.

```{r build the word cloud}

set.seed(11249)
wordcloud(words = d$word, 
          freq = d$freq, 
          min.freq = 1, 
          max.words = 200, 
          random.order = FALSE, 
          rot.per = 0.35, 
          color=brewer.pal(8, "Dark2"))

# See vignette for RColorBrewer for different colour options by running the help command below. Remove the "#" before help.

# help("RColorBrewer")

```

## 8. Export your visualisation as a high-quality PNG

This is a good option to get a high-quality image for adding to reports and presentations.

```{r Build your word cloud again and export it as an image}

# Set your image settings
png("wordcloud_demo_export.png", units="in", width=6, height=6, res=300)

# Create the plot
set.seed(11249)
wordcloud(words = d$word, 
          freq = d$freq, 
          min.freq = 1, 
          max.words = 200, 
          random.order = FALSE, 
          rot.per = 0.35, 
          color=brewer.pal(8, "Dark2"))

# Action the image production - the image should go into your directory
dev.off()
```



