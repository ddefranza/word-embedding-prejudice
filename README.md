# Measuring Prejudice Through Word Embeddings
This tutorial provides an overview of the method used to analyze gender prejudice and stereotypes across 45 languages, as described in the paper "How Language Shapes Prejudice Against Women: An Examination Across 45 World Languages" (DeFranza, Mishra, &amp; Mishra, 2020). 

The methodology described below can be used to reproduce the analysis in the associated paper. Specifically, it will allow researchers to build construct dictionaries, translate these dictionaries across languages, collect pre-trained word embeddings in each language, calculate the similarity between words of interest, and finally, perform statistical tests on the measured similarities. That said, this method can be further generalized to perform semantic analysis in other ways, using different constructs. The authors hope this resource will help encourage other researchers to utilize the method in their work.

The tutorial proceeds as follows:

1. [Building a construct dictionary](#build-dict)
2. [Translating dictionaries](#translate-dict)
3. [Collecting pre-trained word embeddings](#collect-embeddings)
4. Calculating cosine similarity
5. Perfroming statistical (permutation) tests

If you find this tutorial helpful, [**Please cite us!**](#how-to-cite)

## <a name="build-dict"></a>1. Building a construct dictionary
Perhaps the most critical step in this and similar methods is determining the words to include in a construct dictionary. For our research, these dictionaries were drawn from previous research. Specifically, we utilized a dictionary of _male_ and _female_ words from [Bolukbasi et al. (2016)](http://papers.nips.cc/paper/6228-man-is-to-computer-programmer-as-woman-is-to-homemaker-d), and a dictionary of _positive_ and _negative_ words from [Caliskan et al. (2017)](https://science.sciencemag.org/content/356/6334/183). The former was compiled by the researchers and refined using the responses from human raters. The latter was derived from the extant literature on implicit bias.

While these dictionaries are unique to our project, we feel they offer helpful illustrations of the two prominent strategies researchers could use to develop construct dictionaries for their own work.

## <a name="translate-dict"></a>2. Translating dictionaries
If the interest is in drawing comparisons across languages, the core dictionary will have to be translated into the languages of interest. It is worth noting that for some research questions, it might make sense to derive unique representations of each construct for each language. For example, certain types of plants (a common validation construct in IAT literature, for example), may have different associations across cultures. In our case, the constructs of interest were relatively general and universal, so direct translation made the most sense.

The obvious means by which a dictionary could be translated is through trained human translators. This is certainly an option and should be considered. However, our project utilized automated machine translation, specifically through the [Google Translate API](https://cloud.google.com/translate/). This allowed us to quickly translate several hundred words across 45 languages.

To interface with the API, we utilized the [translateR package](https://cran.r-project.org/web/packages/translateR/translateR.pdf) in R. Given a vector of words, for example:

```r
positive <- c("caress", "freedom", "health", "love", "peace", "cheer", "friend")
```
we can translate from English to German via:

```r
translate(content.vec = positive, google.api.key = googleKey, source.lang = "en", target.lang = "de")
```

Where ```positive``` is the name of the vector or list containing the words to be translated, ```googleKey``` is a local variable holding your Google Translate API key, ```"en"``` is the [ISO-639-1 code](https://cloud.google.com/translate/docs/languages) for the source language, and ```"de"``` is the ISO-639-1 code for the target language.

## <a name="collect-embeddings"></a>3. Collecting pre-trained word embeddings
> _You shall know a word by the company it keeps_ ([Firth, 1957](https://www.worldcat.org/title/synopsis-of-linguistic-theory-1930-1955/oclc/177240275))

Pretrained word embeddings are distributed and distributial representations of a language. A thorough description of word embeddings, the underlying theory, and the specific method used to train the embeddings used in this project is [provided in the paper](#how-to-cite). Briefly, however, word embeddings are a numerical representation of a word in terms of its relative association with all other words observed in a language. 

## <a name="bhow-to-cite"></a>How to cite
If you have made use of this tutorial or the associated paper, please including the following citation in your work:

DeFranza, D., Mishra, H., & Mishra, A. (2020). How language shapes prejudice against women: An examination across 45 world languages. *Journal of Personality and Social Psychology*.
