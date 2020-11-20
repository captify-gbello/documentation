### GDPR Classification for feedSource 5


#### Overview

Aim of this project was to develop a classifier that classifies keyphrases as related one of the 5 GDPR categories (or related to none of them). The 5 GDPR categories are:
- health
- religion
- race/ethnicity
- politics/trade union
- sex life/sexual orientation

This was implemented as gRPC service


#### Project location on Captify's codebase
Located here: https://github.com/captify/poly/tree/master/sem/gdpr-model-server


#### Features

fastText embeddings of keyphrases. Averaged over the words in the keyphrase. Find embedding models under s3://captify-semantics/personal/gbello/models/. The fastText embeddings are pre-trained on Common Crawl

- [indirectly represented] word2vec ([`wiki_en.bin`](https://fasttext.cc/docs/en/pretrained-vectors.html)) cosine similarity between clean generic top-level domain (see below) and tier 2 taxonomy names 
- [indirectly represented] word2vec (`wiki_en.bin`) cosine similarity between `pathnquery` (concatenation of keyword feed pixel path, pixel query, referrer path, referrer query) and tier 2 taxonomy names
- [indirectly represented] word2vec embedding (`wiki_en.bin`) of clean generic top-level domain
- URL tier 1 taxonomy category probability (from SEM-69; probability prediction is generated using the 3 features above)


#### Keyphrase Processing Workflow

- Lower case string
- Replace some word-delimiting symbols with whitespace (`re.sub(r"\+|-|_", " ", _)`)
- Tokenize string by `" "`
