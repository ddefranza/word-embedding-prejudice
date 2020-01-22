# Measuring Prejudice Through Word Embeddings
This tutorial provides an overview of the method used to analyze gender prejudice and stereotypes across 45 languages, as described in the paper "How Language Shapes Prejudice Against Women: An Examination Across 45 World Languages" (DeFranza, Mishra, &amp; Mishra, 2020). 

The methodology described below can be used to reproduce the analysis in the associated paper. Specifically, it will allow researchers to build construct dictionaries, translate these dictionaries across languages, collect pre-trained word embeddings in each language, calculate the similarity between words of interest, and finally, perform statistical tests on the measured similarities. That said, this method can be further generalized to perform semantic analysis in other ways, using different constructs. The authors hope this resource will help encourage other researchers to utilize the method in their work.

The tutorial proceeds as follows:

1. [Building a construct dictionary](#build-dict)
2. Translating dictionaries
3. Collecting pre-trained word embeddings
4. Calculating cosine similarity
5. Perfroming statistical (permutation) tests

## <a name="build-dict"></a>1. Building a construct dictionary
Perhaps the most critical step in this and similar methods is determining the words to include in a construct dictionary. For our research, these dictionaries were drawn from previous research. Specifically, we utilized a dictionary of _male_ and _female_ words from [Bolukbasi et al. (2016)](http://papers.nips.cc/paper/6228-man-is-to-computer-programmer-as-woman-is-to-homemaker-d), and a dictionary of _positive_ and _negative_ words from [Caliskan et al. (2017)](https://science.sciencemag.org/content/356/6334/183). The former was compiled by the researchers and refined using the responses from human raters. The latter was derived from the extant literature on implicit bias.

While these dictionaries are unique to our project, we feel they offer helpful illustrations of the two prominent strategies researchers could use to develop construct dictionaries for their own work.

## How to cite
If you have made use of this tutorial or the associated paper, please including the following citation in your work:

DeFranza, D., Mishra, H., & Mishra, A. (2020). How language shapes prejudice against women: An examination across 45 world languages. *Journal of Personality and Social Psychology*.
