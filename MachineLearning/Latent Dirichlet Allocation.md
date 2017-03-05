*简单使用*：
```
# -*- coding: utf-8 -*-
from gensim import corpora
from gensim import models

__author__ = 'Brown'

texts = [
    'The result of our cleaning stage is texts is'.split(),
    'Synonyms for nice at Thesaurus.com with free online thesaurus'.split()
]
dictionary = corpora.Dictionary(texts)
corpus = [dictionary.doc2bow(text) for text in texts]
model = models.ldamodel.LdaModel(corpus, num_topics=2, id2word=dictionary, passes=20)
for idx, topic in model.show_topics(num_topics=2, num_words=5):
    print(topic)
```

<br />

https://rstudio-pubs-static.s3.amazonaws.com/79360_850b2a69980c4488b1db95987a24867a.html
