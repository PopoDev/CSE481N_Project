# CSE481N Reproducibility Project

This repo's goal is to reproduce the results of the paper [Speculative Decoding with Big Little Decoder (BiLD)](https://proceedings.neurips.cc/paper_files/paper/2023/file/7b97adeafa1c51cf65263459ca9d0d7c-Paper-Conference.pdf) by Kim et al. (2023). The paper is published in NeurIPS 2023 and the code can be found on the [official repository](https://github.com/kssteven418/BigLittleDecoder).

## Introduction

Big Little Decoder is a simple framework that enables **faster generative inference**. 
It can dramatically accelerate text generation by ~2x, without compromising performance on a variety of text generation scenarios. 
Furthermore, it is a simple **plug-and-play** solution that requires no training or architecture redesign.

Here's the key underlying idea:

1. BiLD offloads the majority of simple word decisions to a smaller model, and only switches the control back to the larger model when needed.
2. The small model **"fallbacks"** to the large model, when it runs into a hard-to-predict word.
3. In case the small model makes a misstep, the larger model can **"rollback"** the predictions to correct the error
4. This **collaborative text generation** combines the small model's fast autoregressive execution with the large model's accurate and efficient non-autoregressive execution!

# Reproduction

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/PopoDev/CSE481N_Project/blob/main/run.ipynb)

## Requirements

Install the required packages using the following command:

```bash
pip install -r requirements.txt
```

## Evaluation

We provide the scripts to evaluate the pretrained models on the datasets used in the paper. The scripts are located in the root directory and follow the naming convention `run_<experiment>.sh`.

```bash
# Syntax for running experiments
bash ./run_<experiment>.sh [model] [fallback_threshold] [rollback_threshold] [max_eval_samples]
```

Arguments
- `model`: Specifies the model configuration to use. Possible values are:
    - vanilla: Uses only the large model.
    - unaligned: Uses BiLD with a large model and small model.
    - aligned: Uses BilD with a large model and small aligned model.
- `fallback_threshold`: (Optional) Specifies the fallback threshold for the small model. Values can range from [0.1, 0.9].
- `rollback_threshold`: (Optional) Specifies the rollback threshold for the large model. Values can range from [1, 10].
- `max_eval_samples`: (Optional) Specifies the maximum number of samples to evaluate.

### Translation

To evaluate the pretrained models on the translation datasets, run the following command:

```bash
# IWSLT2017 with vanilla model
CUDA_VISIBLE_DEVICES=0 bash run_iwslt2017.sh vanilla

# WMT2014 with BiLD unaligned model
CUDA_VISIBLE_DEVICES=0 bash run_wmt2014.sh unaligned
```

### Summarization

To evaluate the pretrained models on the summarization datasets, run the following command:

```bash
# XSUM with BiLD aligned model
CUDA_VISIBLE_DEVICES=0 bash run_xsum.sh aligned

# CNNDM with BiLD aligned model with (FB, RB)=(0.5, 5.0) over 10 samples
CUDA_VISIBLE_DEVICES=0 bash run_cnndm.sh aligned 0.5 5 10
```

## Results

### Original

|            | IWSLT |         |          | WMT    |         |          | XSUM   |         |          | CNNDM  |         |          |
|------------|-------|---------|----------|--------|---------|----------|--------|---------|----------|--------|---------|----------|
|            | BLEU  | Speedup | (RB,FB)  | BLEU   | Speedup | (RB,FB)  | RL     | Speedup | (RB,FB)  | RL     | Speedup | (RB,FB)  |
| Unaligned  | 40.33 | 1.43×   | (2, 0.6) | 31.28  | 1.34×   | (2, 0.6) | 35.12  | 1.48×   | (3, 0.5) | 41.44  | 1.71×   | (3, 0.4) |
|            | 39.44 | 1.58×   | (3, 0.5) | 30.47  | 1.43×   | (3, 0.5) | 34.02  | 1.72×   | (5, 0.3) | 40.57  | 2.05×   | (6, 0.2) |
| Aligned    | 40.24 | 1.62×   | (3, 0.9) | 31.26  | 1.47×   | (2, 0.8) | 35.05  | 1.50×   | (2, 0.6) | 41.52  | 1.85×   | (3, 0.3) |
|            | 39.13 | 1.78×   | (4, 0.6) | 30.33  | 1.70×   | (5, 0.6) | 33.95  | 1.80×   | (5, 0.4) | 40.96  | 2.12×   | (6, 0.2) |


