<a href="https://explosion.ai"><img src="https://explosion.ai/assets/img/logo.svg" width="125" height="125" align="right" /></a>

# spaCy + StanfordNLP

This package...

[![Travis](https://img.shields.io/travis/explosion/spacy_stanfordnlp/master.svg?style=flat-square&logo=travis)](https://travis-ci.org/explosion/spacy_stanfordnlp)
[![Appveyor](https://img.shields.io/appveyor/ci/explosion/spacy_stanfordnlp/master.svg?style=flat-square&logo=appveyor)](https://ci.appveyor.com/project/explosion/spacy_stanfordnlp)
[![PyPi](https://img.shields.io/pypi/v/spacy_stanfordnlp.svg?style=flat-square)](https://pypi.python.org/pypi/spacy_stanfordnlp)
[![GitHub](https://img.shields.io/github/release/explosion/spacy_stanfordnlp/all.svg?style=flat-square)](https://github.com/explosion/spacy_stanfordnlp)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg?style=flat-square)](https://github.com/ambv/black)

- Statistical tokenization (reflected in the `Doc` and its tokens)
- Lemmatization (`token.lemma` and `token.lemma_`)
- Part-of-speech tagging (`token.tag`, `token.tag_`, `token.pos`, `token.pos_`)
- Dependency parsing (`token.dep`, `token.dep_`, `token.head`)
- Sentence segmentation (`doc.sents`)

## Installation

```bash
pip install spacy_stanfordnlp
```

To use the latest `spacy-nightly`, run the following afterwards:

```bash
pip uninstall spacy
pip install spacy-nightly
```

Make sure to also install one of the
[pre-trained StanfordNLP models](https://stanfordnlp.github.io/stanfordnlp/installation_download.html).

## Usage & Examples

The `StanfordNLPLanguage` class can be initialized with a loaded StanfordNLP
pipeline and returns a spaCy [`Language` object](https://spacy.io/api/language),
i.e. the `nlp` object you can use to process text and create a
[`Doc` object](https://spacy.io/api/doc).

```python
import stanfordnlp
from spacy_stanfordnlp import StanfordNLPLanguage

snlp = stanfordnlp.Pipeline(lang="en")
nlp = StanfordNLPLanguage(snlp)

doc = nlp("Barack Obama was born in Hawaii. He was elected president in 2008.")
for token in doc:
    print(token.text, token.lemma_, token.pos_, token.dep_)
```

If language data for the given language is available in spaCy, the respective
language class will be used as the base for the `nlp` object – for example,
`English()`. This lets you use spaCy's lexical attributes like `is_stop` or
`like_num`. The `nlp` object follows the same API as any other spaCy `Language`
class – so you can visualize it with displaCy, add custom components to it, use
the rule-based matcher and do pretty much anything else you'd normally do in
spaCy.

```python
# Access spaCy's lexical attributes
print([token.is_stop for token in doc])
print([token.like_num for token in doc])

# Visualize dependencies
from spacy import displacy
displacy.serve(doc)  # or displacy.render if you're in a Jupyter notebook

# Efficient processing with nlp.pipe
for doc in nlp.pipe(["Lots of texts", "Even more texts", "..."]):
    print(doc.text)

# Combine with your own custom pipeline components
def custom_component(doc):
    # Do something to the doc here
    return doc

nlp.add_pipe(custom_component)

# Serialize it to a numpy array
np_array = doc.to_array(['ORTH', 'LEMMA', 'POS'])
```

### Advanced: serialization and entry points

> ⚠️ **Important note:** This feature requires spaCy v.2.1.x, currently
> available as `spacy-nightly`. To use this component with the nightly version,
> uninstall `spacy` and then re-install `spacy-nightly`.

The spaCy `nlp` object created by `StanfordNLPLanguage` exposes its language as
`stanfordnlp_xx`.

```python
from spacy.util import get_lang_class
lang_cls = get_lang_class("stanfordnlp_en")
```

Normally, the above would fail because spaCy doesn't include a language class
`stanfordnlp_en`. But because this package exposes a `spacy_languages` entry
point in its [`setup.py`](setup.py) that points to `StanfordNLPLanguage`, spaCy
knows how to initialize it.

This means that saving to and loading from disk works:

```python
snlp = stanfordnlp.Pipeline(lang="en")
nlp = StanfordNLPLanguage(snlp)
nlp.to_disk("./stanfordnlp-spacy-model")
```

Additional arguments on `spacy.load` are automatically passed down to the
language class and pipeline components. So when loading the saved model, you can
pass in the `snlp` argument:

```python
snlp = stanfordnlp.Pipeline(lang="en")
nlp = spacy.load("./stanfordnlp-spacy-model", snlp=snlp)
```

Note that this **will not save any model data by default**. The StanfordNLP
models are very large, so for now, this package expects that you load them
separately.