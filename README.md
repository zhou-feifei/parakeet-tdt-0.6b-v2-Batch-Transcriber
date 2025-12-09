# parakeet-tdt-0.6b-v2-Batch-Transcriber

Parakeet Batch Transcriber is a powerful command-line tool designed for efficient and high-quality audio transcription. It leverages nvidia/parakeet-tdt-0.6b-v2 model to generate accurate, word-level timestamped transcriptions and formats them into perfectly segmented SRT subtitle files.

The script is heavily optimized for handling very long audio files (e.g., podcasts, lectures, meetings) by intelligently pre-splitting audio at silent points to prevent memory overflows and ensure reliability.

## Key Features

-   **Batch Processing**: Transcribe all supported audio/video files within a directory in a single run.
-   **High-Quality Segmentation**: Uses a sophisticated strategy that prioritizes the model's native segment splits, then uses precise word-level timestamps to intelligently divide long lines based on punctuation (sentences, clauses) for optimal readability.
-   **Very Long Audio Handling**: For audio files exceeding 60 minutes, the tool automatically pre-processes and splits the file at silent intervals before transcription, seamlessly merging the results afterward. This allows for the transcription of multi-hour audio files without memory issues.
-   **CUDA Acceleration**: Fully utilizes NVIDIA GPUs for significantly faster processing speeds.
-   **Duplicate Removal**: Automatically detects and removes redundant subtitle segments where both the text and timestamps are identical, cleaning up common ASR model artifacts and improving the final output quality.
-   **Wide Format Support**: Handles a large variety of common audio and video formats thanks to its FFmpeg backend (e.g., `.mp3`, `.wav`, `.m4a`, `.mp4`, `.mkv`, `.mov`).
-   **Customizable**: Easily adjust subtitle parameters like maximum segment duration and word count to fit your specific needs.
-   **Smart & Resilient**: Skips already processed files, retries with robust methods, and provides detailed logging.

## Transcription Performance

Processing speed is measured by the ratio of audio duration to transcription time (e.g., a value of 20 means a 20-minute audio file is transcribed in 1 minute). The performance heavily depends on the GPU.

| GPU              | Real-Time Speed Factor (Approx.) |
| ---------------- | -------------------------------- |
| NVIDIA RTX 4060 Ti | 16x ~ 29x                        |
| NVIDIA B200      | ~212x                            |
| NVIDIA L40S      | ~199x                            |
| NVIDIA H200      | ~195x                            |
| NVIDIA H100      | ~185x                            |
| NVIDIA A100      | ~175x                            |

*Note: Performance may vary based on system configuration, driver versions, and audio complexity.*

## Installation

### 1. Prerequisites
-   Python 3.8+
-   An NVIDIA GPU with CUDA (for GPU acceleration)
-   FFmpeg

### 2. Install FFmpeg

FFmpeg is required for audio file processing.

-   **Windows**: Download the binaries from the [FFmpeg website](https://ffmpeg.org/download.html) and add the `bin` directory to your system's PATH.
-   **macOS**: Install using Homebrew: `brew install ffmpeg`
-   **Linux (Ubuntu/Debian)**: Install using apt: `sudo apt update && sudo apt install ffmpeg`

### 3. Install PyTorch

Install PyTorch with CUDA support. The command below is for CUDA 12.9. Visit the [PyTorch website](https://pytorch.org/get-started/locally/) to get the correct command for your specific CUDA version.

```bash
pip install torch torchvision torchaudio   --index-url https://download.pytorch.org/whl/cu129
````

### 4\. Install Required Python Libraries

Clone or download this repository, navigate to the directory, and install the necessary Python packages.

```bash
pip install nemo_toolkit[asr]==2.4.0 pydub
```


## Usage

The script is run from the command line, pointing to a directory of audio files.

### Basic Usage

To transcribe all supported audio files in a specific directory:

```bash
python transcribe_batch_english.py /path/to/your/audio_files
```

An `.srt` file will be created alongside each audio file.

### Advanced Usage

**Specify file extensions and use CUDA:**

```bash
python transcribe_batch_english.py ~/my_videos --extensions .mp4 .mkv --device cuda
```

**Force re-transcription of all files (overwrite existing SRTs):**

```bash
python transcribe_batch_english.py . --no-skip
```

**Adjust subtitle splitting rules for longer, more conversational segments:**

```bash
python transcribe_batch_english.py ./podcasts --max-segment 10 --max-words 40
```

### Command-Line Arguments

```
usage: transcribe_batch_english.py [-h] [-d {cuda,cpu,auto}] [-e EXTENSIONS [EXTENSIONS ...]] [--no-skip] [-m MODEL]
                                   [--max-segment MAX_SEGMENT] [--max-words MAX_WORDS] [--debug]
                                   directory

Batch transcribe audio files to SRT subtitles (with smart splitting for long audio).

positional arguments:
  directory             Path to the directory containing audio files.

options:
  -h, --help            show this help message and exit
  -d, --device {cuda,cpu,auto}
                        Inference device (default: auto).
  -e, --extensions EXTENSIONS [EXTENSIONS ...]
                        File extensions to process (default: all supported formats).
  --no-skip             Overwrite and re-transcribe if subtitle files already exist.
  -m, --model MODEL     Name or path of the model to use (default: nvidia/parakeet-tdt-0.6b-v2).
  --max-segment MAX_SEGMENT
                        Maximum subtitle segment duration in seconds (default: 7).
  --max-words MAX_WORDS
                        Maximum words per segment, used as a fallback (default: 30).
  --debug               Enable detailed debug output.
```

## How It Works: The Optimization Strategy

This tool is designed to create the most natural and readable subtitles possible by using a multi-layered approach:

1.  **Model-Native Segments**: It first obtains segment-level timestamps directly from the Parakeet model. These are generally high-quality and aligned with natural pauses in speech.
2.  **Word-Level Precision Splitting**: For any segment from the model that is too long (e.g., \>7 seconds), the script uses the highly precise word-level timestamps to find the best possible split point.
3.  **Punctuation Priority**: The splitting logic prioritizes natural boundaries in the following order to avoid awkward breaks:
      - End of a sentence (`.`, `!`, `?`)
      - End of a clause (`;`, `:`)
      - Soft separators (`,`)
4.  **Fallback**: If no punctuation is available for splitting, it falls back to a simple word count to ensure the duration constraints are met.
5.  **Pre-computation Splitting**: For extremely long audio files (\>60 minutes), the script first analyzes the entire file to find long periods of silence. It splits the audio into large, manageable chunks at these silent points, transcribes each chunk individually, and then intelligently merges the final results with adjusted timestamps.

## Program screenshots
![pic](./pic.png)

## License

This project is licensed under the **MIT License**.

Copyright (c) 2025 zhou-feifei

You are free to **use, copy, modify, merge, publish, distribute, sublicense, and/or sell** copies of the software. The only condition is that the original copyright and permission notice must be included. The software is provided **"AS IS", without any warranty**.