### Reproduction 

| IWSLT2017          | Thresholds | Original           | Reproduction       |
|--------------------|------------|--------------------|--------------------|
| BiLD (unaligned)   | (2, 0.6)   | 40.33 & 1.43x      | 40.33 & 1.46x      |
|                    | (3, 0.5)   | 39.44 & 1.58x      | 39.44 & 1.61x      |
| BiLD (aligned)     | (3, 0.9)   | 40.24 & 1.62x      | 40.09 & 1.52x      |
|                    | (4, 0.6)   | 39.13 & 1.62x      | 39.15 & 1.73x      |

- For the BLUE scores, we were able to reproduce the results within 1% of the original scores.
- For the speedup, we were able to reproduce the results within 5% of the reported values.

| WMT14              | Thresholds | Original      | Reproduction  |
|--------------------|------------|---------------|---------------|
| BiLD (unaligned)   | (2, 0.6)   | 31.28 & 1.34× | 31.65 & 1.27× |
|                    | (3, 0.5)   | 30.47 & 1.43× | 30.83 & 1.34× |
| BiLD (aligned)     | (2, 0.8)   | 31.26 & 1.47× | 31.65 & 1.37× |
|                    | (5, 0.6)   | 30.33 & 1.70× | 30.54 & 1.65× |

- For the BLUE scores, we were able to reproduce the results within 1% of the original scores.
- For the speedup, we were able to reproduce the results within 10% of the reported values.

| XSUM           | Thresholds | Original | Reproduction |
|----------------|------------|----------|--------------|
| BiLD (unaligned) | (3, 0.5)  | 35.12 & 1.48× | 35.12 & 1.40× |
|                  | (5, 0.3)  | 34.02 & 1.72× | 34.02 & 1.54× |
| BiLD (aligned)   | (2, 0.6)  | 35.05 & 1.50× | 34.96 & 1.41× |
|                  | (5, 0.4)  | 33.95 & 1.80× | 33.96 & 1.73× |

- For the ROUGE-L scores, we were able to reproduce the results within 1% of the original scores.
- For the speedup, we were able to reproduce the results within 10% of the reported values.

| CNNDM           | Thresholds | Original | Reproduction |
|-----------------|------------|----------|--------------|
| BiLD (unaligned) | (3, 0.4)  | 41.44 & 1.71× | 41.44 & 1.54× |
|                  | (6, 0.2)  | 40.57 & 2.05× | 40.56 & 1.87× |
| BiLD (aligned)   | (3, 0.3)  | 41.52 & 1.85× | 41.33 & 1.48× |
|                  | (6, 0.2)  | 40.96 & 2.12× | 40.79 & 1.67× |

- For the ROUGE-L scores, we were able to reproduce the results within 1% of the original scores.
- For the speedup, we were able to reproduce the results within 20% of the reported values.

## Pretrained Checkpoints

### Authors' Checkpoints

The authors provided the finetuned checkpoints used in the paper.

