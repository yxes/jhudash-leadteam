---
output:
  md_document:
    variant: markdown_github
---

<!-- README.md is generated from README.Rmd. Please edit that file -->



# CDCwonderr

The extra 'r' is for the language.

## Motivation

My team at the JHU DaSH wanted to correlate statistics regarding lead poisoning 
with mental health problems or general "malfeasance". What we settled upon was 
to use statistics of suicides/homicides committed, with [this dataset](ftp://ftp.cdc.gov/pub/Health_Statistics/NCHS/Datasets/DATA2010/State_data_tables/). 

However, we found that there were discrepancies between some xls and 
csv files that were supposed to contain the same data. Moreover, we encountered 
some difficulties in interpreting/comparing between some of the demographics included, 
and not all of those demographic categories were included for each focus condition 
of a spreadsheet, or even between spreadsheets. 

To achieve our end goal, then, we needed to: 
- parse the data from the original xls files to preserve data integrity/quality 
- be able to select certain demographics for which we could extract statistics of interest

In the end, we managed to write some functions to simultaneously accomplish these 
goals, and decided to bundle them up in a package so that perhaps it would be 
useful to others. For our end product, see the Shiny presentation in this repo.


## Example




```r
suppressPackageStartupMessages({
  library(devtools)
  library(dplyr, warn=F)
  library(readxl)
  library(readxl)
})  

# all the xls files have the first line data-free
suici <- read_excel("data-raw/NFOCUS18ST.XLS", skip = 1)
#> DEFINEDNAME: 20 00 00 01 0b 00 00 00 01 00 00 00 00 00 00 07 3b 00 00 01 00 01 00 00 00 ff 00 
#> DEFINEDNAME: 20 00 00 01 0b 00 00 00 01 00 00 00 00 00 00 07 3b 00 00 01 00 01 00 00 00 ff 00 
#> DEFINEDNAME: 20 00 00 01 0b 00 00 00 01 00 00 00 00 00 00 07 3b 00 00 01 00 01 00 00 00 ff 00 
#> DEFINEDNAME: 20 00 00 01 0b 00 00 00 01 00 00 00 00 00 00 07 3b 00 00 01 00 01 00 00 00 ff 00
suici %>% glimpse
#> Observations: 1,863
#> Variables: 34
#> $        Objective                                        (chr) "     ...
#> $ 1998                                                    (chr) "    "...
#> $ Raw                                                     (chr) "   ",...
#> $ error                                                   (chr) "  ", ...
#> $ 1999                                                    (chr) "    "...
#> $ Raw                                                     (chr) "    "...
#> $ error                                                   (chr) "    "...
#> $ 2000                                                    (chr) "    "...
#> $ Raw                                                     (chr) "    "...
#> $ error                                                   (chr) "    "...
#> $ 2001                                                    (chr) "    "...
#> $ Raw                                                     (chr) "     ...
#> $ error                                                   (chr) "    "...
#> $ 2002                                                    (chr) "    "...
#> $ Raw                                                     (chr) "    "...
#> $ error                                                   (chr) "    "...
#> $ 2003                                                    (chr) "    "...
#> $ Raw                                                     (chr) "    "...
#> $ error                                                   (chr) "    "...
#> $ 2004                                                    (chr) "    "...
#> $ Raw                                                     (chr) "    "...
#> $ error                                                   (chr) "    "...
#> $ 2005                                                    (chr) "    "...
#> $ Raw                                                     (chr) "    "...
#> $ error                                                   (chr) "    "...
#> $ 2006                                                    (chr) "    "...
#> $ Raw                                                     (chr) "    "...
#> $ error                                                   (chr) "    "...
#> $ 2007                                                    (chr) "    "...
#> $ Raw                                                     (chr) "   ",...
#> $ error                                                   (chr) "  ", ...
#> $ 2008                                                    (chr) "    "...
#> $ Raw                                                     (chr) "   ",...
#> $ error                                                   (chr) "  ", ...
```

Here, we see that there are a few difficulties. One is that column names are not 
all distinct. Another is that some have empty spaces (e.g., the "Objective" column);
some other files even have empty columns (with footnote numbers in the excel file, 
which probably aren't useful in an analysis). Also (and you can confirm this yourself), the end of the file doesn't have data we're interested in. To fix these problems, 
we can use the following functions: 


```r
load_all(".") # load functions in the package
#> Loading CDCwonderr
suici2 <- suici %>% fix_col_names %>% rm_tail
suici2 %>% glimpse
#> Observations: 1,857
```

The "Objective" column contains state names, names of demographic variables, and 
health conditions as well. A combination of "add_*" and "add_*_names" functions 
can be used to add an entire column with values corresponding to the the 
respective row. For example, 


```r
suici2 %>% 
  add_states %>% 
  add_state_names %>% 
  glimpse
```

should indicate that the manipulations added a column which contains the state name
corresponding to the statistics in each row. 

For our purposes, we wrapped up all the functionality to transform a string with 
the filename, to a tidy dataset ready to incorporate into our analysis, with 
the get_gender_comp function.


```r
homicides <- get_gender_comp("data-raw/NFOCUS18ST.XLS")
homicides %>% glimpse
```

(Sidenote: it seems, as of this writing, that there is some bug with dplyr, 
although I haven't isolated the reason yet...)

## Acknowledgements

Thanks to [Sean Davis](http://watson.nci.nih.gov/~sdavis/) for help with the idea and R package development, and [Steve Wells](http://stevenwells.com/) for help with regex expressions.

