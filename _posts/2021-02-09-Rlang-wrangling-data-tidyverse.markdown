---
layout: post
title:  "Rlang: Wrangling data with tidyverse"
date:   2021-02-09 19:00:00 +0100
categories: rlang tidyverse data
---

In this post I describe the tools I use to manipulate data using R language and often the tidyverse package. 

# Useful documentation
[**R for Data Science**][r4ds] by Hadley Wickham and Gerrett Grolemund - VERY USEFUL

[Advance R][adv-r] by Hadley Wickham

[Tidyverse style guide][style-tidyverse] by Hadley Wickham : good practices for writing R code

[R markdown][r-markdown] by Yihui Xie, J. J. Allaire, Garrett Grolemund - if you use R-markdown to write reports.  

[style-tidyverse]: https://style.tidyverse.org/
[adv-r]: https://adv-r.hadley.nz/
[r4ds]: https://r4ds.had.co.nz/
[r-markdown]: https://bookdown.org/yihui/rmarkdown/

# Source file
Source file so that their content can be used (functions defined in an external script for instance):
```R
source(file= "~/Desktop/FrenchNewQuotatives/loadData.R")
```


# Load DataFrame and clean
To load a csv we can use read.csv, read_csv, read.delim... (not sure about the differences)
```R
dataframe <- read.csv('<path>')
# or 
dataframe <- read.delim('<path>', quote="") #equivalent to import dataset with option 'Quote' to 'None'
```
`%>%` is used in the tidyverse to pass the result of a function to the next so it avoids creating intermediate functions.
For instance, the following will take the dataframe created earlier, *filter* and eliminate entries (rows) for which the value of 'ColumnA' is 'NA'; *select* and keep only the values of 'ColumnB' and 'ColumnC'; from the resulting dataframe, eliminate duplications to obtain *unique* rows; and *mutate* to add a new column based on another and modify an existing one.

```R
newDF <- dataframe %>% 
            filter(!is.na(ColumnA))%>%
            select(ColumnB, ColumnC)%>%
            unique()%>%
            mutate(NewCol = ColumnB*2,
            	   ColumnC = !ColumnC)

# dataframe :
# ColumnA | ColumnB | ColumnC | ColumnD |
#    1    |    11   |   TRUE  |    NA   |
#    2    |    12   |   TRUE  |    b    |
#    NA   |    12   |   FALSE |    c    |
#    3    |    12   |   TRUE  |    d    |
#    4    |    13   |   FALSE |    e    |

# After filter: 
# ColumnA | ColumnB | ColumnC | ColumnD |
#    1    |    12   |   TRUE  |    NA   |
#    2    |    12   |   TRUE  |    b    |
#    3    |    12   |   TRUE  |    d    |
#    4    |    12   |   FALSE |    e    |

# After select: 
# ColumnB | ColumnC |
#    12   |   TRUE  |
#    12   |   TRUE  |
#    12   |   TRUE  |
#    12   |   FALSE |

# After unique: 
# ColumnB | ColumnC |
#    11   |   TRUE  |
#    12   |   TRUE  |
#    13   |   FALSE |

# After mutate: 
# ColumnB | ColumnC | NewCol |
#    11   |   FALSE |   22   |
#    12   |   FALSE |   24   |
#    13   |   TRUE  |   26   |

```

The `subset` function is similar to `filter` ([Stack Overflow discussion][so-subset-filter])



[so-subset-filter]: https://stackoverflow.com/questions/39882463/difference-between-subset-and-filter-from-dplyr


# Define function
If I have to do the same portion of script over and over, I can create a function, which I can then use multiple times.

```R
myNewFunction <- function(){
	do some stuff
}

createNewVariable <- function(){
	do some stuff
	createdVariable <- "new variable"
	return(createdVariable)
}

functionWithArgs(myArg)<- fucntion(){
	do some stuff depending on myArg
	# optional return
}

# use the new function
myNewFunction()

myNewVariable <- createNewVariable()

functionWithArgs(myNewVariable)

```

