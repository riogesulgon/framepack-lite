<h1 align="center">FramePack Lite</h1>

[![Discord](https://img.shields.io/badge/Discord-%235865F2.svg?style=for-the-badge&logo=discord&logoColor=white)](https://discord.gg/MtuM7gFJ3V)[![Patreon](https://img.shields.io/badge/Patreon-F96854?style=for-the-badge&logo=patreon&logoColor=white)](https://www.patreon.com/ColinU)

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/colinurbs/FramePack-Studio)
[![Fork](https://img.shields.io/badge/Fork_of-FP--Studio%2Fframepack--studio-blue?style=flat-square)](https://github.com/FP-Studio/framepack-studio)

> **This is a fork of [FP-Studio/framepack-studio](https://github.com/FP-Studio/framepack-studio) with enhancements for low-VRAM GPUs (6 GB) and quality-of-life improvements.** See [Fork Changes](#fork-changes) below for details.

**FramePack Lite** is a lightweight fork of [FramePack Studio](https://github.com/FP-Studio/framepack-studio) optimized for low-VRAM GPUs (6 GB). It runs the same FramePack video generation models with automatic memory-saving overrides, 4-bit quantization support, and a programmatic automation API.

![screencapture-127-0-0-1-7860-2025-06-12-19_50_37](https://github.com/user-attachments/assets/b86a8422-f4ce-452b-80eb-2ba91945f2ea)
![screencapture-127-0-0-1-7860-2025-06-12-19_52_33](https://github.com/user-attachments/assets/ebfb31ca-85b7-4354-87c6-aaab6d1c77b1)

## Current Features

- **F1, Original and Video Extension Generations**: Run all in a single queue
- **End Frame Control for 'Original' Model**: Provides greater control over generations
- **Upscaling and Post-processing**
- **Timestamped Prompts**: Define different prompts for specific time segments in your video
- **Prompt Blending**: Define the blending time between timestamped prompts
- **LoRA Support**: Works with most (all?) Hunyuan Video LoRAs
- **Queue System**: Process multiple generation jobs without blocking the interface. Import and export queues.
- **Metadata Saving/Import**: Prompt and seed are encoded into the output PNG, all other generation metadata is saved in a JSON file that can be imported later for similar generations.
- **Custom Presets**: Allow quick switching between named groups of parameters. A custom Startup Preset can also be set.
- **I2V and T2V**: Works with or without an input image to allow for more flexibility when working with standard Hunyuan Video LoRAs
- **Latent Image Options**: When using T2V you can generate based on a black, white, green screen, or pure noise image

## Prerequisites

- CUDA-compatible GPU with at least 6GB VRAM (8GB+ recommended for optimal performance; 16GB+ for high-resolution/long videos)
- 16GB System Memory (32GB+ strongly recommended)
- 80GB+ of storage (including ~25GB for each model family: Original and F1)

## Documentation

For information on installation, configuration, and usage, please visit our [documentation site](https://docs.framepackstudio.com/).

## Installation

Please see [this guide](https://docs.framepackstudio.com/docs/get_started/) on the upstream documentation site to get started. Low-VRAM users can also use the provided `start-framepack.sh` script.

## Contributing 

Contributions are welcome! Please open issues and PRs against this repository. For upstream contributions, see the [original project](https://github.com/FP-Studio/framepack-studio).

- Keep Pull Requests Focused: Each PR should address a single issue or feature.
- Discuss Big Changes First: Open an issue before starting a large refactor.


## Fork Changes

This fork extends [FP-Studio/framepack-studio](https://github.com/FP-Studio/framepack-studio) with the following changes:

### 🖥️ Low-VRAM Support (6 GB GPUs, e.g. RTX 2060)

- **Auto-detection of low-VRAM mode** — If ≤ 6 GB free VRAM is detected, the studio automatically applies memory-saving overrides
- **Block cleanup hooks** — Per-transformer-block `torch.cuda.empty_cache()` hooks that free GPU memory between blocks when using `DynamicSwapInstaller`, preventing OOM during the first forward pass on 6 GB cards
- **Auto-capped resolution** — On low-VRAM GPUs, resolution is capped at 384×384 and `latent_window_size` at 6 to prevent attention-layer OOM (the math SDP backend is O(n²) in memory)
- **Conservative defaults** — Low-VRAM mode sets `gpu_memory_preservation=1.0 GB`, `vae_batch_size=4`, and enables MagCache with aggressive settings (`threshold=0.05`, `max_skips=3`, `retention=0.15`)
- **Start/stop scripts** — `start-framepack.sh` (with `PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True`) and `stop-framepack.sh`

### 🔧 BitsAndBytes 4-bit Quantization Helper

- Added `diffusers_helper/quantize.py` — a drop-in helper that replaces all `nn.Linear` layers with `bnb.nn.Linear4bit` (NF4 quantization), reducing model weight memory from ~12 GB (bf16) to ~3 GB. Shares original weight storage to avoid doubling CPU RAM. Ready to integrate into the generator `load_model()` methods.

### 🤖 Automation API

- **`automate.py`** — A CLI client for programmatically submitting generation jobs to a running FramePack Lite instance via the Gradio API. Supports image-to-video, video-to-video, all model types, LoRAs, timestamped prompts, and config-file presets.
- **`modules/automate_config.py`** — Parameter metadata and endpoint matching for the automation client.

### 🛠️ Utilities & Tooling

- **`download-model.sh`** / **`download_missing_model.py`** — Robust model downloaders with resume support, retry logic, and progress logging for the FramePackI2V_HY model
- **`scripts/clean_hf_blob_cache.py`** — Cleans orphaned blob files from the HuggingFace model cache to reclaim disk space
- **`scripts/check_git_sensitive_info.sh`** — A 10-category git history audit script that scans for leaked API keys, tokens, passwords, private keys, PII, and other sensitive information

### 📦 Other Improvements

- **Toolbox graceful degradation** — The toolbox module is now optional; if it fails to import, the studio continues without post-processing/upscaling rather than crashing
- **LoRA weight dict** — The LoRA system now accepts a `{name: weight}` dict instead of positional args, making multi-LoRA control more robust
- **Default resolution lowered to 480×480** — Better default for low-VRAM hardware
- **MagCache support in transformer** — Integration hooks for [MagCache](https://github.com/Zehong-Ma/MagCache) (magnitude-aware caching) to skip redundant diffusion steps

---

## Credits

Many thanks to [Lvmin Zhang](https://github.com/lllyasviel) for the absolutely amazing work on the original [FramePack](https://github.com/lllyasviel/FramePack) code!

Thanks to [Rickard Edén](https://github.com/neph1) for the LoRA code and their general contributions to this growing FramePack scene!

Thanks to [Zehong Ma](https://github.com/Zehong-Ma) for [MagCache](https://github.com/Zehong-Ma/MagCache): Fast Video Generation with Magnitude-Aware Cache!

Thanks to everyone who has joined the Discord, reported a bug, sumbitted a PR, or helped with testing!

    @article{zhang2025framepack,
        title={Packing Input Frame Contexts in Next-Frame Prediction Models for Video Generation},
        author={Lvmin Zhang and Maneesh Agrawala},
        journal={Arxiv},
        year={2025}
    }

    @misc{zhang2025packinginputframecontext,
        title={Packing Input Frame Context in Next-Frame Prediction Models for Video Generation},
        author={Lvmin Zhang and Maneesh Agrawala},
        year={2025},
        eprint={2504.12626},
        archivePrefix={arXiv},
        primaryClass={cs.CV},
        url={https://arxiv.org/abs/2504.12626}
    }
