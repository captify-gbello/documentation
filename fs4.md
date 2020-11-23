# GDPR Classification for feedSource 4 searches


## Background

GDPR regulations bar us from using [sensitive search data](https://ico.org.uk/for-organisations/guide-to-data-protection/guide-to-the-general-data-protection-regulation-gdpr/special-category-data/what-is-special-category-data/). At Captify, the main categories of sensitive data we are concerned with are:
- personal data revealing **racial or ethnic origin**
- data concerning one's **health** 
- personal data revealing **political opinions** and **trade union membership**
- personal data revealing **religious/philosophical beliefs**
- data concerning a person’s **sex life** or **sexual orientation**

We are not permitted to process (including collection or storage) personal searches that fall into these special categories without a proper legal basis, which for these categories is usually explicit consent - however in most cases we will not be able to get explicit consent.

So in our keyword feed data that we get from various publishers, we want to remove any searches that is:
1. related to any one of the 5 categories above, and
2. of a ‘personal’ nature (more details on this in section [Stage II: GDPR sensitivity prediction](#stage-ii-gdpr-sensitivity-prediction) )

To be able to detect such searches, we use an ML approach. The diagram below summarises the pipeline as it is currently:

![feedSource 4 GDPR pipeline](https://raw.githubusercontent.com/captify/notebooks/master/GDPR/files/GDPR_pipeline.png?token=AL5L5INTUD6AYFZOHMPHGE27YE4HY)


In the above image, `Search Event` here is a bundle consisting of the following search fields:   
- keyphrase
- url domain
- keywords:  list of pixel and referrer path/query keywords: [pixel.path, pixel.query, referrer.path, referrer.query]

### Scope of deployment
Meant to be applied to searches in the keyword feed meeting the following criteria:
- feedSource == 4
- country is in EU_countries (see list here)
- detectedLanguage is English

Below we go into details about **Stage I** and **Stage II** in the pipeline.

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

So in Stage II, for each GDPR category, there is a separate model trained to distinguish personal from non-personal searches within each category. At the moment, all of these are SVM classifiers. For these classifiers, all features are generated from only 2 search attributes: keyphrase and url
- Keyphrase-based features:
  - Keyphrase embeddings: embeddings for keyphrase text
- URL-based features:
  - Embeddings of URL domain
  - Embeddings of URL and referrer path and query keywords
  
The models are all in MLFlow prod. Here are the model names:
- Race/Ethnicity: `GDPR_feedsource4_en_ethnicity_classifier`
- Health: `GDPR_feedsource4_en_health_classifier`
- Politics/Trade Union: `GDPR_feedsource4_en_politics_classifier`
- Religion: `GDPR_feedsource4_en_religion_classifier`
- Sex life/orientation: `GDPR_feedsource4_en_sexlife_classifier`

| GDPR Category |   Model name  |
| ------------- | ------------- |
| Race/Ethnicity  | `GDPR_feedsource4_en_ethnicity_classifier`  |
| Health  | `GDPR_feedsource4_en_health_classifier`  |
| Politics/Trade Union  | `GDPR_feedsource4_en_politics_classifier`  |
| Religion  | `GDPR_feedsource4_en_religion_classifier`  |
| Sex life/orientation  | `GDPR_feedsource4_en_sexlife_classifier`  |


### Project location on Captify's codebase
Located here: https://github.com/captify/poly/tree/master/sem/gdpr-model-server/

