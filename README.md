# GenesorAnts
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

The following R packages should be installed and loaded to simulate the item-level data and run network analyses. 

```
install.packages("bindata")
install.packages("psych")
install.packages("qgraph")
install.packages("parcor")
```

## References

The inspiration, theory, and code for the following project comes from the highly active [Psychosystems](http://psychosystems.org/) group. Notable citations and and methodological tutorials with regard to personality and psychopathology constructs as networks of mutually reinforcing behaviors include:

- [Borsboom, D., & Cramer, A. O. (2013). Network analysis: an integrative approach to the structure of psychopathology. Annual review of clinical psychology, 9, 91-121.](https://www.ncbi.nlm.nih.gov/pubmed/23537483)

- [Cramer, A. O., Sluis, S., Noordhof, A., Wichers, M., Geschwind, N., Aggen, S. H., ... & Borsboom, D. (2012). Dimensions of normal personality as networks in search of equilibrium: You can't like parties if you don't like people. European Journal of Personality, 26(4), 414-431.](http://onlinelibrary.wiley.com/doi/10.1002/per.1866/full)

- [Costantini, G., Epskamp, S., Borsboom, D., Perugini, M., Mõttus, R., Waldorp, L. J., & Cramer, A. O. (2015). State of the aRt personality research: A tutorial on network analysis of personality data in R. Journal of Research in Personality, 54, 13-29.](www.sciencedirect.com/science/article/pii/S0092656614000701)

## License

This data and analysis are being prepared for publication and review. The repository is copyrighted and no permission is granted to re-analyse the data for publication or reuse any of the materials. After publication, code and data will ultimately be GPL-2; all measurement scales retain existing licence.
    
Any publication that re-uses the data should cite this publication.
