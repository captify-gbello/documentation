# GDPR Sensitive Searches Classification


## Overview

Aim of this project was to develop a classifier that predicts whether an x-related search is with personal intent, and therefore GDPR sensitive, or not.

![feedSource 4 GDPR pipeline](https://raw.githubusercontent.com/captify/notebooks/master/GDPR/files/GDPR_pipeline.png?token=AL5L5INTUD6AYFZOHMPHGE27YE4HY)


In the above image, `Search Event` here is a bundle consisting of the following search fields:   
- keyphrase
- url domain
- keywords:  list of pixel and referrer path/query keywords: [pixel.path, pixel.query, referrer.path, referrer.query]


### Stage I: Category Matching
Currently unsupervised, similar to approach used in user profiles, taxonomy classifier. To check if keyphrase belongs to GDPR category `c` :
- Use seed words specific for `c`
- Mean cosine similarity of words/tokens in keyphrase to seed word cluster centroid for `c`
  - NOTE: 
    - seed words for category `c` are selected set of words related to this category (see **Seed Words** subsection below for examples)
    - cosine similarity is computed on embeddings of keyphrase and seed word cluster
- If mean cosine similarity exceeds a preset similarity threshold, then the keyphrase is assigned as GDPR category `c`.

##### Seed Words
These are available here: s3://captify-semantics/personal/gbello/GDPR/SeedWords/words_and_embeddings/en/GDPR_SeedWords_embeddings_CENTROIDS_ftCrawl300d2M.csv
Here is a small sample of seed words for each GDPR category:

![short samples of seed words from each GDPR category](https://raw.githubusercontent.com/captify/notebooks/master/GDPR/files/GDPR_SeedWords.png?token=AL5L5IK5VQNXR6N2MQTEHHC7YE554)

### Stage II: GDPR sensitivity prediction
A search is not automatically GDPR sensitive merely because it is related to health, religion, ethnicity, or other GDPR category. For it to be sensitive, it needs to also be of a 'personal' nature. Here is how 'personal' is defined for the purpose of this project:
- will the search {keyphrase + url} allow me to infer (with reasonable certainty) the religion/ health status/sex life or orientation/ethnicity/political affiliation of the searcher? If so, then it is 'personal'.    

Therefore, Stage II of the pipeline tries to make distinction between searches of a 'personal', sensitive nature, and generic searches that do not reveal any 'personal' details. Here is an example, from the 'Health' category. The 2 searches below are related to health, and would be flagged in Stage I and passed to Stage II. However, only one of them is of a 'personal' nature (the first one):
- “_will pregnancy affect my diabetes?_” (![#f03c15](https://via.placeholder.com/15/f03c15/000000?text=+) sensitive!)
- “_statistics on dementia in uk_” (![#c5f015](https://via.placeholder.com/15/c5f015/000000?text=+) not sensitive)


For a keyphrase designated as related to some GDPR category, the Stage II model predict if it is of a ‘personal’ nature or not. 
Needs to also be of a ‘personal’ nature
‘personal’: will search {keyphrase + url} allow me to infer (with reasonable certainty) the religion/ health status/sex life or orientation/ethnicity/political affiliation of the searcher?*

For example, if Stage I matches a keyphrase as related to religion, it is fed to a religion-specific binary classifier that determines if the search is of a personal nature or not. 
There is a separate such classifier for each GDPR category. At the moment, all of these are SVM classifiers. For these classifiers, all features are generated from only 2 search attributes: keyphrase and url
- Keyphrase-based features:
  - Keyphrase embeddings: embeddings for keyphrase text
- URL-based features:
  - Embeddings of URL domain
  - Embeddings of URL and referrer path and query keywords
  
  

#### Keyphrase Processing Workflow

- Lower case string
- Replace some word-delimiting symbols with whitespace (`re.sub(r"\+|-|_", " ", _)`)
- Tokenize string by `" "`



