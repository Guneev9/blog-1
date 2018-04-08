---
title: Exploratory Data Analysis
author: ''
date: '2018-02-28'
slug: exploratory-data-analysis
categories:
  - data analysis
tags:
  - data analysis
  - rstats
  - statistics
output:
  #pdf_document: default
  html_document: default
---

```{r setup, include=FALSE, message=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

```{r}
library(ggplot2)
library(extraDistr)
library(tidyverse)
#library(plotly)
```

The data source is obtained from https://www.cdc.gov/nchs/nsfg/nsfg_cycle6.htm
This contains fixed width files, and stata dictionaries consisting of columns for the data files
```{r}
#helper function to parse Stata dictionary
dct.parser <- function(dct, includes = c("StartPos", "StorageType", "ColName", 
                                         "ColWidth", "VarLabel"),
                       preview = FALSE) {
  temp <- readLines(dct)
  temp <- temp[grepl("_column", temp)]
  
  if (isTRUE(preview)) {
    head(temp)
  } else {
    possibilities <- c("StartPos", "StorageType", 
                       "ColName", "ColWidth", "VarLabel")
    classes <- c("numeric", "character", "
                 character", "numeric", "character")
    pattern <- c(StartPos = ".*\\(([0-9 ]+)\\)",
                 StorageType = "(byte|int|long|float|double|str[0-9]+)",
                 ColName = "(.*)",
                 ColWidth = "%([0-9.]+)[a-z]+",
                 VarLabel = "(.*)")
    
    mymatch <- match(includes, possibilities)
    
    pattern <- paste(paste(pattern[mymatch], 
                           collapse ="\\s+"), "$", sep = "")    
    
    metadata <- setNames(lapply(seq_along(mymatch), function(x) {
      out <- gsub(pattern, paste("\\", x, sep = ""), temp)
      out <- gsub("^\\s+|\\s+$", "", out)
      out <- gsub('\"', "", out, fixed = TRUE)
      class(out) <- classes[mymatch][x] ; out }), 
                         possibilities[mymatch])
    
    implicit.dec <- grepl("\\.[1-9]", metadata[["ColWidth"]])
    if (any(implicit.dec)) {
      message("Some variables may need to be corrected for implicit decimals. 
              Try 'MESSAGES(output_from_dct.parser)' for more details.")
      metadata[["Decimals"]] <- rep(NA, length(metadata[["ColWidth"]]))
      metadata[["Decimals"]][implicit.dec] <-
        as.numeric(gsub("[0-9]+\\.", "", 
                        metadata[["ColWidth"]][implicit.dec]))
      metadata[["ColWidth"]] <- floor(as.numeric(metadata[["ColWidth"]]))
    }
    
    metadata[["ColName"]] <- make.names(
      gsub("\\s", "", metadata[["ColName"]]))
    
    metadata <- data.frame(metadata)
    
    if ("StorageType" %in% includes) {
      metadata <- 
        within(metadata, {
          colClasses <- ifelse(
            StorageType == "byte", "raw",
            ifelse(StorageType %in% c("double", "long", "float"), 
                   "numeric", 
                   ifelse(StorageType == "int", "integer",
                          ifelse(substr(StorageType, 1, 3) == "str", 
                                 "character", NA))))
        })
    }
    if (any(implicit.dec)) {
      attr(metadata, "MESSAGE") <- c(sprintf("%s", paste(
        "Some variables might need to be corrected for implicit decimals. 
        A variable, 'Decimals', has been created in the metadata that
        indicates the number of decimal places the variable should hold. 
        To correct the output, try (where your stored output is 'mydf'): 
        
        lapply(seq_along(mydf[!is.na(Decimals)]), 
        function(x) mydf[!is.na(Decimals)][x]
        / 10^Decimals[!is.na(Decimals)][x])
        
        The variables in question are:
        ")), sprintf("%s", metadata[["ColName"]][!is.na(metadata[["Decimals"]])]))
            class(attr(metadata, "MESSAGE")) <- c(
                "MESSAGE", class(attr(metadata, "MESSAGE")))
        }
        attr(metadata, "original.dictionary") <- 
            c(dct, basename(dct))
        metadata
    }
}
```


We can read the coulmns from 2002FemPreg.dct and use those columns to import the data from the fixed width file 2002FemPreg.dat
```{r}
femPreg2002columns <- dct.parser('~/Documents/CodeWork/ThinkStats2/code/2002FemPreg.dct')
femPreg2002 <- read.fwf('~/Documents/CodeWork/ThinkStats2/code/2002FemPreg.dat', widths = femPreg2002columns$ColWidth, col.names = femPreg2002columns$ColName)
```

Taking a look at the data
```{r}
head(femPreg2002)
```

We can see a lot of missing values. We'll clean the data for the columns that we want to analyze.

## Transformation

1. agepreg contains the mother's age at the end of the pregnancy. In the data file, agepreg is encoded as an integer number of centiyears. So the first line divides each element of agepreg by 100, yielding a floating-point value in years.

2. birthwgt_lb and birthwgt_oz contain the weight of the baby, in pounds and ounces, for pregnancies that end in live birth. In addition it uses several special codes:
  97 NOT ASCERTAINED
  98 REFUSED
  99 DONT KNOW
Special values encoded as numbers are dangerous because if they are not
handled properly, they can generate bogus results, like a 99-pound baby. Assuming that a baby can't be generally more than 20 lb at birth, we will replace all other values with NA, as they are NOT ASCERTAINED(97),  REFUSED(98), DONT KNOW(99), or invalid values.
Similarly, the age of father has these similar special codes, which we will replace by NA
```{r}
cleanFemPreg <- function(data){
  # mother's age is encoded in centiyears; convert to years
  data['agepreg'] <-  data['agepreg']/100.0
  
  # birthwgt_lb contains at least one bogus value (51 lbs)
  # replace with NaN
  data$birthwgt_lb[data$birthwgt_lb > 20] <- NA
  
  # replace 'not ascertained', 'refused', 'don't know' with NA
  na_vals = c(97, 98, 99)
  data$birthwgt_oz[data$birthwgt_oz %in% na_vals] <- NA
  data$hpagelb[data$hpagelb %in% na_vals] <- NA
  
  # birthweight is stored in two columns, lbs and oz.
  # convert to a single column in lb
  data['totalwgt_lb'] <- data$birthwgt_lb + (data$birthwgt_oz / 16.0)
  
  return (data)
}
```

```{r}
femPregCleaned <- cleanFemPreg(femPreg2002)
```

### Validation
One way to validate data is to compute basic statistics and compare them with published results. For example, the NSFG codebook includes tables that summarize each variable. Here is the table for outcome, which encodes the outcome of each pregnancy:
value     label         Total
1         LIVE BIRTH        9148
2         INDUCED ABORTION  1862
3         STILLBIRTH        120
4         MISCARRIAGE       1921
5         ECTOPIC PREGNANCY 190
6         CURRENT PREGNANCY 352

```{r}
femPreg2002 %>%
  group_by(outcome) %>%
  summarise(Total = length(outcome))
```

Comparing the results with the published table, it looks like the values in
outcome are correct. Similarly, here is the published table for birthwgt_lb
value     label             Total
.         INAPPLICABLE      4449
0-5       UNDER 6 POUNDS    1125
6         6 POUNDS          2223
7         7 POUNDS          3049
8         8 POUNDS          1889
9-95      9 POUNDS OR MORE  799

```{r}
femPreg2002 %>%
  group_by(birthwgt_lb) %>%
  summarise(Total = length(birthwgt_lb))
```

The counts for 6, 7, and 8 pounds check out, and if you add up the counts
for 0-5 and 9-95, they check out, too. But if you look more closely, you will
notice one value that has to be an error, a 51 pound baby! This has been cleaned in the cleanFemPreg function.

### Interpretation
To work with data effectively, you have to think on two levels at the same time: the level of statistics and the level of context.
As an example, let's look at the sequence of outcomes for a few respondents.
This example looks up one respondent and prints a list of outcomes for her
pregnancies:
```{r}
CASEID = 10229
femPregCleaned %>%
  filter(caseid==CASEID) %>%
  .$outcome
```
The outcome code 1 indicates a live birth. Code 4 indicates a miscarriage; that is, a pregnancy that ended spontaneously, usually with no known medical cause.

Statistically this respondent is not unusual. Miscarriages are common and there are other respondents who reported as many or more. But remembering the context, this data tells the story of a woman who was pregnant six times, each time ending in miscarriage. Her seventh and most recent pregnancy ended in a live birth. If we consider this data with empathy,
it is natural to be moved by the story it tells.

Each record in the NSFG dataset represents a person who provided honest answers to many personal and difficult questions. We can use this data to answer statistical questions about family life, reproduction, and health. At the same time, we have an obligation to consider the people represented by the data, and to afford them respect and gratitude.