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

```Rouge
##long-string index

long <- longstring(HPIDat.red[,65:270])
sum(long > 30)
HPIDat.red %>% dplyr::filter(long, sum == 1)

##Identify # pairs with r > .40

HPI.item.r <- correlate(HPIDat.red[,65:270])
HPI.df <- stretch(HPI.item.r, na.rm = TRUE)
sum(HPI.df %>% filter(r > .40))
synonym <- psychsyn(HPIDat.red[,65:270], critVal = .40)
sum(synonym > .15)

##Add carelessness indices to HPIDat dataset

HPIDat.red <- cbind(HPIDat.red, long, synonym, outliers)
HPIDat.fin <- HPIDat.red %>% filter(long <= 30, synonym > .15)
```

### Step 3. Set Seed and Randomly Split into Training and Validation Sample (80/20 split)
------

Set seed and randomly sample

```Rouge
set.seed(99); HPIDat.fin.tra <- sample_frac(HPIDat.fin, .80)
sid <- as.numeric(rownames(HPIDat.fin.tra))
HPIDat.fin.tes <- HPIDat.fin[-sid,]
```

Check and convert class of items to binary-ordered (HPI uses dichotomous responses)

```
varTable(HPIDat.red)
HPIDat.fin.tes[,65:270] <- lapply(HPIDat.fin.tes[,65:270], ordered)
HPIDat.fin.tra[,65:270] <- lapply(HPIDat.fin.tra[,65:270], ordered)
```

### Step 4. Create scoring keys
------

For all subsequent syntax, we provide an example of (a) creating a 5-item scale only, using (b) a  10-item scale with generic labels to protect the proprietary nature of the HPI key. Note all syntax examples below were re-run for 7 different scales x 3 item length sets  (5-item, 10-item, 15-item) x 3 analyses (Stepwise Confirmatory, Genetic, Ant Colony) for 63 seperate analyses. Syntax and iterative updating need to be adjusted to reflect your study purposes, scale length, and unique scoring keys. 

Create a key list for scoring

```
key.list <- list(Scale=c('item1','item2','item3','item4','item5','item6','item7','item8','item9','item10'))
keys <- make.keys(10, key.list, item.labels = colnames(HPIDat.fin))
```

Create seperate item and full-scale score matrices

```
HPIItems <- as.matrix(HPIDat.fin[,65:270])
HPIscores <- scoreItems(keys, HPIDat.fin)
HPI.scales <- as.data.frame(HPIscores$scores)
HPI.scales <- cbind(HPIDat.fin[,7:13], HPI.scales)
```

Create seperate item and full-scale score matrices (for later validation)

```

```

                 
