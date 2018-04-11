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

Create full-scale scores and bind with earlier dataset

```
HPIscores <- scoreItems(keys, HPIDat.fin)
HPI.scales <- as.data.frame(HPIscores$scores)
HPI.scales <- cbind(HPIDat.fin[,7:13], HPI.scales)
```

### Step 5. Stepwise Confirmatory Factor Analysis (SCFA)
------

Create a list of item names
`item_names <- names(HPIDat.fin.tes[,65:270])`

Iteratively carry out the SCFA and eliminate items with the lowest loading until the target length (e.g., 5 items) is reached.

```
##Round 1

Scale_m1 <- ' Scale =~ item1 + item2 + item3 + item4 + item5 + item6 + item7 + item8 + item9 + item10'
scale_fit1 <- cfa(Scale_m1, HPIDat.fin.tra[,item_names])
scale_sum1 <- summary(scale_fit1, fit.measure=T, rsq=T, standardized=T)
inspect(scale_fit1,what="std")$lambda[order(inspect(scale_fit1,what="std")$lambda, decreasing =F), ]

#Decide to drop item1

##Round 2
Scale_m2 <- ' Scale =~ item2 + item3 + item4 + item5 + item6 + item7 + item8 + item9 + item10'
scale_fit2 <- cfa(Scale_m2, HPIDat.fin.tra[,item_names])
scale_sum2 <- summary(scale_fit2, fit.measure=T, rsq=T, standardized=T)
inspect(scale_fit2,what="std")$lambda[order(inspect(scale_fit2,what="std")$lambda, decreasing =F), ]

#Decide to drop item2

#Repeat until five items remaining
```

### Step 6. Genetic Algorithm (GA)
------

Provide full scale scores and items in matrices

```
Scale.items <- as.matrix(HPIDat.fin.tra[,c('item1','item2','item3','item4','item5','item6','item7','item8','item9','item10')])
Scale.scale <- as.matrix(HPIDat.fin.tra$scalescore)
```

Genetic abbreviate to five items

```
GAA.Scale.5 <- GAabbreviate(Scale.items, Scale.scale, itemCost = .05, maxItems = 15,
                            popSize = 50, maxiter = 200, run = 100, crossVal = F, impute = T)
```

### Step 6. Ant Colony Optimization (ACO)
------

Note the ACO takes a long time to run (upwards of 24 hours). To compensate, we ran all analyses using Amazon EC2 online cloud server space. To run the algorithm, R must be able to access a new .csv sheet which is updated with results as the analyses run. There are automated procedures to synchronize amazon cloud services with dropbox.  

```
Items.adj <- colnames(Adj.items)

covariates <- c("RReliability", "RManager", "RSales")
 
nitems <- 10 # number of items in the short scale
iter <- 60   # number of iterations
ants <- 150  # number of iterations (start with a iter/ants ratio of 2/3)
evaporation <- 0.90
```
                 
