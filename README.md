### GDPR Classification for feedSource 5


## Overview

Aim of this project was to develop a classifier that classifies keyphrases as related one of the 5 GDPR categories (or related to none of them). The 5 GDPR categories are:
- health
- religion
- race/ethnicity
- politics/trade union
- sex life/sexual orientation

This was implemented as gRPC service, with server side written in python. This service is called from the keyword enricher.

### Scope of deployment
Meant to be applied to searches in the keyword feed meeting the following criteria:
- feedSource == 5
- country is in `EU_countries` (see list [here](https://github.com/captify/migrations/blob/develop/migrations/src/main/resources/migrations/c3/312_1__fix_bug_add_gdpr_flag_to_sel_country.sql))
- detectedLanguage is in [en, de, fr, it, es]

### Pipeline
this pipeline takes a keyphrase and its detected language as input, and based on detected language, it will classify the keyphrase using the appropriate model for that language.


### General approach
General approach is a language-specific approach, where there is a separate model for each language. Take a look at [SEM-844](https://jira.captifymedia.com/browse/SEM-844) and [SEM-862](https://jira.captifymedia.com/browse/SEM-862).

#### Model training
Data was from translated data.  For each language, training was carried out on auto-translated version of the English dataset (to the language of interest), and validation was carried out on manually annotated datasets created for the language of interest.


### Project location on Captify's codebase
Located here: https://github.com/captify/poly/tree/master/sem/gdpr-model-server

## Models
SVM models, one for each language. Located on MLFlow prod. There is a separate model for each language:
- EN: `GDPR_feedsource5_fastText_classifier_EN`
- FR: `GDPR_feedsource5_fastText_classifier_FR`
- DE: `GDPR_feedsource5_fastText_classifier_DE`
- IT: `GDPR_feedsource5_fastText_classifier_IT`
- ES: `GDPR_feedsource5_fastText_classifier_ES`

### Features

All the models use the same kind of feature: fastText embeddings of the keyphrases (mean word embedding vectors over the words in the keyphrase). Each language uses pre-trained fastText embedding models trained for that language using common crawl data. Find embedding models under s3://captify-semantics/personal/gbello/models/. Under this location, the embedding filenames are:
- EN: `crawl-300d-2M.vec.bin`
- FR: `cc.fr.300.vec.bin`
- DE: `cc.de.300.vec.bin`
- IT: `cc.it.300.vec.bin`
- ES: `cc.es.300.vec.bin`

### Keyphrase Processing Workflow

- Lower case string
- Replace word-delimiting symbols (`+` and `-`) with whitespace
- Tokenize string by `" "`
