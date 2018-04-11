### Step 1. Install and load required packages for data manipulation, graphing, and psychometric/SEM analyses.
------
`libs <- c("grid", "psych", "tidyverse", "gridExtra", "GAabbreviate", "lavaan", "careless", "stringr", "corrr")`

`lapply(libs, require, character.only = T)`

### Step 2. Filter dataset and eliminate careless respondents
------

Subset data to only full cases for relevant variables

```Rouge
HPIDat.red <- HPIDat %>%
  select(hjobfamily, JobCode, Age, Gender_New, Ethnicity,
         rval:RManager, Valid:hpi206) %>%
  filter(complete.cases(.))
```
Clean and remove aberrant respondens

#long-string index
```Rouge
long <- longstring(HPIDat.red[,65:270])
sum(long > 30)
HPIDat.red %>% dplyr::filter(long, sum == 1)
```
#Identify # pairs with r > .40
```Rouge
HPI.item.r <- correlate(HPIDat.red[,65:270])
HPI.df <- stretch(HPI.item.r, na.rm = TRUE)
sum(HPI.df %>% filter(r > .40))
synonym <- psychsyn(HPIDat.red[,65:270], critVal = .40)
sum(synonym > .15)
```

### Step 3. Filter dataset and eliminate careless respondents
------
