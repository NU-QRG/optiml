# Optiml: Fast Large Language Model Serving with a Consumer-grade GPU

## TL;DR
Optiml is a CPU/GPU LLM inference engine leveraging **activation locality** for your device.

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](https://opensource.org/licenses/MIT)

[Project Kanban](https://github.com/orgs/KAIST-KEAI/projects/2/views/2)

## Latest News 🔥
- [2024/3/14] We supported ProSparse Llama 2 ([7B](https://huggingface.co/SparseLLM/prosparse-llama-2-7b)/[13B](https://huggingface.co/SparseLLM/prosparse-llama-2-13b)), ReLU models with ~90% sparsity, matching original Llama 2's performance (Thanks THUNLP & ModelBest)!
- [2024/1/11] We supported Windows with GPU inference!
- [2023/12/24] We released an online [gradio demo](https://Optiml-gradio.vercel.app/) for Falcon(ReLU)-40B-FP16!
- [2023/12/19] We officially released Optiml!
## Demo 🔥

https://github.com/KAIST-KEAI/Optiml/assets/34213478/fe441a42-5fce-448b-a3e5-ea4abb43ba23

Optiml v.s. llama.cpp on a single RTX 4090(24G) running Falcon(ReLU)-40B-FP16 with a 11x speedup!

<sub>Both Optiml and llama.cpp were running on the same hardware and fully utilized VRAM on RTX 4090.</sub>

> [!NOTE]
> **Live Demo Online⚡️**
>
> Try out our [Gradio server](https://Optiml-gradio.vercel.app/) hosting Falcon(ReLU)-40B-FP16 on a RTX 4090!
>
> <sub>Experimental and without warranties 🚧</sub>

## Abstract

We introduce Optiml, a high-speed Large Language Model (LLM) inference engine on a personal computer (PC)
equipped with a single consumer-grade GPU. The key underlying the design of Optiml is exploiting the high **locality**
inherent in LLM inference, characterized by a power-law distribution in neuron activation.

This distribution indicates that a small subset of neurons, termed hot neurons, are consistently activated
across inputs, while the majority, cold neurons, vary based on specific inputs.
Optiml exploits such an insight to design a GPU-CPU hybrid inference engine:
hot-activated neurons are preloaded onto the GPU for fast access, while cold-activated neurons are computed
on the CPU, thus significantly reducing GPU memory demands and CPU-GPU data transfers.
Optiml further integrates adaptive predictors and neuron-aware sparse operators,
optimizing the efficiency of neuron activation and computational sparsity.

Evaluation shows that Optiml attains an average token generation rate of 13.20 tokens/s, with a peak of 29.08 tokens/s, across various LLMs (including OPT-175B) on a single NVIDIA RTX 4090 GPU,
only 18\% lower than that achieved by a top-tier server-grade A100 GPU.
This significantly outperforms llama.cpp by up to 11.69x while retaining model accuracy.

## Features
Optiml is a high-speed and easy-to-use inference engine for deploying LLMs locally.

Optiml is fast with:

- **Locality-centric design**: Utilizes sparse activation and 'hot'/'cold' neuron concept for efficient LLM inference, ensuring high speed with lower resource demands.
- **Hybrid CPU/GPU Utilization**: Seamlessly integrates memory/computation capabilities of CPU and GPU for a balanced workload and faster processing.

Optiml is flexible and easy to use with:

- **Easy Integration**: Compatible with popular [ReLU-sparse models](https://huggingface.co/SparseLLM).
- **Local Deployment Ease**: Designed and deeply optimized for local deployment on consumer-grade hardware, enabling low-latency LLM inference and serving on a single GPU.
- **Backward Compatibility**: While distinct from llama.cpp, you can make use of most of `examples/` the same way as llama.cpp such as server and batched generation. Optiml also supports inference with llama.cpp's model weights for compatibility purposes, but there will be no performance gain.

You can use these models with Optiml today:

- Falcon-40B
- Llama2 family
- ProSparse Llama2 family
- Bamboo-7B

We have tested Optiml on the following platforms:

- x86-64 CPUs with AVX2 instructions, with or without NVIDIA GPUs, under **Linux**.
- x86-64 CPUs with AVX2 instructions, with or without NVIDIA GPUs, under **Windows**.
- Apple M Chips (CPU only) on **macOS**. (As we do not optimize for Mac, the performance improvement is not significant now.)

And new features coming soon:

- Mistral-7B model
- Metal backend for sparse inference on macOS

Please kindly refer to our [Project Kanban](https://github.com/orgs/KAIST-KEAI/projects/2/views/2) for our current focus of development.

## Getting Started

- [Installation](#setup-and-installation)
- [Model Weights](#model-weights)
- [Inference](#inference)

## Setup and Installation

### Pre-requisites

Optiml requires the following dependencies:

- CMake (3.17+)
- Python (3.8+) and pip (19.3+), for converting model weights and automatic FFN offloading

### Get the Code

```bash
git clone https://github.com/KAIST-KEAI/Optiml
cd Optiml
pip install -r requirements.txt # install Python helpers' dependencies
```
### Build
In order to build Optiml you have two different options. These commands are supposed to be run from the root directory of the project.

Using `CMake`(3.17+):
* If you have an NVIDIA GPU:
```bash
cmake -S . -B build -DLLAMA_CUBLAS=ON
cmake --build build --config Release
```
* If you have just CPU:
```bash
cmake -S . -B build
cmake --build build --config Release
```

## Model Weights

Optiml models are stored in a special format called *Optiml GGUF* based on GGUF format, consisting of both LLM weights and predictor weights.

### Download Optiml GGUF via Hugging Face

You can obtain Optiml GGUF weights at `*.Optiml.gguf` as well as profiled model activation statistics for 'hot'-neuron offloading from each Hugging Face repo below.

| Base Model            | Optiml GGUF                                                                                               |
|-----------------------|---------------------------------------------------------------------------------------------------------------|
| LLaMA(ReLU)-2-7B      | [Optiml/ReluLLaMA-7B-Optiml-GGUF](https://huggingface.co/Optiml/ReluLLaMA-7B-Optiml-GGUF)     |
| LLaMA(ReLU)-2-13B     | [Optiml/ReluLLaMA-13B-Optiml-GGUF](https://huggingface.co/Optiml/ReluLLaMA-13B-Optiml-GGUF)   |
| Falcon(ReLU)-40B      | [Optiml/ReluFalcon-40B-Optiml-GGUF](https://huggingface.co/Optiml/ReluFalcon-40B-Optiml-GGUF) |
| LLaMA(ReLU)-2-70B     | [Optiml/ReluLLaMA-70B-Optiml-GGUF](https://huggingface.co/Optiml/ReluLLaMA-70B-Optiml-GGUF)   |
| ProSparse-LLaMA-2-7B  | [Optiml/ProSparse-LLaMA-2-7B-GGUF](https://huggingface.co/Optiml/prosparse-llama-2-7b-gguf)           |
| ProSparse-LLaMA-2-13B | [Optiml/ProSparse-LLaMA-2-13B-GGUF](https://huggingface.co/Optiml/prosparse-llama-2-13b-gguf)         |

We recommend using [`huggingface-cli`](https://huggingface.co/docs/huggingface_hub/guides/cli) to download the whole model repo. For example, the following command will download [Optiml/ReluLLaMA-7B-Optiml-GGUF](https://huggingface.co/Optiml/ReluLLaMA-7B-Optiml-GGUF) into the `./ReluLLaMA-7B` directory.

```shell
huggingface-cli download --resume-download --local-dir ReluLLaMA-7B --local-dir-use-symlinks False Optiml/ReluLLaMA-7B-Optiml-GGUF
```

As such, Optiml can automatically make use of the following directory structure for feature-complete model offloading:
```
.
├── *.Optiml.gguf (Unquantized Optiml model)
├── *.q4.Optiml.gguf (INT4 quantized Optiml model, if available)
├── activation (Profiled activation statistics for fine-grained FFN offloading)
│   ├── activation_x.pt (Profiled activation statistics for layer x)
│   └── ...
├── *.[q4].Optiml.gguf.generated.gpuidx (Generated GPU index at runtime for corresponding model)
```

### Convert from Original Model Weights + Predictor Weights

Hugging Face limits single model weight to 50GiB. For unquantized models >= 40B, you can convert Optiml GGUF from the original model weights and predictor weights obtained from Hugging Face.

| Base Model            | Original Model                                                                            | Predictor |
|-----------------------|-------------------------------------------------------------------------------------------|---------------------|
| LLaMA(ReLU)-2-7B      | [SparseLLM/ReluLLaMA-7B](https://huggingface.co/SparseLLM/ReluLLaMA-7B)                   |  [Optiml/ReluLLaMA-7B-Predictor](https://huggingface.co/Optiml/ReluLLaMA-7B-Predictor)
| LLaMA(ReLU)-2-13B     | [SparseLLM/ReluLLaMA-13B](https://huggingface.co/SparseLLM/ReluLLaMA-13B)                 |  [Optiml/ReluLLaMA-13B-Predictor](https://huggingface.co/Optiml/ReluLLaMA-13B-Predictor)
| Falcon(ReLU)-40B      | [SparseLLM/ReluFalcon-40B](https://huggingface.co/SparseLLM/ReluFalcon-40B)               | [Optiml/ReluFalcon-40B-Predictor](https://huggingface.co/Optiml/ReluFalcon-40B-Predictor)
| LLaMA(ReLU)-2-70B     | [SparseLLM/ReluLLaMA-70B](https://huggingface.co/SparseLLM/ReluLLaMA-70B)                 |  [Optiml/ReluLLaMA-70B-Predictor](https://huggingface.co/Optiml/ReluLLaMA-70B-Predictor)
| ProSparse-LLaMA-2-7B  | [SparseLLM/ProSparse-LLaMA-2-7B](https://huggingface.co/SparseLLM/prosparse-llama-2-7b)   |  [Optiml/ProSparse-LLaMA-2-7B-Predictor](https://huggingface.co/Optiml/prosparse-llama-2-7b-predictor)
| ProSparse-LLaMA-2-13B | [SparseLLM/ProSparse-LLaMA-2-13B](https://huggingface.co/SparseLLM/prosparse-llama-2-13b) |  [Optiml/ProSparse-LLaMA-2-13B-Predictor](https://huggingface.co/Optiml/prosparse-llama-2-13b-predictor)

You can use the following command to convert the original model weights and predictor weights to Optiml GGUF:
```bash
# make sure that you have done `pip install -r requirements.txt`
python convert.py --outfile /PATH/TO/Optiml/GGUF/REPO/MODELNAME.Optiml.gguf /PATH/TO/ORIGINAL/MODEL /PATH/TO/PREDICTOR
# python convert.py --outfile ./ReluLLaMA-70B-Optiml-GGUF/llama-70b-relu.Optiml.gguf ./SparseLLM/ReluLLaMA-70B ./Optiml/ReluLLaMA-70B-Predictor
```
For the same reason, we suggest keeping the same directory structure as Optiml GGUF repos after conversion.

<details>

<summary>Convert Original models into dense GGUF models(compatible with llama.cpp)</summary>

```bash
python convert-dense.py --outfile /PATH/TO/DENSE/GGUF/REPO/MODELNAME.gguf /PATH/TO/ORIGINAL/MODEL
# python convert-dense.py --outfile ./Bamboo-DPO-v0.1-gguf/bamboo-7b-dpo-v0.1.gguf --outtype f16 ./Bamboo-DPO-v0.1
```

Please note that the generated dense GGUF models might not work properly with llama.cpp, as we have altered activation functions (for ReluLLaMA and Prosparse models), or the model architecture (for Bamboo models). The dense GGUF models generated by convert-dense.py can be used for Optiml in dense inference mode, but might not work properly with llama.cpp.

</details>

## Inference

For CPU-only and CPU-GPU hybrid inference with all available VRAM, you can use the following instructions to run Optiml:
```bash
./build/bin/main -m /PATH/TO/MODEL -n $output_token_count -t $thread_num -p $prompt
# e.g.: ./build/bin/main -m ./ReluFalcon-40B-Optiml-GGUF/falcon-40b-relu.q4.Optiml.gguf -n 128 -t 8 -p "Once upon a time"
# For Windows: .\build\bin\Release\main.exe -m .\ReluFalcon-40B-Optiml-GGUF\falcon-40b-relu.q4.Optiml.gguf -n 128 -t 8 -p "Once upon a time"
```

If you want to limit the VRAM usage of GPU:
```bash
./build/bin/main -m /PATH/TO/MODEL -n $output_token_count -t $thread_num -p $prompt --vram-budget $vram_gb
# e.g.: ./build/bin/main -m ./ReluLLaMA-7B-Optiml-GGUF/llama-7b-relu.Optiml.gguf -n 128 -t 8 -p "Once upon a time" --vram-budget 8
# For Windows: .\build\bin\Release\main.exe -m .\ReluLLaMA-7B-Optiml-GGUF\llama-7b-relu.Optiml.gguf -n 128 -t 8 -p "Once upon a time" --vram-budget 8
```
Under CPU-GPU hybrid inference, Optiml will automatically offload all dense activation blocks to GPU, then split FFN and offload to GPU if possible.

## Quantization

Optiml has optimized quantization support for INT4(`Q4_0`) models. You can use the following instructions to quantize Optiml GGUF model:
```bash
./build/bin/quantize /PATH/TO/MODEL /PATH/TO/OUTPUT/QUANTIZED/MODEL Q4_0
# e.g.: ./build/bin/quantize ./ReluFalcon-40B-Optiml-GGUF/falcon-40b-relu.Optiml.gguf ./ReluFalcon-40B-Optiml-GGUF/falcon-40b-relu.q4.Optiml.gguf Q4_0
# For Windows: .\build\bin\Release\quantize.exe .\ReluFalcon-40B-Optiml-GGUF\falcon-40b-relu.Optiml.gguf .\ReluFalcon-40B-Optiml-GGUF\falcon-40b-relu.q4.Optiml.gguf Q4_0
```
Then you can use the quantized model for inference with Optiml with the same instructions as above.

## More Documentation
- [Performance troubleshooting](./docs/token_generation_performance_tips.md)

## Evaluation

We evaluated Optiml vs. llama.cpp on a single RTX 4090(24G) with a series of FP16 ReLU models under inputs of length 64, and the results are shown below. Optiml achieves up to 11x speedup on Falcon 40B and up to 3x speedup on Llama 2 70B.

![github-eval-4090](https://github.com/KAIST-KEAI/Optiml/assets/34213478/d700fa6c-77ba-462f-a2fc-3fd21c898f33)
<sub>The X axis indicates the output length, and the Y axis represents the speedup compared with llama.cpp. The number above each bar indicates the end-to-end generation speed (total prompting + generation time / total tokens generated, in tokens/s).</sub>

We also evaluated Optiml on a single RTX 2080Ti(11G) with INT4 ReLU models under inputs of length 8, and the results are illustrated in the same way as above. Optiml achieves up to 8x speedup on Falcon 40B and up to 3x speedup on Llama 2 70B.

![github-eval-2080ti-q4](https://github.com/KAIST-KEAI/Optiml/assets/34213478/0fc1bfc4-aafc-4e82-a865-bec0143aff1a)

Please refer to our [paper]() for more evaluation details.

## FAQs
1. What if I encountered `CUDA_ERROR_OUT_OF_MEMORY`?
   - You can try to run with `--reset-gpu-index` argument to rebuild the GPU index for this model to avoid any stale cache.
   - Due to our current implementation, model offloading might not be as accurate as expected. You can try with `--vram-budget` with a slightly lower value or `--disable-gpu-index` to disable FFN offloading.

2. Does Optiml support mistral, original llama, Qwen, ...?
   - Now we only support models with ReLU/ReGLU/Squared ReLU activation function. So we do not support these models now. It's worth mentioning that a [paper](https://arxiv.org/pdf/2310.04564.pdf) has demonstrated that using the ReLU/ReGLU activation function has a negligible impact on convergence and performance.

3. Why is there a noticeable downgrade in the performance metrics of our current ReLU model, particularly the 70B model?
   - In contrast to the typical requirement of around 2T tokens for LLM training, our model's fine-tuning was conducted with only 5B tokens. This insufficient retraining has resulted in the model's inability to regain its original performance. We are actively working on updating to a more capable model, so please stay tuned.

4. What if...
   - Issues are welcomed! Please feel free to open an issue and attach your running environment and running parameters. We will try our best to help you.

## TODOs
We will release the code and data in the following order, please stay tuned!

- [x] Release core code of Optiml, supporting Llama-2, Falcon-40B.
- [ ] Support Mistral-7B
- [x] Support Windows
- [ ] Support text-generation-webui
- [ ] Release perplexity evaluation code
- [ ] Support Metal for Mac
- [ ] Release code for OPT models
- [ ] Release predictor training code
- [x] Support online split for FFN network
- [ ] Support Multi-GPU


## Paper and Citation
More technical details can be found in our [paper]().

If you find Optiml useful or relevant to your project and research, please kindly cite our paper:

```bibtex
@misc{song2023Optiml,
      title={Optiml: Fast Large Language Model Serving with a Consumer-grade GPU},
      author={Yechan Hwang},
      year={2023},
      eprint={2312.12456},
      archivePrefix={arXiv},
      primaryClass={cs.LG}
}
```

## Acknowledgement
We are thankful for the easily modifiable operator library [ggml](https://github.com/ggerganov/ggml) and execution runtime provided by [llama.cpp](https://github.com/ggerganov/llama.cpp). We also extend our gratitude to [THUNLP](https://nlp.csai.tsinghua.edu.cn/) for their support of ReLU-based sparse models. We also appreciate the research of [Deja Vu](https://proceedings.mlr.press/v202/liu23am.html), which inspires Optiml.
