
### Contribution
A layer called Vi-SATNets that facilitates deeper pattern recognition from pictures.

'Recall that the objective of a SATNet layer is to learn a set of rules such that when applied, unknown variables in a given incomplete puzzle can be solved。 Similarly, the objective of a Vi-SATNet model is to learn a set of rules such that when applied, missing feature values (extracted from an image) in a given incomplete feature map can be re-generated'


### Approach
At each index position, a vector is used to represent the value corresponding to each feature channel. The number of vectors equals to the number of unites in a feature map. The number of entries in each vector equals to the number of channels. 

Forward pass: Feature vec set -> mask -> weight matrix S -> cosine loss. 
Backward pass: loss -> update weight in S.

>***NOTE***
>normalization does not cause info loss

Each Vi-SATNets only needs to learn one label class.

Quality of regeneration is determined by classification accuracy with a classifier.

Overall flow:
Image -> feature extractor -> More channels, less size -> S matrix -> Rule contribution for each channel -> vary missing feature vector so it product to the S matrix column is minimal comparing to the original value 




### Obtained on:
Learning Reliable Rules by Re-generating Deep Features. _Submitted to ICLR 2025_. OpenReview submission ID: ELQ8X02IEp. Available at: [https://openreview.net/forum?id=ELQ8X02IEp](https://openreview.net/forum?id=ELQ8X02IEp)