# Measuring Prejudice Through Word Embeddings
This tutorial provides an overview of the method used to analyze gender prejudice and stereotypes across 45 languages, as described in the paper "[How Language Shapes Prejudice Against Women: An Examination Across 45 World Languages](https://doi.org/10.1037/pspa0000188)" (DeFranza, Mishra, &amp; Mishra, 2020). 

The methodology described below will help researchers to build dictionaries, translate these dictionaries across languages, collect pre-trained word embeddings in each language, calculate the similarity between words of interest, and finally, perform statistical tests on the measured similarities. That said, this method can be further generalized to perform semantic analysis in other ways, using different constructs. The authors hope this resource will encourage other researchers to utilize the method in their work.

The tutorial proceeds as follows:

1. [Building a construct dictionary](#build-dict)
2. [Translating dictionaries](#translate-dict)
3. [Collecting pre-trained word embeddings](#collect-embeddings)
4. [Calculating cosine similarity](#cos-sim)
5. [Perfroming statistical (permutation) tests](#stats-tests)

If you find this tutorial helpful, [**please cite us**](#how-to-cite)!

## <a name="build-dict"></a>1. Building a construct dictionary
Perhaps the most critical step in this and similar methods is determining the words to include in a construct dictionary. For our research, these dictionaries were drawn from previous research. Specifically, we utilized a dictionary of _male_ and _female_ words from [Bolukbasi et al. (2016)](http://papers.nips.cc/paper/6228-man-is-to-computer-programmer-as-woman-is-to-homemaker-d), and a dictionary of _positive_ and _negative_ words from [Caliskan et al. (2017)](https://science.sciencemag.org/content/356/6334/183). The former was compiled by the researchers and refined using the responses from human raters. The latter was derived from the extant literature on implicit bias.

While these dictionaries are unique to our project, we feel they offer helpful illustrations of the two prominent strategies researchers could use to develop construct dictionaries for their own work.

## <a name="translate-dict"></a>2. Translating dictionaries
If the interest is in drawing comparisons across languages, the core dictionary will have to be translated into the languages of interest. It is worth noting that for some research questions, it might make sense to derive unique representations of each construct for each language. For example, certain types of plants (a common validation construct in IAT literature), may have different associations across cultures. In our case, the constructs of interest were relatively general and universal, so direct translation made the most sense.

The obvious means by which a dictionary could be translated is through trained human translators. This is certainly an option and should be considered. However, our project utilized automated machine translation, specifically through the [Google Translate API](https://cloud.google.com/translate/). This allowed us to quickly translate several hundred words across 45 languages.

To interface with the API, we utilized the [translateR package](https://cran.r-project.org/web/packages/translateR/translateR.pdf) in R. Given a list of words, for example:

```r
positive <- c("caress", "freedom", "health", "love", "peace", "cheer", "friend")
```
we can translate from English to German via:

```r
translate(content.vec = positive, google.api.key = googleKey, source.lang = "en", target.lang = "de")
```

Where ```positive``` is the name of the list containing the words to be translated, ```googleKey``` is a local variable holding your Google Translate API key, ```"en"``` is the [ISO-639-1 code](https://cloud.google.com/translate/docs/languages) for the source language, and ```"de"``` is the ISO-639-1 code for the target language.

## <a name="collect-embeddings"></a>3. Collecting pre-trained word embeddings
> _You shall know a word by the company it keeps_ ([Firth, 1957](https://www.worldcat.org/title/synopsis-of-linguistic-theory-1930-1955/oclc/177240275))

Pretrained word embeddings are distributed representations of a language ([Mikolov et al., 2013](https://papers.nips.cc/paper/5021-distributed-representations-of-words-and-phrases-and-their-compositionality.pdf)). A thorough description of word embeddings, the underlying theory, and the specific method used to train the embeddings used in this project is [provided in the paper](#how-to-cite) (DeFranza et al., 2020). Briefly, however, word embeddings are a numerical representation of a word in terms of its relative association with all other words observed in a language. 

Given a large corpus, researchers can train their own embeddings using one of the popular algorithms such as [GloVe](https://nlp.stanford.edu/projects/glove/) or [word2vec](http://text2vec.org/). However, developers maintaining implementations of the algorithms and researchers developing new methods have made numerous pretrained word embeddings available. These pretrained embeddings are conventient, validated, and interesting in that they are used in a wide range of production systems. Examples include word embeddings trained using [GloVe](https://nlp.stanford.edu/projects/glove/) and [fastText](https://fasttext.cc/). 

Our work utilized a set of word embeddings trained using fastText on the [Wikipedia](https://github.com/facebookresearch/fastText/blob/master/docs/pretrained-vectors.md) and [Common Crawl](https://github.com/facebookresearch/fastText/blob/master/docs/crawl-vectors.md) corpora. If only a small set of pretrained word embedding models are needed, they can simply be downloaded from the appropriate website.

If working with a large number of different pretrained embeddings, as we were, it is helpful to automate this process through a function. For example, in R, we can download each embedding from a list of URLs with the following:

```r
download.file(URL_for_embedding,
              "./path_to_working_directory/embedding_name.vec.gz",
              method = "auto", 
              mode = "wb", 
              cacheOK = FALSE)
```

Where `URL_for_embedding` is the URL, `"./path_to_working_directory/embedding_name.vec.gz"` is the path and file name where the file will be saved on the local machine, `method = "auto"` is the method R will use to perform the download (this usually defaults to `wget` or `curl`), `mode = "wb"` defines how the file will be encoded, in this case in binary, which is especially important if downloading compressed files (e.g., `.gz`) in a Windows environment, and `cacheOK = FALSE` refuses files from the server-side cache which increases stability.

Given a list of URLs,

```r
urls <- c("url_1", "url_2", "url_3")
```

a simple loop can collect all of the required pretrained embeddings:

```r
for (url in urls){
     download.file(url,
                   paste0("./path_to_working_directory/", basename(url)),
                   method = "auto", 
                   mode = "wb", 
                   cacheOK = FALSE)
}
```

Where `url` takes the `i`<sup>th</sup> item from the list `urls` and `paste0("./path_to_working_directory/", basename(url))` concatenates without intervening spaces (`paste0`) the working directory `"./path_to_working_directory/"` and the filename from the provided URL `basename(url)`. 

Once the files are downloaded, they can be read into R via any of the common functions (e.g., the `fread` function in the [`data.table` package](https://cran.r-project.org/web/packages/data.table/vignettes/datatable-intro.html)) and individual vectors can be extracted for each dictionary word. Note that this may cause memory issues on some machines due to the large size of pretrained embedding files. There are many possible solutions to this challenge. However, the one used by us utilizes a chunk-wise filter that reads only the immediately relevant rows from the word embedding file into memory.

For example, such a memory-conserving filter could be implemented using a function similar to:

```r
embedding_filter <- function(words, embeddingFile) {
  ## This function manages the input of source files larger than working
  ## memory. It does this by reading files a chunk at a time, then filtering
  ## based on a reference list.
  ##
  ## words is a reference list of words used for filtering
  ## embeddingFile is the path to or name of the input file
  ##
  ## Initialize the chunk counter.
  index <- 0
  ## Initialize the chunk size.
  chunkSize <- 100000
  ## Initialize a row skip value.
  skipValue <- 1
  ## Initialize the output data frame.
  working_embedding <- NULL
  
  ## Start the loop.
  ## Repeat until the stop condition.
  repeat {
      ## Read the file.
      chunk <- fread(file = embeddingFile,          # File to be read
                     skip = skipValue,              # Number of rows to skip
                     nrows = chunkSize,             # Number of rows to read
                     header = FALSE,                # Don't use a header
                     encoding = "UTF-8",            # Use UTF-8 encoding
                     sep = " ",                     # Define the file separator
                     quote = "")                    # Turn off quoting

      ## Count the rows in the unfiltered chunk.
      chunkLength <- nrow(chunk)
      
      ## Filter the chunk based on presence in the reference list.
      chunk <- chunk %>%
            mutate(V1 = stri_trans_tolower(V1)) %>%
            filter(V1 %in% words)
          
      ## Add the filtered results to the output data frame.
      working_embedding <- rbind(working_embedding, chunk)
      ## Increment the chunk counter.
      index <- index + 1
      ## Increment the row skip value.
      skipValue <- index * chunkSize
      
      ## Check the stop condition for the loop.
      if (chunkLength != chunkSize) {
          break
      }
}
```

Once the pretrained embeddings are obtained and filtered, calculating the cosine similarity is relatively straight-forward.

## 4. <a name="cos-sim"></a>Calculating cosine similarity

A word embedding is the mapping of a string, or word, into a high-dimensional vector or geometric space. Formally, ![equation](https://latex.codecogs.com/gif.latex?w_i%20%5Cmapsto%20%5Cmathbb%7BR%7D%5Ed), where ![equation](https://latex.codecogs.com/gif.latex?d) in our case was 300. This allows us to utilize simple metrics of geometric association to calculate similarity between the vector for each word. Specifically, we use the commonly applied metric of orientation known as _cosine similarity_ to measure association between vectors. Formally, cosine similarity is calculated as:

![equation](https://latex.codecogs.com/gif.latex?%5Ccos%28%5Ctheta%29%20%3D%20%5Cfrac%7B%5Csum_%7Bi%3D1%7D%5En%20w1_iw2_1%7D%7B%5Csqrt%7B%5Csum_%7Bn%3D1%7D%5En%20w1%5E2_i%7D%5Csqrt%7B%5Csum_%7Bn%3D1%7D%5En%20w2%5E2_i%7D%7D)

Where the numerator is the dot product of the vectors for word ![equation](https://latex.codecogs.com/gif.latex?w1) and word ![equation](https://latex.codecogs.com/gif.latex?w2), repectively, and the denominator is the product of the norms of the vectors for the same words.

The cosine similarity calculation can be implemented directly through a function in R (the approach the authors took) or through a number of popular packages for R, for example [`text2vec`](http://text2vec.org/). Either way, the analysis procedes by calculating each similarity for each of the required word combinations (e.g., cos(_man_, _freedom_), cos(_man_, _health_), cos(_woman_, _freedom_), cos(_woman_, _health_)). By taking the mean of each pair within each construct, relationships can be measured using basic arithmatic. Details of this calculation are provided in the associated paper (DeFranza et al., 2020).

Note that the pairwise calculation of cosine similarities can be simplified (see [Garten et al., 2018](https://link.springer.com/article/10.3758/s13428-017-0875-9) for a discussion).

## 5. <a name="stats-tests"></a>Perfroming statistical (permutation) tests

The cosine similarity is representative of the relative orientation of two words in vector space. However, this calculation could be the result of random chance. As such, we test the validity of our observations through the use of the permutation test (adapted from [Caliskan et al. (2017)](https://science.sciencemag.org/content/356/6334/183)). Simply, the permutation test compares the likelihood of an observation with that of all possible combinations of the data. An observation that is more extreme than the majority of the permutations is considered unlikely to be the result of chance alone.

Practically speaking, we implemented the permutation test with our data by first taking the observation of interest (e.g., _mean_(cos(_male words_, _positive words_))) and comparing it to thousands of similar calculations (10,000 - 50,000 are commonly used), shuffling one of the labels (e.g., male-female labels) each iteration, to approximate all possible permutations of male-female and positive-negative word pairs.

This simulation method allows us to calculate a _p_-value, effect size, and other common statistics. One note is that often bootstrapping and other simulation methods produce a _p_-value of 0, which is mathematically impossible. A correction for this has been implemented in the [`permp` package](https://cran.r-project.org/web/packages/perm/perm.pdf) ([Phipson & Smyth, 2016](https://arxiv.org/abs/1603.05766)).

Once these calculations have been completed, constructs can be compared using traditional methods (e.g., linear models including ANOVA, OLS regression, etc.). A full discussion of our statistical analyses is included in the paper (DeFranza et al., 2020).

## <a name="bhow-to-cite"></a>How to cite
If you have made use of this tutorial or the associated paper, please include the following citation in your work:

DeFranza, D., Mishra, H., & Mishra, A. (2020). How language shapes prejudice against women: An examination across 45 world languages. *Journal of Personality and Social Psychology*. [https://doi.org/10.1037/pspa0000188](https://doi.org/10.1037/pspa0000188)

```
@article{DeFranza2020,
  doi = {10.1037/pspa0000188},
  url = {https://doi.org/10.1037/pspa0000188},
  year = {2020},
  month = feb,
  publisher = {American Psychological Association ({APA})},
  author = {David DeFranza and Himanshu Mishra and Arul Mishra},
  title = {How language shapes prejudice against women: An examination across 45 world languages.},
  journal = {Journal of Personality and Social Psychology}
}
```