| Dataset |  Model | Link |
| -------- | -------- | -------- | 
| IWSLT-2017-De-En    |  mT5-small  |  [link](https://huggingface.co/kssteven/mT5-small-iwslt2017-de-en) | 
| IWSLT-2017-De-En    |  mT5-small (aligned)  |  [link](https://huggingface.co/kssteven/mT5-small-iwslt2017-de-en-bild-aligned) | 
| IWSLT-2017-De-En    |  mT5-large  |  [link](https://huggingface.co/kssteven/mT5-large-iwslt2017-de-en) | 
| WMT-2014-De-En    |  mT5-small  |  [link](https://huggingface.co/kssteven/mT5-small-wmt2014-de-en) | 
| WMT-2014-De-En    |  mT5-small (aligned)  |  [link](https://huggingface.co/kssteven/mT5-small-wmt2014-de-en-bild-aligned) | 
| WMT-2014-De-En    |  mT5-large  |  [link](https://huggingface.co/kssteven/mT5-large-wmt2014-de-en) | 
| XSUM    |  T5-small  |  [link](https://huggingface.co/kssteven/T5-small-xsum) | 
| XSUM    |  T5-small (aligned)  |  [link](https://huggingface.co/kssteven/T5-small-xsum-bild-aligned) | 
| XSUM    |  T5-large  |  [link](https://huggingface.co/kssteven/T5-large-xsum) | 
| CNNDM    |  T5-small  |  [link](https://huggingface.co/kssteven/T5-small-cnndm) | 
| CNNDM    |  T5-small (aligned) |  [link](https://huggingface.co/kssteven/T5-small-cnndm-bild-aligned) | 
| CNNDM    |  T5-large  |  [link](https://huggingface.co/kssteven/T5-large-cnndm) | 

### Aligned Models
We trained our own aligned models using the outputs of the authors' large finetuned models on each of the four benchmarks. We prove the links to these aligned models below.

| Benchmark for Alignment | Link |
| ----------------- | ---- |
| IWSLT-2017-De-En  | [link](https://huggingface.co/paulh27/iwslt_aligned_smallT5_cont0) |
| WMT-2014-De-En    | [link](https://huggingface.co/paulh27/wmt_aligned_smallmT5) |
| XSUM              | [link](https://huggingface.co/paulh27/xsum_aligned_smallmT5) |
| CNNDM             | [link](https://huggingface.co/paulh27/cnn_aligned_smallT5) |

### Alignment Datasets
The general idea of alignment is to align the predictions produced by the small and large models. As part of this process, we require a calibration dataset for each benchmark which represents the output sentence distribution of the large model. Then, we fine-tune the small model on this dataset so that it will better follow the output distribution of the large model. To create the calibration dataset for a particular benchmark, we take the inputs of the benchmark dataset and generate the corresponding output sequence using the large model, which creates the (input, output) dataset samples. 

The authors did not open source the calibration datasets. As such, we had to create these ourselves, which we link below.

| Calibration Dataset | Link |
| ----------------- | ---- |
| IWSLT-2017-De-En  | [link](https://huggingface.co/datasets/paulh27/alignment_iwslt2017_de_en) |
| WMT-2014-De-En    | [link](https://huggingface.co/datasets/paulh27/alignment_wmt2014_de_en) |
| XSUM              | [link](https://huggingface.co/datasets/lilferrit/xsum_t5_distillation) |
| CNNDM             | [link](https://huggingface.co/datasets/lilferrit/cnn_dailymail_t5_distillation) |

## Self-trained small models

In addition to using model checkpoints provided by the original paper, we also trained our own
small models from scratch targeting a more complete reproduction of the performance pattern
between aligned and unaligned models. The pattern in the original paper was that aligned models
offer a larger speedup while having a marginal performance difference with the unaligned models.
Despite only training our small models for 100,000 steps, compared to the 500,000 steps in the
original paper, we were largely able to reproduce this pattern. A table showing the complete
benchmarks of our small models can be found below. Our self trained models were trained and
benchmarked on Nvidia RTX 2080 GPUs provided by the UW NLPG machines.

|                  | Speedup | Fallback Percentage | Rollback Percentage | Rouge L / BLEU  |
|------------------|---------|---------------------|---------------------|-----------------|
| **XSum Vanilla** | NA      | NA                  | NA                  | 35.10 (Rouge L) |
| **XSum Aligned** | 1.62x   | 10.90%              | 4.125%              | 34.28 (Rouge L) |
| **XSum Unaligned** | 1.35x   | 30.43%              | 3.008%              | 34.26 (Rouge L) |
| **WMT14 Vanilla** | NA      | NA                  | NA                  | 31.88 (BLEU)    |
| **WMT14 Aligned** | 1.60x   | 3.230%              | 2.327%              | 28.71 (BLEU)    |
| **WMT14 Unaligned** | 1.41x   | 16.36%              | 2.359%              | 27.81 (BLEU)    |

## Further Extensions
As part of the reproduction, we additionally conducted several experiments not done in the paper to probe the robustness of the Big-Little decoding architecture. 

### Code API Generation Task
As an additional experiment, we tested Big Little Decoder on a Code API Generation task availabe on GitHub [CodeTrans](https://github.com/agemagician/CodeTrans). The dataset is published on Huggingface and can be found [here](https://huggingface.co/datasets/paulh27/java_code_api_generation). We used the pretrained [small](https://huggingface.co/SEBIS/code_trans_t5_small_api_generation_transfer_learning_finetune) and [large](https://huggingface.co/SEBIS/code_trans_t5_large_api_generation_transfer_learning_finetune) T5 models given by the authors.

```bash
# API Generation with BiLD unaligned model
CUDA_VISIBLE_DEVICES=0 bash run_api.sh unaligned
```



