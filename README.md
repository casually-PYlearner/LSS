
Latent Semantic Scaling
=======================

LSS is a highly efficient vector-space model for subject-specific sentiment analysis used by Kohei Watanabe in several published studies:

-   Kohei Watanabe, 2017. "[Measuring News Bias: Russia’s Official News Agency ITAR-TASS’s Coverage of the Ukraine Crisis](http://journals.sagepub.com/eprint/TBc9miIc89njZvY3gyAt/full)", *European Journal Communication*
-   Kohei Watanabe, 2017. "[The spread of the Kremlin’s narratives by a western news agency during the Ukraine crisis](http://www.tandfonline.com/eprint/h2IHsz2YKce6uJeeCmcd/full)", *Journal of International Communication*
-   Tomila Lankina and Kohei Watanabe. Forthcoming. "Russian Spring’ or ‘Spring Betrayal’? The Media as a Mirror of Putin’s Evolving Strategy in Ukraine"", *Europe-Asia Studies*

How to install
--------------

``` r
devtools::install_github("koheiw/LSS")
```

How to use
----------

LSS is created to perform sentiment analysis of long texts, but training of its models should be done on smaller units, typically sentences. The [sample dataset](https://www.dropbox.com/s/555sr2ml6wc701p/guardian-sample.RData?dl=0) contains 6,000 Guardian news articles, but larger corpus should be used to [estimate parameters accurately](https://koheiw.net/?p=629).

### Fit a LSS model

``` r
require(quanteda)
require(LSS)
```

``` r
load('/home/kohei/Dropbox/Public/guardian-sample.RData')

corp_train <- corpus_reshape(data_corpus_guardian, 'sentences')
toks_train <- tokens(corp_train, remove_punct = TRUE)
mt_train <- dfm(toks_train, remove = stopwords())
mt_train <- dfm_remove(mt_train, c('*.uk', '*.com', '*.net', '*.it', '*@*'))
mt_train <- dfm_trim(mt_train, min_count = 10)

#' sentiment model on economy
eco <- char_keyness(toks_train, 'econom*')
lss_eco <- textmodel_lss(mt_train, seedwords('pos-neg'), pattern = eco)

head(lss_eco$beta) # most positive words
```

    ##        positive         reasons     opportunity pic.twitter.com 
    ##      0.06436414      0.04925601      0.04260535      0.04198808 
    ##        november          strong 
    ##      0.04095714      0.03822935

``` r
tail(lss_eco$beta) # most negative words
```

    ##      blamed       rates      warned        poor         bad    negative 
    ## -0.05706067 -0.06038286 -0.06087443 -0.06842289 -0.07496055 -0.08114593

``` r
# sentiment model on politics
pol <- char_keyness(toks_train, 'politi*')
lss_pol <- textmodel_lss(mt_train, seedwords('pos-neg'), pattern = pol)

head(lss_pol$beta) # most positive words
```

    ##       team    reasons   november      faith      force   research 
    ## 0.05061622 0.04925601 0.04095714 0.03940039 0.03662932 0.03587227

``` r
tail(lss_pol$beta) # most negative words
```

    ##   criticised   chancellor increasingly        power      happens 
    ##  -0.03815973  -0.03833504  -0.04339442  -0.04397653  -0.04523024 
    ##        rates 
    ##  -0.06038286

Predict sentiment of news
-------------------------

``` r
mt <- dfm(data_corpus_guardian)
```

### Economic sentiment

``` r
sent_eco <- scale(predict(lss_eco, newdata = mt))
plot(docvars(data_corpus_guardian, 'date'), sent_eco, pch = 16, col = rgb(0, 0, 0, 0.1),
     ylim = c(-1, 1), ylab = 'economic sentiment')
lines(lowess(docvars(data_corpus_guardian, 'date'), sent_eco, f = 0.05), col = 1)
abline(h = 0)
```

![](man/images/unnamed-chunk-6-1.png)

### Political sentiment

``` r
sent_pol <- scale(predict(lss_pol, newdata = mt))
plot(docvars(data_corpus_guardian, 'date'), sent_pol, pch = 16, col = rgb(0, 0, 0, 0.1),
      ylim = c(-1, 1), ylab = 'political sentiment')
lines(lowess(docvars(data_corpus_guardian, 'date'), sent_pol, f = 0.05), col = 1)
abline(h = 0)
```

![](man/images/unnamed-chunk-7-1.png)

### Comparison

The sentiment analysis models were trained on the same corpus with the same seed words, but they are sensitive to different subjects. We can see in the chart below that Guardian's framing of economy became positive in early 2015, while political sentiment were falling down gradually.

``` r
plot(docvars(data_corpus_guardian, 'date'), rep(0, ndoc(data_corpus_guardian)),  
     type = 'n', ylim = c(-0.5, 0.5), ylab = 'economic/political sentiment')
grid()
lines(lowess(docvars(data_corpus_guardian, 'date'), sent_pol, f = 0.1), col = 1)
lines(lowess(docvars(data_corpus_guardian, 'date'), sent_eco, f = 0.1), col = 2)
abline(h = 0)
legend('topright', lty = 1, col = 1:2, legend = c('political', 'economic'))
```

![](man/images/unnamed-chunk-8-1.png)
