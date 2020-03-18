# <span style="font-variant:small-caps;">x</span>-stance

Documentation and evaluation script accompanying the paper ["X-Stance: A Multilingual Multi-Target Dataset for Stance Detection"](https://arxiv.org/).

The data can be downloaded [here](https://www.cl.uzh.ch/dam/jcr:f866f285-7784-4d04-a828-d3c2a9242776/xstance-data-v1.0.zip). A detailed description can be found in the [paper](https://arxiv.org/).

## Summary

The <span style="font-variant:small-caps;">x</span>-stance dataset contains more than **150 political questions**, and **67k comments** written by candidates on those questions.

It can be used to train and evaluate stance detection systems.

The comments are partly **German**, partly **French** and **Italian**. The questions are available in all the three languages plus **English**.

The data have been extracted from the Swiss voting advice platform [Smartvote](https://smartvote.ch/).

Data example:

<img alt="Data Example" src="example.png" width="700">

## Structure

The dataset contains the following files:
- *train.jsonl*
- *valid.jsonl*
- *test.jsonl*
- *questions.{en,de,fr,it}.jsonl*

Example for a train, valid or test instance:

```json
{
   "id": 20475,
   "language": "de",
   "question_id": 3469,
   "question": "Soll der Bundesrat ein Freihandelsabkommen mit den USA anstreben?",
   "comment": "Nicht unter einem Präsidenten, welcher die Rechte anderer mit Füssen tritt und Respektlos gegenüber ändern ist.",
   "label": "AGAINST",
   "numerical_label": 0,
   "author": "8aa829c3b86f",
   "topic": "Foreign Policy",
   "test_set": "new_comments_defr"
}
```

Details:
- Languages: The files *train.jsonl* and *valid.jsonl* contain about 75% German data and 25% French data. The file *test.jsonl* also contains some Italians samples to test zero-shot cross-lingual transfer.
- `"label"` can be `"FAVOR"` or `"AGAINST"`.
- `"numerical_label"` provides a more fine-grained label (not used in our baseline). Range of values: {0, 25, 75, 100}, where 0 means "no" and 100 means "yes".
- `"test_set"`: Only *test.jsonl* has this field. Specifies the test partition (new comments / new questions / new topics; German+French / Italian). For details on the test partitions please refer to Table 2 in the paper.

In the train, valid and test files, the comments are paired with a version of the question in the same language (e.g. German comment + German version of the question). The *questions.xx.jsonl* files provide complete translations of all the questions.

## Evaluation

Dependencies: Python 3; `scikit-learn`

Downloading the data:
```bash
wget https://www.cl.uzh.ch/dam/jcr:f866f285-7784-4d04-a828-d3c2a9242776/xstance-data-v1.0.zip
unzip xstance-data-v1.0.zip -d data
```

Usage:
```bash
python evaluate.py \
  --gold data/test.jsonl \
  --pred predictions/pred.jsonl 
```

The predictions file should be a JSON lines file (http://jsonlines.org/). The lines in the file should correspond to the lines in the gold file (*test.jsonl*).

Example prediction:
```json
{"label": "AGAINST"}
```

The evaluation script outputs the macro-average of the F1 score for each label, per test partition and per language:

```
new_comments_defr
DE 76.83541377429334
FR 76.61281705054353

new_questions_defr
DE 68.46881591336131
FR 68.3831150794995

new_topics_defr
DE 68.90323152487849
FR 70.8982523359103

new_comments_it
IT 70.19234360410832
```

## BERT Baseline
Dependencies:
- Python 3
- AllenNLP 0.9.0 (http://docs.allennlp.org/master/)
- `pip install -r baseline/requirements.txt`
- The commands below assume GPU computation. They can be adapted for CPU, however.

Downloading the data (if not done in the previous section):
```bash
wget https://www.cl.uzh.ch/dam/jcr:f866f285-7784-4d04-a828-d3c2a9242776/xstance-data-v1.0.zip
unzip xstance-data-v1.0.zip -d data
```

Training:
```bash
cd baseline
allennlp train smartvote.jsonnet \
    --include-package allennlp_xstance \
    -s mymodel
```

Predicting:
```bash
cd baseline
allennlp predict mymodel ../data/test.jsonl \
    --include-package allennlp_xstance \
    --predictor smartvote_predictor \
    --cuda-device 0 \
    --output-file ../predictions/mypred.jsonl
```

Evaluating:
```bash
cd ..
python evaluate.py \
  --gold data/test.jsonl \
  --pred predictions/mypred.jsonl 
```


## Licenses
- This repository: MIT License
- Dataset: CC BY-NC 4.0 (© www.smartvote.ch)

## References
The dataset and baseline model are described in:

```bibtex
@article{vamvas2020xstance,
  title={X-Stance: A Multilingual Multi-Target Dataset for Stance Detection},
  author={Vamvas, Jannis and Sennrich, Rico},
  journal={arXiv preprint arXiv:XXXXXX},
  year={2020}
}
```
