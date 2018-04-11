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

### Step 7. Ant Colony Optimization (ACO)
------

Note the ACO takes a long time to run (upwards of 24 hours). To compensate, we ran all analyses using Amazon EC2 online cloud server space. To run the algorithm, R must be able to access a new .csv sheet which is updated with results as the analyses run. There are automated procedures to synchronize amazon cloud services with dropbox.  

```
Items.adj <- colnames(Scale.items)

covariates <- c("RReliability", "RManager", "RSales") ##Insert criterion or performance ratings here
 
nitems <- 10 # number of items in the short scale
iter <- 60   # number of iterations
ants <- 150  # number of iterations (start with a iter/ants ratio of 2/3)
evaporation <- 0.90

#FUNCTION START:
antcolony <- function(evaporation, Items.adj, nitems, iter, ants, summaryfile, summaryfile2){

  best.pheromone <- 0
  best.so.far.pheromone <- 0
  item.vector <- Items.adj
  
  #creates the table of initial pheromone levels.
  include <- rep(2, length(Items.adj))
  
  #puts initial best solution (all items selected).
  best.so.far.solution <- include
  
  #creates a list to store factors.
  selected.items <- Items.adj
  
  #starts counting the interations
  count <- 1
  
  #starts counting the continuous runs regardless of result
  run <- 1
  
  #defines initial solution
  previous.solution <- include
  set.seed(99)
  
  #starts loop through iterations.
  while (count <= iter) {
    
    #sends a number of ants per time.
    ant  <- 1
    while (ant <= ants) {
      
      mod.1dim <- NULL
      
      #selects the items for a short form for the factor
      positions <- is.element(item.vector, Items.adj)
      prob <- include[positions]/ sum(include[positions])
      items <- sample(Items.adj, size = nitems, replace = F, prob)
      
      #stores selected items 
      selected.items <- items
      
      # specifies CFA model
      mod.1dim <-  paste("lv =~", paste(selected.items, collapse = " + "))
      
      #creates a 0/1 vector of the same length of the long form indicating
      #whether an item was selected or not for the short form.
      select.indicator <- is.element(item.vector, selected.items)
      notselect.indicator <- (select.indicator == FALSE)
      
      # estimates CFA model
      fit.1dim <- cfa(model = mod.1dim, 
                      data = HPIDat.fin.tra, 
                      ordered = Items.adj,
                      estimator = 'WLSMV',
                      std.lv = TRUE)
      
      # save the complete lavaan ouput
      out <- capture.output(summary(fit.1dim, fit.measures=TRUE, standardized=TRUE))
      
      # reads the fit statistics (CFI, RMSEA, Chi^2)
      CFI <- fitMeasures(fit.1dim, "cfi.scaled")[[1]]
      RMSEA <- fitMeasures(fit.1dim, "rmsea.scaled")[[1]]
      Chi <- fitMeasures(fit.1dim, "chisq.scaled")[[1]]
      
      # optimization #1: model fit
      phi.CFI <- 1/(1+exp(36-40*CFI)) #modified
      phi.RMSEA <- 1-(1/(1+exp(5-100*RMSEA)))
      phi.fit <- (phi.CFI + phi.RMSEA)/2
      
      # optimization #2: factor saturation
      std <- standardizedSolution(fit.1dim)
      min.lam <- min(std[1:nitems, 4])
      mean.lam <- mean(std[1:nitems, 4])
      all.lam <- paste(std[1:nitems, 4], collapse="/")
      sum.lam <- sum(std[1:nitems, 4])^2
      sum.the <- sum(1-std[1:nitems, 4]^2)
      omega <- sum.lam/(sum.lam + sum.the)
      phi.lam <- 1/(1+exp(7-10*omega))
      
      # optimization #3: sensitivity
      mean.diff <- mean(psych::describe(HPIDat.fin.tra[, selected.items])[[3]])
      phi.sens <- -5*(mean.diff - .50)^2+1
      
      # optimization #4: differnces in correlation to covariates
      HPIDat.fin.tra$adj.short <- rowSums(HPIDat.fin.tra[, selected.items])
      cor.covariates <- cor(HPIDat.fin.tra[,c("radj", "adj.short", covariates)], use="p")
      cor.diff.cova.1 <- cor.covariates[1,3] - cor.covariates[2,3]
      cor.diff.cova.2 <- cor.covariates[1,4] - cor.covariates[2,4]
      cor.diff.cova.3 <- cor.covariates[1,5] - cor.covariates[2,5]
      
      max.cor.diff <- max(abs(c( cor.diff.cova.1,  cor.diff.cova.2,  cor.diff.cova.3)))
      phi.cor <- 1/-(1+exp(5-100*max.cor.diff))+1
      
      # max f(x), optimize = maximize
      pheromone <- phi.fit + phi.lam + phi.cor + phi.sens
      
      #saves information about the selected items and the model fit they generated.
      fit.info <- matrix(c(select.indicator, run, count, ant, 
                           Chi, CFI, RMSEA, phi.CFI, phi.RMSEA, phi.fit, 
                           min.lam, mean.lam, all.lam, phi.lam, 
                           cor.diff.cova.1, cor.diff.cova.2, cor.diff.cova.3, 
                           max.cor.diff, phi.cor, mean.diff, phi.sens,
                           pheromone, round(include,2)), 1)
      
      write.table(fit.info, file = summaryfile, append = T,
                  quote = F, sep = ";", row.names = F, col.names = F)
      
      #adjust count based on outcomes and selects best solution.
      if (pheromone >= best.pheromone) {
      
        # updates solution.
        best.solution <- select.indicator
        best.pheromone <- pheromone
        
        # updates best model fit
        best.Chi <- Chi
        best.RMSEA <- RMSEA
        best.CFI <- CFI
        best.phi.CFI <- phi.CFI
        best.phi.RMSEA <- phi.RMSEA
        best.phi.fit <- phi.fit
        
        # updates best factor loadings
        best.min.lam <- min.lam
        best.mean.lam <- mean.lam
        best.all.lam <- all.lam
        best.phi.lam <- phi.lam
        
        # updates best correlation differences
        best.cor.diff.cova.1 <- cor.diff.cova.1
        best.cor.diff.cova.2 <- cor.diff.cova.2
        best.cor.diff.cova.3 <- cor.diff.cova.3
        best.max.cor.diff <- max.cor.diff
        best.phi.cor <- phi.cor
        
        # updates best distribution overlap
        best.mean.diff <- mean.diff
        best.phi.sens <- phi.sens
      } 
      
      #Move to next ant.
      ant <- ant + 1
      
      #ends loop through ants.
    }
    
    #adjusts pheromone only if the current pheromone is better than the previous.
    if (best.pheromone > best.so.far.pheromone) {
     
      #implements pheromone evaporation.
      include <- include * evaporation
      
      #adjusts the pheromone levels.
      include.pheromone <- best.solution * best.pheromone * run * 0.2
      
      #updates pheromone.
      include <- include + include.pheromone
      
      # updates best so far solution and pheromone.
      best.so.far.solution <- best.solution
      best.so.far.pheromone <- best.pheromone
      best.so.far.Chi <- best.Chi
      best.so.far.RMSEA <- best.RMSEA
      best.so.far.CFI <- best.CFI
      best.so.far.phi.CFI <- best.phi.CFI
      best.so.far.phi.RMSEA <- best.phi.RMSEA
      best.so.far.phi.fit <- best.phi.fit
      
      best.so.far.min.lam <- best.min.lam
      best.so.far.mean.lam <- best.mean.lam
      best.so.far.all.lam <- best.all.lam
      best.so.far.phi.lam <- best.phi.lam
      
      best.so.far.cor.diff.cova.1 <- best.cor.diff.cova.1
      best.so.far.cor.diff.cova.2 <- best.cor.diff.cova.2
      best.so.far.cor.diff.cova.3 <- best.cor.diff.cova.3
      best.so.far.max.cor.diff <- best.max.cor.diff
      best.so.far.phi.cor <- best.phi.cor
      
      best.so.far.mean.diff <- best.mean.diff
      best.so.far.phi.sens <- best.phi.sens
      
      #re-starts count.
      count <- 1
      
      #end if clause for pheromone adjustment.
    } else {
      
      #advances count.
      count <- count + 1
    }
    
    #ends loop.
    run <- run + 1
  }
  
  title.final.solution <- matrix(c("Items", "Chi", "CFI", "RMSEA", "phi.CFI", "phi.RMSEA", "phi.fit", 
                                   "min.lam", "mean.lam", "all.lam", "phi.lam", 
                                   "cor.diff.cova.1", "cor.diff.cova.2", "cor.diff.cova.3", 
                                   "max.cor.diff", "phi.cor",
                                   "mean.diff", "phi.sens",
                                   "pheromone", item.vector), 1)
  
  write.table(title.final.solution, file = summaryfile2, append = T, 
              quote = F, sep = ";", row.names = F, col.names = F)
  
  # Compile a matrix with the final solution.
  final.solution <- matrix(c(sum(nitems), best.so.far.Chi, best.so.far.CFI, best.so.far.RMSEA, best.so.far.phi.CFI, 
                             best.so.far.phi.RMSEA, best.so.far.phi.fit, 
                             best.so.far.min.lam, best.so.far.mean.lam, best.so.far.all.lam, best.so.far.phi.lam, 
                             best.so.far.cor.diff.cova.1, best.so.far.cor.diff.cova.2, best.so.far.cor.diff.cova.3,
                             best.so.far.max.cor.diff, best.so.far.phi.cor,
                             best.so.far.mean.diff, best.so.far.phi.sens,
                             best.so.far.pheromone, best.so.far.solution), 1)
  
  write.table(final.solution, file = summaryfile2, append = T,
              quote = F, sep = ";", row.names = F, col.names = F)
  
  return(best.so.far.solution)
  
}

# run the function
short <- antcolony(evaporation, Items.adj, nitems, iter, ants, summaryfile, summaryfile2)         
```
