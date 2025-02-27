RoBERTa is a transformers model pretrained on a large corpus of English data in a self-supervised fashion. This means
it was pretrained on the raw texts only, with no humans labelling them in any way (which is why it can use lots of
publicly available data) with an automatic process to generate inputs and labels from those texts. 

More precisely, it was pretrained with the Masked language modeling (MLM) objective. Taking a sentence, the model
randomly masks 15% of the words in the input then run the entire masked sentence through the model and has to predict
the masked words. This is different from traditional recurrent neural networks (RNNs) that usually see the words one
after the other, or from autoregressive models like GPT which internally mask the future tokens. It allows the model to
learn a bidirectional representation of the sentence.

This way, the model learns an inner representation of the English language that can then be used to extract features
useful for downstream tasks: if you have a dataset of labeled sentences for instance, you can train a standard
classifier using the features produced by the BERT model as inputs.

# Training Details

## Training Data

The RoBERTa model was pretrained on the reunion of five datasets:
- [BookCorpus](https://yknzhu.wixsite.com/mbweb), a dataset consisting of 11,038 unpublished books;
- [English Wikipedia](https://en.wikipedia.org/wiki/English_Wikipedia) (excluding lists, tables and headers) ;
- [CC-News](https://commoncrawl.org/2016/10/news-dataset-available/), a dataset containing 63 millions English news
  articles crawled between September 2016 and February 2019.
- [OpenWebText](https://github.com/jcpeterson/openwebtext), an opensource recreation of the WebText dataset used to
  train GPT-2,
- [Stories](https://arxiv.org/abs/1806.02847) a dataset containing a subset of CommonCrawl data filtered to match the
  story-like style of Winograd schemas.

Together theses datasets weight 160GB of text.

## Training Procedure

### Preprocessing

The texts are tokenized using a byte version of Byte-Pair Encoding (BPE) and a vocabulary size of 50,000. The inputs of
the model take pieces of 512 contiguous token that may span over documents. The beginning of a new document is marked
with `<s>` and the end of one by `</s>`

The details of the masking procedure for each sentence are the following:
- 15% of the tokens are masked.
- In 80% of the cases, the masked tokens are replaced by `<mask>`.

- In 10% of the cases, the masked tokens are replaced by a random token (different) from the one they replace.
- In the 10% remaining cases, the masked tokens are left as is.

Contrary to BERT, the masking is done dynamically during pretraining (e.g., it changes at each epoch and is not fixed).

### Pretraining

The model was trained on 1024 V100 GPUs for 500K steps with a batch size of 8K and a sequence length of 512. The
optimizer used is Adam with a learning rate of 4e-4, \\(\beta_{1} = 0.9\\), \\(\beta_{2} = 0.98\\) and
\\(\epsilon = 1e-6\\), a weight decay of 0.01, learning rate warmup for 30,000 steps and linear decay of the learning
rate after.

# Evaluation Results

When fine-tuned on downstream tasks, this model achieves the following results:

Glue test results:

| Task | MNLI | QQP  | QNLI | SST-2 | CoLA | STS-B | MRPC | RTE  |
|:----:|:----:|:----:|:----:|:-----:|:----:|:-----:|:----:|:----:|
|      | 90.2 | 92.2 | 94.7 | 96.4  | 68.0 | 96.4  | 90.9 | 86.6 |

# Limitations and Biases

The training data used for this model contains a lot of unfiltered content from the internet, which is far from
neutral. This bias will also affect all fine-tuned versions of this model.

You can use the raw model for masked language modeling, but it's mostly intended to be fine-tuned on a downstream task.

Note that this model is primarily aimed at being fine-tuned on tasks that use the whole sentence (potentially masked) to make decisions, such as sequence classification, token classification or question answering. For tasks such as text generation you should look at model like GPT2.

# Inference samples

Inference type|Python sample (Notebook)
|--|--|
Real time|[sdk-example.ipynb](https://aka.ms/sdk-notebook-examples)
Real time|[fill-mask-online-endpoint.ipynb](https://aka.ms/fill-mask-online-endpoint-oss)

# Sample inputs and outputs

### Sample input
```json
{
    "input_data": [
        "Paris is the <mask> of France.",
        "Today is a <mask> day!"
    ]
}
```

### Sample output
```json
[
  "capital",
  "great"
]
```
