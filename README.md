### GDPR Health-Sensitive Searches Classification


#### Overview

Aim of this project was to develop a classifier that predicts whether a health-related search is with personal intent, and therefore GDPR sensitive, or not (e.g. education purp


#### URL Features

- [indirectly represented] word2vec ([`wiki_en.bin`](https://fasttext.cc/docs/en/pretrained-vectors.html)) cosine similarity between clean generic top-level domain (see below) and tier 2 taxonomy names 
- [indirectly represented] word2vec (`wiki_en.bin`) cosine similarity between `pathnquery` (concatenation of keyword feed pixel path, pixel query, referrer path, referrer query) and tier 2 taxonomy names
- [indirectly represented] word2vec embedding (`wiki_en.bin`) of clean generic top-level domain
- URL tier 1 taxonomy category probability (from SEM-69; probability prediction is generated using the 3 features above)


#### Keyphrase Processing Workflow

- Lower case string
- Replace some word-delimiting symbols with whitespace (`re.sub(r"\+|-|_", " ", _)`)
- Tokenize string by `" "`
