
# Topic modeling    

A topic model is a type of statistical model for discovering the abstract "topics" that occur in a collection of documents. Topic modeling is a frequently used text-mining tool for discovery of hidden semantic structures in a text body.  A document typically concerns multiple topics in different proportions; thus, in a document that is 10% about cats and 90% about dogs, there would probably be about 9 times more dog words than cat words. The "topics" produced by topic modeling techniques are clusters of similar words. 




## Latent Dirichlet Allocation 

> Latent Dirichlet allocation (LDA) is a particularly popular method for fitting a topic model. It treats each document as a mixture of topics, and each topic as a mixture of words. This allows documents to “overlap” each other in terms of content, rather than being separated into discrete groups, in a way that mirrors typical use of natural language.  

The LDA model is guided by two principles: 

- **Each document is a mixture of topics**. In a 3 topic model we could assert that a document is 70% about topic A, 30 about topic B, and 0% about topic C.    

- **Every topic is a mixture of words**. A topic is considered a probabilistic distribution over multiple words.

<div class="figure" style="text-align: center">
<img src="images/LDA.jpg" alt="Source: http://nlpx.net/wp/wp-content/uploads/2016/01/LDA_image2.jpg" width="120%" />
<p class="caption">(\#fig:unnamed-chunk-2)Source: http://nlpx.net/wp/wp-content/uploads/2016/01/LDA_image2.jpg</p>
</div>

In particular, LDA is a imagined generative process, illustrated in the plate notation below:  


<div class="figure" style="text-align: center">
<img src="images/LDA_GPM.png" alt="(ref:LDA)" width="120%" />
<p class="caption">(\#fig:unnamed-chunk-3)(ref:LDA)</p>
</div>


(ref:LDA) Source: @lee  


- $M$ denotes the number of documents   
- $N$ is the number of words in a given document (document $i$ has $N_i$ words)   
- $\vec{\theta_m}$ is the expected topic proportion of document $m$, which is generated by a Dirichlet distribution parameterized by $\vec{\alpha}$ (e.g., in a two topic model $\theta_m = [0.3, 0.7]$ means document $m$ is expected to have 30% topic 1 and 70% topic 2)    
- $\vec{\phi_k}$ is the word distribution of topic $k$, which is generated by a Dirichlet distribution parameterized by $\vec{\beta}$  
- $z_{m, n}$ is the topic for the $n$th word in document $m$, one word are assigned to one topic.  
- $w_{m, n}$ is the word in the $n$th position word of document $m$   

The only observed variable in this graphical probabilistic model is $w_{m, n}$, so it is "latent".

To actually infer the topics in a corpus, we imagine the generative process as follows. LDA assumes the following generative process for a corpus $D$ consisting of $M$ M documents each of length $N_i$:  

1. Generate $\vec{\theta_i} \sim \text{Dir}(\vec{\alpha})$, where $i \in \{1, 2, ..., M\}$. $\text{Dir}(\vec{\alpha})$ is a Dirichlet distribution with symmetric parameter $\vec{\alpha}$ where $\vec{\alpha}$ is often sparse.

2. Generate $\vec{\phi_k} \sim \text{Dir}(\vec{\beta})$, where $k \in \{1, 2, ..., K\}$ and $\vec{\beta}$ is typically sparse  

3. For the $n$th position in document $m$, where $n \in \{1, 2, ..., N_m\}$ and $m \in \{1, 2, ..., M\}$   
    a. Choose a topic $z_{m, n}$ for that position which is generated from $z_{m, n} \sim \text{Multinomial}(\vec{\theta_i})$   
    b. Fill in that position with word $w_{m, n}$ which is generated from the word distribution of the topic picked in the previous step $w_{i,j} \sim \text{Multinomial}(\phi_{z_{m, n}})$ 
  
### Example: Associated Press  

We come to the `AssociatedPress` document term matrix (the required data strcture for the modeling function) and fit a two topic LDA model with `stm::stm` (stm stands for structural equation modeling). The `stm` takes as its input a document-term matrix, either as a sparse matrix (using `cast_sparse`) or a `dfm` from `quanteda` (using `cast_dfm`). Here we specify a two topic model by setting $K = 2$ for demonstration purposes, in \@ref(tuning-number-of-topics) we will see how to choose $K$ with metrics such as semantic coherence. 


```r
library(stm)
data("AssociatedPress", package = "topicmodels")

ap_dfm <- tidy(AssociatedPress) %>% 
  cast_dfm(document = document, term = term, value = count)

ap_lda <- stm(ap_dfm, 
              K = 2, 
              verbose = FALSE, 
              init.type = "LDA")
summary(ap_lda)
#> A topic model with 2 topics, 2246 documents and a 10473 word dictionary.
#> Topic 1 Top Words:
#>  	 Highest Prob: i, people, two, police, years, officials, government 
#>  	 FREX: police, killed, miles, attorney, death, army, died 
#>  	 Lift: accident, accusations, ackerman, adams, affidavit, algotson, alice 
#>  	 Score: police, killed, trial, prison, died, army, hospital 
#> Topic 2 Top Words:
#>  	 Highest Prob: percent, new, million, year, president, bush, billion 
#>  	 FREX: percent, bush, billion, market, trade, prices, economic 
#>  	 Lift: corporate, growth, lawmakers, percentage, republicans, abboud, abm 
#>  	 Score: percent, billion, dollar, prices, yen, stock, market
```

`stm` objects have a `summary()` method for displaying words with highest probability in each topic. But we want to go back to data frames to take advantage of `dplyr` and `ggplot2`. For tidying model objects, `tidy(model_object, matrix = "beta")` (the default) access the topic-word probability vector (we denotes with $\vec{\phi_k}$)  


```r
tidy(ap_lda)
#> # A tibble: 20,946 x 3
#>   topic term            beta
#>   <int> <chr>          <dbl>
#> 1     1 aaron     0.0000324 
#> 2     2 aaron     0.0000117 
#> 3     1 abandon   0.00000654
#> 4     2 abandon   0.0000676 
#> 5     1 abandoned 0.000140  
#> 6     2 abandoned 0.0000341 
#> # ... with 20,940 more rows
```

Which words have a relateve higher probabiltity to appear in each topic?    


```r
tidy(ap_lda) %>% 
  group_by(topic) %>%
  top_n(10) %>% 
  ungroup() %>% 
  mutate(topic = str_c("topic", topic)) %>%
  facet_bar(y = term, x = beta, by = topic) +
  labs(title = "Words with highest probability in each topic")
```

<img src="topic-modeling_files/figure-html/unnamed-chunk-6-1.png" width="672" style="display: block; margin: auto;" />

As an alternative, we could consider the terms that had the **greatest difference** in $\vec{\phi_k}$ between topic 1 and topic 2. This can be estimated based on the log ratio of the two: $\log_2(\frac{\phi_{1n}}{\phi_{2n}})$, $\phi_{1n} / \phi_{2n}$ being the probability ratio of the sam e word $n$ in two topics (a log ratio is useful because it makes the difference symmetrical)  


```r
phi_ratio <- tidy(ap_lda) %>%
  mutate(topic = str_c("topic", topic)) %>% 
  pivot_wider(names_from = topic, values_from = beta) %>% 
  filter(topic1 > .001 | topic2 > .001) %>%
  mutate(log_ratio = log2(topic2 / topic1))
```



This can answer a question like: which word is most representative of a topic?  


```r
phi_ratio %>% 
  top_n(20, abs(log_ratio)) %>% 
  ggplot(aes(y = fct_reorder(term, log_ratio),
             x = log_ratio)) + 
  geom_col() + 
  labs(y = "",
       x = "log ratio of phi between topic 2 and topic 1 (base 2)")
```

<img src="topic-modeling_files/figure-html/unnamed-chunk-8-1.png" width="672" style="display: block; margin: auto;" />


To extrac the word proportion vector $\vec{\theta_m}$ for document $m$, use `matrix = "gamma"` in `tidy()`  


```r
tidy(ap_lda, matrix = "gamma")
#> # A tibble: 4,492 x 3
#>   document topic gamma
#>      <int> <int> <dbl>
#> 1        1     1 0.990
#> 2        2     1 0.610
#> 3        3     1 0.983
#> 4        4     1 0.698
#> 5        5     1 0.891
#> 6        6     1 0.519
#> # ... with 4,486 more rows
```


With this data frame, we want to knwo which document is most charateristic of each topic?  


```r
library(reshape2)
library(wordcloud)

tidy(ap_lda, matrix = "gamma") %>% 
  group_by(topic) %>%
  top_n(15) %>% 
  mutate(document = as.character(document)) %>% 
  acast(document ~ topic, value.var = "gamma", fill = 0) %>% 
  comparison.cloud(colors = c("gray20", "gray80"), scale = c(2, 8))
```

<img src="topic-modeling_files/figure-html/unnamed-chunk-10-1.png" width="672" style="display: block; margin: auto;" />

This plot would definitely be more insightful if we have document titles rather than an ID.  

\BeginKnitrBlock{rmdnote}<div class="rmdnote">To sum up, the topic modeling workflow involves:  
- use tidy tools like dplyr, tidyr, and ggplot2 for initial data exploration and preparation.  
- **cast** to a non-tidy structure to perform some machine learning algorithm.
- **tidy** the modeling results to to use tidy tools again (exploring, visualization, etc.)</div>\EndKnitrBlock{rmdnote}


## Example: the great library heist  

To evaluate our topic model, we first divided 4 books into chapters. If a topic model with $K = 4$ performs well, then there should be a corresponding segmentation among those chpaters coming from those 4 different books.  


```r
titles <- c("Twenty Thousand Leagues under the Sea", 
            "The War of the Worlds",
            "Pride and Prejudice", 
            "Great Expectations")  

library(gutenbergr)

books <- gutenberg_works(title %in% titles) %>%
  gutenberg_download(meta_fields = "title")
```


```r
# add a chapter column
by_chapter <- books %>%
  group_by(title) %>%
  mutate(chapter = cumsum(str_detect(text, regex("^chapter ", 
                                                 ignore_case = TRUE)))) %>%
  ungroup() %>%
  filter(chapter > 0) %>%
  unite(col = document, title, chapter)


# find document-word counts
word_counts <- by_chapter %>%
  unnest_tokens(word, text) %>% 
  anti_join(stop_words) %>%
  rename(chapter = document) %>% 
  count(chapter, word, sort = TRUE) %>%
  ungroup()

word_counts
#> # A tibble: 104,722 x 3
#>   chapter               word        n
#>   <chr>                 <chr>   <int>
#> 1 Great Expectations_57 joe        88
#> 2 Great Expectations_7  joe        70
#> 3 Great Expectations_17 biddy      63
#> 4 Great Expectations_27 joe        58
#> 5 Great Expectations_38 estella    58
#> 6 Great Expectations_2  joe        56
#> # ... with 104,716 more rows
```

### LDA on chapters  



```r
chapters_dfm <- word_counts %>% 
  cast_dfm(document = chapter, term = word, value = n)

chapters_lda <- stm(chapters_dfm, K = 4, init.type = "LDA", verbose = FALSE)
```


Much as we did on the Associated Press data, we can examine per-topic-per-word probabilities.  


```r
chapter_topics <- tidy(chapters_lda, matrix = "beta")
chapter_topics
#> # A tibble: 72,860 x 3
#>   topic term              beta
#>   <int> <chr>            <dbl>
#> 1     1 _accident_   0.0000263
#> 2     2 _accident_   0        
#> 3     3 _accident_   0        
#> 4     4 _accident_   0        
#> 5     1 _advantages_ 0.0000263
#> 6     2 _advantages_ 0        
#> # ... with 72,854 more rows
```

We can find top 5 terms within each topic.  


```r
chapter_topics %>% 
  group_by(topic) %>% 
  top_n(5) %>% 
  ungroup() %>% 
  mutate(topic = str_c("topic", topic)) %>% 
  facet_bar(y = term, x = beta, by = topic) + 
  labs(x = expression(beta), 
       title = "topic-term probabilities")
```

<img src="topic-modeling_files/figure-html/unnamed-chunk-16-1.png" width="672" style="display: block; margin: auto;" />

I am not an expert on the other 3 books aside from Pride & Prejudice, but according to [Julia](https://www.tidytextmining.com/topicmodeling#lda-on-chapters), each topic did correspond to one book by and large!  

### Per-document classification

We may want to how and which topics are associated with each document, in particular, the majority of chapters in the same book should belong to the same topic (if we assign a chapter$_m$ to a topic$_k$ when the $k$th postion in $\hat{\theta}_m$ is significantly higher).  


```r
chapters_gamma <- tidy(chapters_lda,
                       matrix = "gamma",
                       document_names = rownames(chapters_dfm)) %>% 
  separate(document, c("title", "chapter"), sep = "_", convert = TRUE) %>% 
  mutate(topic = factor(topic) %>% fct_inseq())

chapters_gamma
#> # A tibble: 772 x 4
#>   title              chapter topic    gamma
#>   <chr>                <int> <fct>    <dbl>
#> 1 Great Expectations      57 1     0.00602 
#> 2 Great Expectations       7 1     0.0149  
#> 3 Great Expectations      17 1     0.0403  
#> 4 Great Expectations      27 1     0.000570
#> 5 Great Expectations      38 1     0.0281  
#> 6 Great Expectations       2 1     0.000461
#> # ... with 766 more rows
```




```r
ggplot(chapters_gamma) + 
  geom_boxplot(aes(topic, gamma)) + 
  facet_wrap(~ title) + 
  labs(title = "chapter-topic proportion", y = expression(gamma))
```

<img src="topic-modeling_files/figure-html/unnamed-chunk-18-1.png" width="672" style="display: block; margin: auto;" />

Ideally we would expect that in every book panel, there is one boxplot highly centered at 1 with the other 3 boxes at 0, since chapters in the same book are categorized in the same topic. 


Another way of visualizaing this is to plot the histogaram of chapter-topic proportions of each topic. We would expect to see two extremes  


```r
ggplot(chapters_gamma) + 
  geom_histogram(aes(gamma, fill = topic), show.legend = FALSE) + 
  facet_wrap(~ topic) + 
  labs(y = "Number of chapters",
       x = expression(gamma),
       title = "Distribution of document probabilities for each topic")
```

<img src="topic-modeling_files/figure-html/unnamed-chunk-19-1.png" width="672" style="display: block; margin: auto;" />


It does look like some chapters from *Twenty Thousand Leagues under the Sea* were somewhat associated with other topic 3 (whereas most chapters are assigned to topic 2). Let's put in some investigation


```r
chapters_gamma %>% 
  filter(title == "Twenty Thousand Leagues under the Sea") %>%
  ggplot() + 
  geom_histogram(aes(gamma, fill = topic),
                 position = "identity",
                 alpha = 0.6) + 
  guides(fill = guide_legend(override.aes = list(alpha = 1))) + 
  labs(title = "Twenty Thousand Leagues under the Sea",
       subtitle = "chapter-topic proportions")
```

<img src="topic-modeling_files/figure-html/unnamed-chunk-20-1.png" width="672" style="display: block; margin: auto;" />

Which chapters have a relatively high proportion of topic 3?


```r
chapters_gamma %>% 
  filter(title == "Twenty Thousand Leagues under the Sea") %>% 
  filter(topic == "3", gamma > 0.1)
#> # A tibble: 0 x 4
#> # ... with 4 variables: title <chr>, chapter <int>, topic <fct>, gamma <dbl>
```



As we see here, topci modeling can be viewed as text classification to some degree. We can find the topic that was most associated with each chapter using `top_n()`, which is essentially the "classification" of that chapter. For example, the 57th chapter of *Great Expectations* are assigned to topic 1.


```r
chapter_classifications <- chapters_gamma %>%
  group_by(title, chapter) %>%
  top_n(1, gamma) %>%
  ungroup()

chapter_classifications
#> # A tibble: 193 x 4
#>   title               chapter topic gamma
#>   <chr>                 <int> <fct> <dbl>
#> 1 Pride and Prejudice      43 1     0.994
#> 2 Pride and Prejudice      18 1     1.00 
#> 3 Pride and Prejudice      45 1     0.999
#> 4 Pride and Prejudice      16 1     0.999
#> 5 Pride and Prejudice      29 1     0.999
#> 6 Pride and Prejudice      10 1     0.999
#> # ... with 187 more rows
```

We can then compare each to the "consensus" topic for each book (the most common topic among its chapters), and see if there is misidentification


```r
chapter_classifications %>% 
  count(title, topic)
#> # A tibble: 5 x 3
#>   title                                 topic     n
#>   <chr>                                 <fct> <int>
#> 1 Great Expectations                    3         5
#> 2 Great Expectations                    4        54
#> 3 Pride and Prejudice                   1        61
#> 4 The War of the Worlds                 3        27
#> 5 Twenty Thousand Leagues under the Sea 2        46
```

In all of the 4 books, no single chapter is misidentified to another topic! 

For future need, classification results are stored in `classification` 


```r
classification <- chapter_classifications %>% 
    count(title, topic) %>%
    group_by(title) %>% 
    top_n(1, n) %>% 
    ungroup() %>%
    transmute(assigned_book = title, topic)
```



### By word assignments: `augment()`   

One step of the LDA algorithm is assigning each word in each document to a topic $z_{m, n}$. The more words in a document are assigned to that topic, generally, the more weight $\theta_m$ will go on that document-topic classification.

We may want to take the original document-word pairs and find which words in each document were assigned to which topic. This is the job of the `augment()` function, which is to add information to each observation in the original data.  


```r
assignments <- augment(chapters_lda, data = chapters_dfm) %>% 
  transmute(chapter = document, 
            term,
            count,
            topic = factor(.topic))

assignments
#> # A tibble: 104,722 x 4
#>   chapter               term  count topic
#>   <chr>                 <chr> <dbl> <fct>
#> 1 Great Expectations_57 joe      88 4    
#> 2 Great Expectations_7  joe      70 4    
#> 3 Great Expectations_17 joe       5 4    
#> 4 Great Expectations_27 joe      58 4    
#> 5 Great Expectations_2  joe      56 4    
#> 6 Great Expectations_23 joe       1 4    
#> # ... with 104,716 more rows
```

To get a sense of how our model works, we can draw a bar plot of assigned topics in each book  


```r
assignments %>% 
  separate(chapter, into = c("title", "chapter"), sep = "_") %>%
  count(title, topic, wt = count) %>%
  ggplot(aes(topic, n)) + 
  geom_col(width = 0.5) + 
  facet_wrap(~ title, scales = "free") + 
  labs(y = "Number of words",
       x = "topic",
       title = "By word assignments")
```

<img src="topic-modeling_files/figure-html/unnamed-chunk-26-1.png" width="672" style="display: block; margin: auto;" />


We can combine this assignments table with the `classification` to find which words were incorrectly classified by a coofusion matrix.  


```r
assignments <- assignments %>%
  separate(chapter, c("title", "chapter"), sep = "_", convert = TRUE) %>%
  left_join(classification, by = c("topic" = "topic"))

# misidentified words
assignments %>% 
  filter(title != assigned_book)
#> # A tibble: 7,173 x 6
#>   title                   chapter term    count topic assigned_book             
#>   <chr>                     <int> <chr>   <dbl> <fct> <chr>                     
#> 1 Twenty Thousand League~       8 miss        1 4     Great Expectations        
#> 2 Great Expectations            5 sergea~    37 3     The War of the Worlds     
#> 3 Great Expectations           46 captain     1 2     Twenty Thousand Leagues u~
#> 4 Great Expectations           32 captain     1 2     Twenty Thousand Leagues u~
#> 5 The War of the Worlds        17 captain     5 2     Twenty Thousand Leagues u~
#> 6 Pride and Prejudice           7 captain     3 2     Twenty Thousand Leagues u~
#> # ... with 7,167 more rows
```


```r
# confusion matrix  
confusion_df <- assignments %>% 
  count(title, topic, wt = count) %>% 
  group_by(title) %>% 
  mutate(percent = n / sum(n)) %>% 
  ungroup()

 
confusion_df %>% 
  ggplot(aes(topic, title, fill = percent)) + 
  geom_tile() + 
  scale_fill_distiller(type = "div",
                       palette = "RdBu",
                       label = scales::percent_format()) +
  scale_x_discrete(guide = guide_axis(n.dodge = 2)) + 
  theme_minimal() +
  theme(legend.position = "top") + 
  labs(x = "Book words were assigned to",
       y = "Book words came from",
       fill = "% of assignments")
```

<img src="topic-modeling_files/figure-html/unnamed-chunk-28-1.png" width="960" style="display: block; margin: auto;" />

What were the most commonly mistaken words?  


```r
assignments %>% 
  filter(title != assigned_book) %>% 
  count(title, assigned_book, term, wt = count) %>%
  arrange(desc(n))
#> # A tibble: 4,265 x 4
#>   title              assigned_book         term        n
#>   <chr>              <chr>                 <chr>   <dbl>
#> 1 Great Expectations The War of the Worlds joe's      55
#> 2 Great Expectations The War of the Worlds orlick     51
#> 3 Great Expectations The War of the Worlds black      50
#> 4 Great Expectations The War of the Worlds night      49
#> 5 Great Expectations The War of the Worlds water      46
#> 6 Great Expectations The War of the Worlds kitchen    42
#> # ... with 4,259 more rows
```

We can see that a number of words were often assigned to *Pride and Prejudice* or *The War of the Worlds* cluster even when they appeared in *Great Expectations* or *Twenty Thousand Leagues under the Sea*. For some of these words, such as "Jane", it comes as no surprise that it will be assigned to *Pride and Prejudice*.

It is possible that a word is assigned to a book, even though it never appears in that book. 

## Tuning number of topics

https://juliasilge.com/blog/evaluating-stm/

To compare topic models (not necessarily LDA) with different number of topic $K$, we need first to propose metrics for comparison or topic quality. **Semantic coherence** s maximized when the most probable words in a given topic frequently co-occur together, which correlates human judgement of topic quality.

https://rdrr.io/cran/stm/man/exclusivity.html


```r
data("data_corpus_inaugural", package = "quanteda")
inaugural_counts <- tidy(data_corpus_inaugural) %>%
  mutate(document = str_c(Year, President, sep = "_")) %>% 
  unnest_tokens(word, text) %>% 
  anti_join(stop_words) %>% 
  count(document, word, sort = TRUE)

inaugural_dfm <- inaugural_counts %>% 
  cast_dfm(document = document, term = word, value = n)
```


It took several minutes and quite a lot computing power to run the following code. So 


```r
library(furrr)
plan(multiprocess)

models <- tibble(K = 2:6) %>%
  mutate(topic_model = future_map(K, ~ stm(inaugural_dfm,
                                           init.type = "Spectral",
                                           K = .,
                                           verbose = FALSE)))
```




```r
models
#> # A tibble: 5 x 2
#>       K topic_model
#>   <int> <list>     
#> 1     2 <STM>      
#> 2     3 <STM>      
#> 3     4 <STM>      
#> 4     5 <STM>      
#> 5     6 <STM>
```


```r
heldout <- make.heldout(inaugural_dfm)

k_result <- models %>%
  mutate(exclusivity        = map(topic_model, exclusivity),
         semantic_coherence = map(topic_model, semanticCoherence, inaugural_dfm),
         eval_heldout       = map(topic_model, eval.heldout, heldout$missing),
         residual           = map(topic_model, checkResiduals, inaugural_dfm),
         bound              = map_dbl(topic_model, function(x) max(x$convergence$bound)),
         lfact              = map_dbl(topic_model, function(x) lfactorial(x$settings$dim$K)),
         lbound             = bound + lfact,
         iterations         = map_dbl(topic_model, function(x) length(x$convergence$bound)))
```



```r
k_result %>%
  transmute(K,
            `Lower bound`         = lbound,
            Residuals             = map_dbl(residual, "dispersion"),
            `Semantic coherence`  = map_dbl(semantic_coherence, mean),
            `Held-out likelihood` = map_dbl(eval_heldout, "expected.heldout")) %>%
  pivot_longer(-K, names_to = "metrics", values_to = "value") %>%
  ggplot(aes(K, value, color = metrics)) +
  geom_line(size = 1.5) +
  facet_wrap(~ metrics, scales = "free_y")
```

<img src="topic-modeling_files/figure-html/unnamed-chunk-35-1.png" width="672" style="display: block; margin: auto;" />

