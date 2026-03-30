# Content-to-Video Tool for Debian proot-distro on Termux

This is a Python-based tool designed to scrape content from various internet sources, process it into a short, engaging script, and generate a vertical video suitable for platforms like TikTok and YouTube Shorts. The tool is specifically optimized to run within a Debian proot-distro environment on Android (Termux).

## Features

- **Content Scraping**: Gathers information from RSS feeds (BBC, CNN), Reddit, and Wikipedia based on a user-defined topic.
- **Script Generation**: Processes scraped content to create a concise, conversational script (30-60 seconds).
- **Vertical Video Production**: Generates 9:16 aspect ratio videos with:
    - Dynamic visuals from free stock media (Pexels/Pixabay APIs).
    - Text-to-speech voiceover using gTTS.
    - Automatic captions synchronized with the audio.
    - Optional royalty-free background music.
- **Local Execution**: Runs entirely on the device, utilizing only free, open-source software and APIs.

## Environment Assumptions

- You have [Termux](https://termux.com/) installed on your Android device.
- You have set up a Debian proot-distro environment within Termux (e.g., via `pkg install proot-distro` and `proot-distro install debian`).
- The tool will be executed inside this Debian environment.

## Installation Guide

Follow these steps to set up and run the Content-to-Video tool in your Debian proot-distro environment.

### 1. Enter Debian proot-distro

First, open Termux and enter your Debian environment:

```bash
proot-distro login debian
```

### 2. Update Packages and Install System Dependencies

Once inside Debian, update your package list and install the necessary system dependencies. `ffmpeg` is crucial for video processing with `moviepy`, and `python3-venv` is recommended for managing Python environments.

```bash
apt update && apt upgrade -y
apt install -y python3 python3-pip python3-venv ffmpeg imagemagick git
```

### 3. Clone the Repository

Clone this repository to your desired location. For example, in your home directory:

```bash
git clone https://github.com/your-username/content-to-video.git # Replace with actual repo URL
cd content-to-video
```

### 4. Set up Python Virtual Environment

It's highly recommended to use a Python virtual environment to manage dependencies.

```bash
python3 -m venv venv
source venv/bin/activate
```

### 5. Install Python Libraries

Install the required Python packages using `pip`:

```bash
pip install -r requirements.txt
```

### 6. Obtain and Set Up API Keys

This tool relies on free APIs for content scraping and media fetching. You need to obtain API keys for Pexels or Pixabay (for stock media) and optionally for Reddit (for Reddit content).

1.  **Pexels (for stock photos/videos)**:
    - Go to [Pexels API](https://www.pexels.com/api/)
    - Sign up for a free account and generate your API key.
2.  **Pixabay (for stock photos/videos)**:
    - Go to [Pixabay API](https://pixabay.com/api/docs/)
    - Sign up for a free account and generate your API key.
3.  **Reddit (optional, for Reddit content)**:
    - Go to [Reddit App Preferences](https://www.reddit.com/prefs/apps)
    - Click "create another app..."
    - Choose "script" as the app type.
    - Fill in a name (e.g., `ContentToVideoBot`), description, and set `http://localhost:8080` as the redirect URI.
    - Note down your `client_id` (under your app name) and `client_secret`.

Once you have your API keys, create a file named `.env` in the root directory of the `content-to-video` project (where `main.py` is located) and add your keys as follows. You can copy `env.example` and rename it.

```ini
# .env

# Reddit API Credentials (Optional but recommended)
REDDIT_CLIENT_ID=your_reddit_client_id
REDDIT_CLIENT_SECRET=your_reddit_client_secret
REDDIT_USER_AGENT=ContentToVideoBot/0.1

# Stock Media API Keys (At least one required for stock media)
PEXELS_API_KEY=your_pexels_api_key
PIXABAY_API_KEY=your_pixabay_api_key

# Optional: Output directory (Default is ./output)
# OUTPUT_DIR=/home/ubuntu/videos/
```

**Important**: You need at least one of `PEXELS_API_KEY` or `PIXABAY_API_KEY` for the tool to fetch stock media. If both are missing, the tool will generate videos with a plain background.

## Usage

After installation and API key setup, you can run the tool from your Debian proot-distro environment.

```bash
source venv/bin/activate # Activate your virtual environment if not already active
python main.py --topic "quantum computing" --output ~/videos/my_video.mp4
```

### Command-line Arguments

-   `--topic <TOPIC>` (Required): The topic for which you want to generate a video (e.g., "artificial intelligence", "climate change").
-   `--output <PATH>` (Optional): The full path where the final video will be saved. Defaults to `./output/final_video.mp4`.
-   `--no-music` (Optional): Use this flag to disable background music in the video.
-   `--dry-run` (Optional): Use this flag to perform content scraping and script generation without creating the video. The script will be saved to `./output/script.txt`.

### Example

To generate a video about "space exploration" without background music and save it to your Termux home directory (which can be accessed from Android's internal storage via `/data/data/com.termux/files/home`):

```bash
python main.py --topic "space exploration" --output /home/ubuntu/videos/space_exploration.mp4 --no-music
```

To save the video to a directory accessible by Android's gallery (e.g., `/sdcard/DCIM/` or `/sdcard/Movies/`), you might need to create a symlink or manually move the file after generation. For example, to create a `videos` folder in your Termux home and symlink it to `/sdcard/Movies`:

```bash
mkdir -p ~/videos
ln -s /sdcard/Movies ~/videos/sdcard_movies # Create symlink once
python main.py --topic "space exploration" --output ~/videos/sdcard_movies/space_exploration.mp4
```

## Troubleshooting

-   **`ffmpeg` not found**: Ensure `ffmpeg` is installed in your Debian environment (`sudo apt install -y ffmpeg`).
-   **Network errors**: Check your internet connection. If you're facing issues with API calls, ensure your API keys are correctly set in the `.env` file.
-   **`moviepy` issues**: `moviepy` can sometimes be finicky with codecs. Ensure `ffmpeg` is properly installed. If you encounter errors, try updating `moviepy` (`pip install --upgrade moviepy`).
-   **`gTTS` errors**: `gTTS` requires an active internet connection to Google's TTS service. Check your network.
-   **No content found**: Try a broader topic. Some very niche topics might not have enough publicly available content on the configured sources.
-   **Video resolution/performance**: The tool is designed for mobile hardware. If videos are too slow to generate or too large, consider reducing the `VIDEO_WIDTH` and `VIDEO_HEIGHT` in `utils/config.py` (though 1080x1920 is standard for vertical video).

## Project Structure

```
content-to-video/
├── main.py
├── requirements.txt
├── .env.example
├── README.md
├── scraper/
│   ├── __init__.py
│   ├── news.py
│   ├── reddit.py
│   ├── wikipedia.py
│   └── cache.py
├── processor/
│   ├── __init__.py
│   └── summarizer.py
├── video/
│   ├── __init__.py
│   ├── visuals.py
│   ├── audio.py
│   ├── captions.py
│   └── composer.py
└── utils/
    ├── __init__.py
    ├── config.py
    └── file_utils.py
```

## License

This project is open-source and available under the MIT License. (You may choose a different license if preferred.)
