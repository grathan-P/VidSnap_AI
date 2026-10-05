# VidSnapAI

VidSnapAI is a Flask-based web application for turning a collection of images and a
written narration into a vertical, social-media-ready video reel. It combines
ElevenLabs text-to-speech with FFmpeg so users can upload images, provide narration,
and browse the generated reels from a simple web interface.

## What it does

- Provides a home page with an overview of the application.
- Accepts multiple image uploads for a new reel.
- Stores the narration text with the uploaded image sequence.
- Generates an MP3 voice-over using the ElevenLabs API.
- Stitches the uploaded images and narration into a 1080x1920 MP4 reel with FFmpeg.
- Displays completed reels in a browser-based gallery.

## Tech stack

- **Python** — application and processing scripts
- **Flask** — web server and HTML template rendering
- **Jinja2** — server-rendered templates through Flask
- **ElevenLabs API** — text-to-speech narration
- **FFmpeg** — image sequencing, video encoding, and audio muxing
- **HTML/CSS/JavaScript** — upload form, navigation, and gallery UI
- **Bootstrap 5 and Font Awesome** — UI layout and icons loaded from CDNs

## How the application works

1. Open **Create Reel** and select one or more `.png`, `.jpg`, or `.jpeg` files.
2. Enter the narration text in the text area and submit the form.
3. The Flask app creates a unique folder under `user_uploads/`, saves the images,
   and writes `description.txt` and `input.txt`.
4. `generate_process.py` watches `user_uploads/` for new folders.
5. The worker sends the narration to ElevenLabs and saves `audio.mp3`.
6. FFmpeg converts the image sequence and audio into a vertical MP4 under
   `static/reels/`.
7. Open **Gallery** to play the generated reel.

## Requirements

- Python 3.10 or newer
- FFmpeg installed and available on your `PATH`
- An ElevenLabs API key with access to text-to-speech

The project currently does not include a `requirements.txt` file. Install the Python
dependencies with:

```bash
pip install Flask python-dotenv elevenlabs
```

## Configuration

Create a `.env` file in the project root and add your ElevenLabs key:

```env
ELEVENLABS_API_KEY=your_elevenlabs_api_key
```

Do not commit `.env` or expose the API key publicly.

## Running locally

Run both processes from the project root in separate terminals.

### 1. Start the Flask application

```bash
python main.py
```

The application runs in Flask debug mode and is available at
`http://127.0.0.1:5000`.

### 2. Start the reel processor

```bash
python generate_process.py
```

The processor checks for new upload folders every five seconds. Leave it running
while creating reels. Generated files are recorded in `done.txt` so completed
folders are not processed again.

## Project structure

```text
vidsnap_ai/
├── main.py                 # Flask routes and upload handling
├── generate_process.py     # Background image/audio-to-reel worker
├── text_to_audio.py        # ElevenLabs text-to-speech integration
├── templates/              # Jinja2 HTML templates
├── static/
│   ├── css/                # Application stylesheets
│   └── reels/              # Generated MP4 reels
├── user_uploads/           # Per-request uploaded files and processing inputs
├── done.txt                # Processed upload folder IDs
└── .env                    # Local API configuration; do not commit
```

## Output format

Generated reels use:

- 1080x1920 portrait dimensions
- H.264 video encoding
- AAC audio encoding
- 30 frames per second
- A 1-second duration per source image, or the length of the narration when shorter

Images are scaled to fit the portrait canvas while preserving their aspect ratio.
Unused space is padded with black.

## Current limitations

- The processor is a polling script rather than a managed job queue.
- The Flask development server is not intended for production deployment.
- Upload size limits, authentication, and user-specific access controls are not
  currently configured.
- The app accepts image files even though some page copy refers to video uploads.
- Existing generated assets and uploaded files can grow without automatic cleanup.
- `requirements.txt`, automated tests, and production deployment configuration are
  not currently included.