#### Define function for mutate
If you want to use a custom function for a mutate (or any case that will apply the same function on many rows), a "normal" function will not work for some reason, so we need to use `sapply` (or `mapply` in case of multiple arguments functions). 

```R
# apply the same function on a dataframe column (a vector)
accent_mean<-function(x){sapply(x, function(x){ #need sapply to use 'mutate' or 'within'...
  df%>%filter(region1.quantised==x)%>%pull(accentFort)%>% mean()
})}

df$regionMeanAccent <- accent_mean(df$region1.quantised)

# multiple arguments function
meanDuration<- function(x,y){df.filesDur%>%filter(speakerCode==x, File == y)%>%pull(Duration)%>% mean()}

df.filesDur <- df.filesDur%>% mutate(durationMean = mapply(meanDuration, speakerCode, File))

```


# Combining data
#### Merge
Two dataframe can be merged with the function [merge][merge-doc]. 
It adds up the columns, based on one or several columns row values. 
Options can be added to specify by which columns to merge, if the columns names are not identical in each dataframe, how to manage columns with the same name, what to do with rows that are missing, etc.

```R
dataframe <- df1 %>% merge(df2)
#df1: 
# ColumnB | ColumnC |
#    11   |   TRUE  |
#    12   |   TRUE  |
#    13   |   FALSE |
# df2:
# ColumnA | ColumnB | ColumnD |
#    1    |    11   |    NA   |
#    2    |    12   |    b    |
#    3    |    13   |    d    |

# dataframe: 
# ColumnA | ColumnB | ColumnC | ColumnD |
#    1    |    11   |   TRUE  |    NA   |
#    2    |    12   |   TRUE  |    b    |
#    3    |    13   |   TRUE  |    d    |

```


#### Bind-rows
[bind_rows][bind_all-doc] allows to combine two dataframes rows.


```R
dataframe <- df1 %>% bind_rows(df2)
#or/equivalent
dataframe <- bind_rows(df1, df2)
# df1
# ColumnB | ColumnC |
#    11   |   TRUE  |
#    12   |   TRUE  |
#    13   |   FALSE |
#df2
# ColumnB | ColumnC |
#    14   |   TRUE  |
#    15   |   FALSE |

#dataframe
# ColumnB | ColumnC |
#    11   |   TRUE  |
#    12   |   TRUE  |
#    13   |   FALSE |
#    14   |   TRUE  |
#    15   |   FALSE |

```


[merge-doc]: https://www.rdocumentation.org/packages/base/versions/3.6.2/topics/merge
[bind_all-doc]: https://www.rdocumentation.org/packages/dplyr/versions/0.4.0/topics/rbind_all

# Pivot dataframe
[pivot_longer][pivot_longer-doc] reduces the number of columns (the information persists in multiple rows). 
The reverse exists with p[ivot_wider][pivot_wider-doc].

```R
  df.new <- df.old %>% pivot_longer(c(ColumnA, ColumnB, ColumnD), names_to = "ColumnID", values_to = "Values")

# df.old: 
# ColumnA | ColumnB | ColumnC | ColumnD |
#    1    |    11   |   TRUE  |    NA   |
#    2    |    12   |   TRUE  |    b    |
#    3    |    13   |   FALSE |    d    |

# df.new
# ColumnA | ColumnID | Values |
#    1    | ColumnB  |   11   |
#    1    | ColumnC  |  TRUE  |
#    1    | ColumnD  |   NA   |
#    2    | ColumnB  |   12   |
#    2    | ColumnC  |  TRUE  |
#    2    | ColumnD  |   b    |
#    3    | ColumnB  |   13   |
#    3    | ColumnC  |  FALSE |
#    3    | ColumnD  |   d    |


```

[pivot_longer-doc]: https://www.rdocumentation.org/packages/tidytable/versions/0.5.7/topics/pivot_longer.
[pivot_wider-doc]: https://www.rdocumentation.org/packages/tidytable/versions/0.5.8/topics/pivot_wider.