---
title: Basic Usage
layout: default
nav_order: 1
parent: Code
---

This example will walk you throught the basic usage of PromptBench. We hope that you can get familiar with the APIs and use it in your own projects later.

First, there is a unified import of `import promptbench as pb` that easily imports the package.


```python
import promptbench as pb
```

## Load dataset

First, PromptBench supports easy load of datasets.


```python
# print all supported datasets in promptbench
print('All supported datasets: ')
print(pb.SUPPORTED_DATASETS)

# load a dataset, sst2, for instance.
# if the dataset is not available locally, it will be downloaded automatically.
dataset = pb.DatasetLoader.load_dataset("sst2")

# print the first 5 examples
dataset[:5]
```

    All supported datasets: 
    ['cola', 'sst2', 'qqp', 'mnli', 'mnli_matched', 'mnli_mismatched', 'qnli', 'wnli', 'rte', 'mrpc', 'mmlu', 'squad_v2', 'un_multi', 'iwslt', 'math', 'bool_logic', 'valid_parentheses', 'gsm8k', 'csqa', 'bigbench_date', 'bigbench_object_tracking']

    [{'content': "it 's a charming and often affecting journey . ", 'label': 1},
     {'content': 'unflinchingly bleak and desperate ', 'label': 0},
     {'content': 'allows us to hope that nolan is poised to embark a major career as a commercial yet inventive filmmaker . ',
      'label': 1},
     {'content': "the acting , costumes , music , cinematography and sound are all astounding given the production 's austere locales . ",
      'label': 1},
     {'content': "it 's slow -- very , very slow . ", 'label': 0}]



## Load models

Then, you can easily load LLM models via promptbench.


```python
# print all supported models in promptbench
print('All supported models: ')
print(pb.SUPPORTED_MODELS)

# load a model, flan-t5-large, for instance.
model = pb.LLMModel(model='google/flan-t5-large', max_new_tokens=10)
```

    All supported models: 
    ['google/flan-t5-large', 'llama2-7b', 'llama2-7b-chat', 'llama2-13b', 'llama2-13b-chat', 'llama2-70b', 'llama2-70b-chat', 'phi-1.5', 'gpt-3.5-turbo', 'gpt-4', 'gpt-4-1106-preview', 'gpt-3.5-turbo-1106', 'vicuna-7b', 'vicuna-13b', 'vicuna-13b-v1.3', 'google/flan-ul2']

    You are using the default legacy behaviour of the <class 'transformers.models.t5.tokenization_t5.T5Tokenizer'>. This is expected, and simply means that the `legacy` (previous) behavior will be used so nothing changes for you. If you want to use the new behaviour, set `legacy=False`. This should only be set if you understand what it means, and thouroughly read the reason why this was added as explained in https://github.com/huggingface/transformers/pull/24565
    Special tokens have been added in the vocabulary, make sure the associated word embeddings are fine-tuned or trained.


## Construct prompts

Prompts are the key interaction interface to LLMs. You can easily construct a prompt by call the Prompt API.


```python
# Prompt API supports a list, so you can pass multiple prompts at once.
prompts = pb.Prompt(["Classify the sentence as positive or negative: {content}",
                     "Determine the emotion of the following sentence as positive or negative: {content}"
                     ])
```

You may need to define the projection function for the model output.
Since the output format defined in your prompts may be different from the model output.
For example, for sst2 dataset, the label are '0' and '1' to represent 'negative' and 'positive'.
But the model output is 'negative' and 'positive'.
So we need to define a projection function to map the model output to the label.


```python
def proj_func(pred):
    mapping = {
        "positive": 1,
        "negative": 0
    }
    return mapping.get(pred, -1)
```

## Perform evaluation using prompts, datasets, and models

Finally, you can perform standard evaluation using the loaded prompts, datasets, and labels.


```python
from tqdm import tqdm
for prompt in prompts:
    preds = []
    labels = []
    for data in tqdm(dataset):
        # process input
        input_text = pb.InputProcess.basic_format(prompt, data)
        label = data['label']
        raw_pred = model(input_text)
        # process output
        pred = pb.OutputProcess.cls(raw_pred, proj_func)
        preds.append(pred)
        labels.append(label)
    
    # evaluate
    score = pb.Eval.compute_cls_accuracy(preds, labels)
    print(f"{score:.3f}, {prompt}")
```

    100%|██████████| 872/872 [02:16<00:00,  6.37it/s]


    0.947, Classify the sentence as positive or negative: {content}


    100%|██████████| 872/872 [02:18<00:00,  6.29it/s]

    0.947, Determine the emotion of the following sentence as positive or negative: {content}


    

