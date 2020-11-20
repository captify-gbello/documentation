### GDPR Health-Sensitive Searches Classification


#### Overview

Aim of this project was to develop a classifier that predicts whether a health-related search is with personal intent, and therefore GDPR sensitive, or not (e.g. education purposes or pet-related).

2 main source of features have been used: (i) URL; and (ii) extracted keyphrase. URL gives more context as it contains domain and other peripheral information that can help with identifying whether intent is personal or educational or otherwise. E.g. if the website/domain is a dictionary, this is likely educational. If website is nhs, this is likely personal, even if the keyphrase in both is "adenocarcinoma".


#### URL Processing Workflow 

- Remove entries without a `urlDomain`
- Replace empty pixel path, pixel query, referrer path and referrer query with " "
- Concatenate pixel and referrer path/query (`pathnquery`)
- Only keep entries with non empty `pathnquery` (len < 0)
- Clean up the `urlDomain` to only keep the generic top-level domain (i.e. `forums.news.cnn.com`--> cnn)


#### URL Features

- [indirectly represented] word2vec ([`wiki_en.bin`](https://fasttext.cc/docs/en/pretrained-vectors.html)) cosine similarity between clean generic top-level domain (see below) and tier 2 taxonomy names 
- [indirectly represented] word2vec (`wiki_en.bin`) cosine similarity between `pathnquery` (concatenation of keyword feed pixel path, pixel query, referrer path, referrer query) and tier 2 taxonomy names
- [indirectly represented] word2vec embedding (`wiki_en.bin`) of clean generic top-level domain
- URL tier 1 taxonomy category probability (from SEM-69; probability prediction is generated using the 3 features above)


#### Keyphrase Processing Workflow

- Lower case string
- Replace some word-delimiting symbols with whitespace (`re.sub(r"\+|-|_", " ", _)`)
- Tokenize string by `" "`


#### Keyphrase features

- facebook InferSent keyphrase embedding (`infersent2.pkl`), based on fastText `crawl-300d-2M.vec` word embeddings, and NLTK `punkt` tokenizer
- word2vec (`wiki_en.bin`) cosine similarity of keyphrase to veterinary terms e.g. `feline`, `canine` etc.
- number of possessive pronouns and other curated tokens in keyphrase: `does`, `are`, `is`, `why`, `will`, `would`, `do`, `can`, `could`, `am`, `my`, `i`, `we`, `have`, `having`, `our`, `my`, `mine`, `i`,


#### Modeling

- Logistic Regression in `sklearn`
- Default hyper-parameters, `l2` penalty, `liblinear` solver, and `5000` maximum iterations



#### Future Considerations
- model could further be improved with larger sample size, more validation sets
- update sentence embeddings to handle out-of-vocabulary terms
- more feature engineering: richer set of features could be developed, and current features can be improved/refined
- Adjusting classification threshold with PR-AUC, though better approach might be to use "risk scores" (derived from classification probability) rather than discrete classes
