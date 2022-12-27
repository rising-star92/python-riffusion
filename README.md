# Riffusion

Riffusion is a library for real-time music and audio generation with stable diffusion.

Read about it at https://www.riffusion.com/about and try it at https://www.riffusion.com/.

This repository contains the core riffusion image and audio processing code and supporting apps,
including:

 * diffusion pipeline that performs prompt interpolation combined with image conditioning
 * package for (approximately) converting between spectrogram images and audio clips
 * interactive playground using streamlit
 * command-line tool for common tasks
 * flask server to provide model inference via API
 * various third party integrations
 * test suite

Related repositories:
* Web app: https://github.com/riffusion/riffusion-app
* Model checkpoint: https://huggingface.co/riffusion/riffusion-model-v1

## Citation

If you build on this work, please cite it as follows:

```
@article{Forsgren_Martiros_2022,
  author = {Forsgren, Seth* and Martiros, Hayk*},
  title = {{Riffusion - Stable diffusion for real-time music generation}},
  url = {https://riffusion.com/about},
  year = {2022}
}
```

## Install

Tested with Python 3.9 + 3.10 and diffusers 0.9.0.

To run this model in real time, you need a GPU that can run stable diffusion with approximately 50
steps in under five seconds. A 3090 or A10G will do it.

Install in a virtual Python environment:
```
conda create --name riffusion python=3.9
conda activate riffusion
python -m pip install -r requirements.txt
```

If torchaudio has no audio backend, see
[this issue](https://github.com/riffusion/riffusion/issues/12).

You can open and save WAV files with pure python. For opening and saving non-wav files – like mp3 –
you'll need to install ffmpeg with `suod apt-get install ffmpeg` or `brew install ffmpeg`.

Guides:
* [Windows Simple Instructions](https://www.reddit.com/r/riffusion/comments/zrubc9/installation_guide_for_riffusion_app_inference/)

## Backends

#### CUDA
`cuda` is the recommended and most performant backend.

To use with CUDA, make sure you have torch and torchaudio installed with CUDA support. See the
[install guide](https://pytorch.org/get-started/locally/) or
[stable wheels](https://download.pytorch.org/whl/torch_stable.html). Check with:

```python3
import torch
torch.cuda.is_available()
```

Also see [this issue](https://github.com/riffusion/riffusion/issues/3) for help.

#### CPU
`cpu` works but is quite slow.

#### MPS
The `mps` backend on Apple Silicon is supported for inference but some operations fall back to CPU,
particularly for audio processing. You may need to set
PYTORCH_ENABLE_MPS_FALLBACK=1.

In addition, this backend is not deterministic.

## Command-line interface

Riffusion comes with a command line interface for performing common tasks.

See available commands:
```
python -m riffusion-cli -h
```

Get help for a specific command:
```
python -m riffusion.cli image-to-audio -h
```

Execute:
```
python -m riffusion.cli image-to-audio --image spectrogram_image.png --audio clip.wav
```

## Streamlit playground

Riffusion also has a streamlit app for interactive use and exploration.
This app is called the Riffusion Playground.

Run with:
```
python -m streamlit run riffusion/streamlit/playground.py --browser.serverAddress 127.0.0.1 --bro
wser.serverPort 8501
```

And access at http://127.0.0.1:8501/

## Run the model server

Riffusion can be run as a flask server that provides inference via API. Run with:

```
python -m riffusion.server --host 127.0.0.1 --port 3013
```

You can specify `--checkpoint` with your own directory or huggingface ID in diffusers format.

Use the `--device` argument to specify the torch device to use.

The model endpoint is now available at `http://127.0.0.1:3013/run_inference` via POST request.

Example input (see [InferenceInput](https://github.com/hmartiro/riffusion-inference/blob/main/riffusion/datatypes.py#L28) for the API):
```
{
  "alpha": 0.75,
  "num_inference_steps": 50,
  "seed_image_id": "og_beat",

  "start": {
    "prompt": "church bells on sunday",
    "seed": 42,
    "denoising": 0.75,
    "guidance": 7.0
  },

  "end": {
    "prompt": "jazz with piano",
    "seed": 123,
    "denoising": 0.75,
    "guidance": 7.0
  }
}
```

Example output (see [InferenceOutput](https://github.com/hmartiro/riffusion-inference/blob/main/riffusion/datatypes.py#L54) for the API):
```
{
  "image": "< base64 encoded JPEG image >",
  "audio": "< base64 encoded MP3 clip >"
}
```

## Test
Tests live in the `test/` directory and are implemented with `unittest`.

To run all tests:
```
python -m unittest test/*_test.py
```

To run a single test:
```
python -m unittest test.audio_to_image_test
```

To preserve temporary outputs for debugging, set `RIFFUSION_TEST_DEBUG`:
```
RIFFUSION_TEST_DEBUG=1 python -m unittest test.audio_to_image_test
```

To run a single test case:
```
python -m unittest test.audio_to_image_test -k AudioToImageTest.test_stereo
```

To run tests using a specific torch device, set `RIFFUSION_TEST_DEVICE`. Tests should pass with
`cpu`, `cuda`, and `mps` backends.

## Development
Install additional packages for dev with `pip install -r dev_requirements.txt`.

* Linter: `ruff`
* Formatter: `black`
* Type checker: `mypy`

These are configured in `pyproject.toml`.

The results of `mypy .`, `black .`, and `ruff .` *must* be clean to accept a PR.

CI is run through GitHub Actions from `.github/workflows/ci.yml`.

Contributions are welcome through opening pull requests.
