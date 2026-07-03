# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository overview

This is the official [PyTorch](https://github.com/pytorch/pytorch) `examples` repository: a collection of small, independent, self-contained scripts demonstrating how to use PyTorch for common model types and training patterns. There is no shared library, build system, or test suite tying the examples together — each top-level directory is its own standalone project with its own `main.py`/`train.py` entry point and its own `requirements.txt`.

Directories:
- `dcgan/` — DCGAN (Deep Convolutional GAN) on LSUN/CIFAR10/ImageNet/folder/lfw datasets
- `fast_neural_style/` — Fast neural style transfer (train + stylize), code lives under `fast_neural_style/neural_style/`
- `imagenet/` — ImageNet training/evaluation with torchvision architectures (ResNet, AlexNet, VGG, ...)
- `mnist/` — Basic MNIST convnet
- `mnist_hogwild/` — Hogwild (asynchronous, lock-free) multiprocess training of a shared MNIST convnet
- `regression/` — Fits a 4th-degree polynomial with a single fully-connected layer
- `reinforcement_learning/` — REINFORCE and actor-critic on CartPole (OpenAI Gym)
- `snli/` — Natural Language Inference on SNLI using GloVe vectors, LSTMs, and torchtext
- `super_resolution/` — Sub-pixel convolution super-resolution network (BSD300 dataset)
- `time_sequence_prediction/` — LSTMCell-based sine wave prediction (toy example)
- `vae/` — Variational Autoencoder on MNIST
- `word_language_model/` — Word-level RNN (RNN_TANH/RNN_RELU/LSTM/GRU) language modeling on PTB, with a `generate.py` for sampling from a trained model

There is no root-level build, lint, or CI configuration — treat each example directory independently.

## Working in an example directory

Each example is run from within its own directory. The general pattern:

```bash
cd <example_dir>
pip install -r requirements.txt   # most examples have one; a few (regression, time_sequence_prediction) have none
python main.py [args]             # or train.py, neural_style/neural_style.py, etc. — see below
```

There are no unit tests, linters, or formatters configured anywhere in this repo. "Testing" a change means actually running the script (usually for a short number of epochs/iterations) and checking it trains without error and produces sane output/artifacts.

### Per-example entry points and quirks

- **dcgan**: `python main.py --dataset <cifar10|lsun|imagenet|folder|lfw> --dataroot <path>` (`--dataset`/`--dataroot` required). Writes `real_samples.png`/`fake_samples.png` every 100 iterations and `netG_epoch_%d.pth`/`netD_epoch_%d.pth` every epoch.
- **imagenet**: `python main.py -a <arch> <imagenet-folder-with-train-and-val>` — positional dataset dir is required; use `--lr 0.01` for alexnet/vgg (default 0.1 is tuned for ResNet).
- **mnist**: `python main.py` (use `CUDA_VISIBLE_DEVICES=N` to pick a GPU).
- **mnist_hogwild**: `python main.py` — spawns multiple processes sharing one model via `torch.multiprocessing`; core loop is in `train.py`.
- **reinforcement_learning**: `python reinforce.py` or `python actor_critic.py`.
- **snli**: `python train.py` (`model.py` defines the encoder, `util.py` has helpers); requires `torchtext` for GloVe/SNLI loading.
- **super_resolution**: `--upscale_factor` is required. Train with `main.py`, then run inference with `python super_resolve.py --input_image <img> --model <model_epoch_N.pth> --output_filename out.png`.
- **time_sequence_prediction**: run `python generate_sine_wave.py` first to create the dataset, then `python train.py`.
- **vae**: `python main.py`.
- **word_language_model**: train with `python main.py --cuda --epochs 6 [--tied]`, then `python generate.py` to sample from the saved model. `data.py` builds the `Corpus`/vocab from `data/penn`; `model.py` defines the RNN.
- **fast_neural_style**: everything runs via `neural_style/neural_style.py` with a `train` or `eval` subcommand, e.g. `python neural_style/neural_style.py eval --content-image <path> --model <path> --output-image <path> --cuda 0` or `... train --dataset <path> --style-image <path> --save-model-dir <path> --epochs 2 --cuda 1`. Pretrained models can be fetched with `download_saved_models.sh`.

## Conventions across examples

- Scripts use `argparse` with `--cuda` (or, in newer scripts, `--no-cuda`/device detection) flags; CUDA is optional, not required.
- Datasets are downloaded/read from a local `data`-style directory that is gitignored (`.gitignore` excludes `data`, `dcgan/data`, `OpenNMT/data`) — never commit downloaded datasets or generated checkpoints/images.
- Each example is meant to be copy-pasteable and educational: keep scripts self-contained and avoid introducing shared/cross-example abstractions or dependencies between example directories.
- When editing an example, update that example's own `README.md` (usage strings, reported perplexities/accuracy, etc.) if behavior or CLI args change.
