# Genes or Ants
Code to reproduce metaheuristic algorithms for 2018 SIOP poster on scale shortening with Genes or Ants

## Project and Data Description
This repository includes R syntax for the 2018 SIOP poster: "Gens or Ants: Genes or Ants: Meta-heuristic algorithms for scale length optimization." 
The following aspects of the project can be accessed.

- Full submission in `'. item-level correlations (N = 62,997), descriptive statistics, and code for simulating a multivariate binary-item dataset in the `SimData-Syn.md` file.
- Simulated dataset of the 42-item Hogan Development Survey analog of the Dark Triad in `SimData.HDS.zip`
- R syntax for replicating network diagrams and analyses in `DarkNet.Syn`
- Full submission in `Dark Side Derailers as Causal Systems-2017 Poster.docx.` Item content removed to protect propietary key. 

## Requirements

- R > 3.0 (possibly earlier versions will work)

The following R packages should be installed and loaded to run all the needed data manipulations, figures, and algorithms

```
install.packages("grid")
install.packages("psych")
install.packages("tidyverse")
install.packages("gridExtra")
install.packages("GAabbreviate")
install.packages("lavaan")
install.packages("careless")
install.packages("corrr")
```

## References

The inspiration, theory, and code for the following project comes from active applications of these techniques by Gabriel Olaru, Oliver Wilhelm, and Ulrich Schroeders in personality psychology. The primary work providing the logic, structure, and open source syntax for applying these algorithms comes from the following citation:

- [Schroeders, U., Wilhelm, O., & Olaru, G. (2016). Meta-heuristics in short scale construction: ant colony optimization and genetic algorithm. PloS one, 11(11), e0167110.](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0167110)

## License

This data and analysis are being prepared for publication and review. The repository is copyrighted and no permission is granted to re-analyse the data for publication or reuse any of the materials. After publication, code and data will ultimately be GPL-2; all measurement scales retain existing licence.
    
Any publication that re-uses the data should cite this publication.
