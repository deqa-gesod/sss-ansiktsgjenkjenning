# Ansiktsgjenkjenning og velkomstsystem

A face-recognition welcome system that runs on a Raspberry Pi. The camera watches the scene, recognises people it already knows from a folder of photos, draws a box with their name on the live video, and greets them out loud in Norwegian: "velkommen kjære kunde [navn]".

It is a working prototype. The idea was a small reception or shop setup where a regular customer walks in and gets greeted by name.

## What it does

1. The Pi camera records video and writes frames to shared memory.
2. Each frame is compared against the known faces in the `images/` folder.
3. When a face matches, the program draws a red box around it and writes the person's name above it.
4. The first time it sees a known person, it speaks a Norwegian greeting through the speaker. It won't repeat the greeting for the same person back to back.

Unknown faces are still boxed, just labelled "Unknown" and not greeted.

## How it is built

- **Python 3.11**
- **face_recognition** for the actual recognition (face encodings and matching)
- **OpenCV** for reading frames, drawing boxes and showing the window
- **gTTS** to turn the greeting text into Norwegian speech
- **pydub** to play the generated audio
- **libcamera** on the Raspberry Pi to capture from the camera

Recognition logic lives in its own class, `FaceRecognizer` in `face_recognizer.py`, so the main script stays focused on the camera loop and display. To keep things fast enough on a Pi, frames are shrunk to a quarter size before matching, then the face coordinates are scaled back up for drawing.

## Requirements

This needs a Raspberry Pi with a camera. The capture step uses `libcamera-vid`, so it won't run as-is on a regular laptop without swapping out the camera code.

## Setup

```bash
pip install -r requirements.txt
```

Put a photo of each known person in the `images/` folder. The file name becomes the displayed name, so `ola.jpg` shows up as "Ola". One clear, front-facing face per photo works best.

## Running it

```bash
python fcrecog-sss.py
```

Press `q` to quit. Generated audio files land in `sounds/`, which is created automatically.

## Project structure

```
├── fcrecog-sss.py     # Main program: camera loop, drawing, greeting
├── face_recognizer.py # FaceRecognizer class (loading + matching faces)
├── images/            # Photos of known people
├── sounds/            # Generated audio (created at runtime)
└── requirements.txt
```

## Limitations and what I'd improve

- It is tied to the Raspberry Pi camera through `libcamera-vid`. A small abstraction over the capture source would let it run on a normal webcam too.
- Matching uses the default face_recognition tolerance with a nearest-encoding check. There's no confidence threshold, so a borderline match can still be labelled with a name. Adding a distance cutoff would cut down false positives.
- The greeting calls gTTS live, which needs an internet connection and adds a short delay. Caching one audio file per person, or switching to offline speech, would make it snappier.
- Recognition runs on every frame. Doing it every few frames would lighten the load on the Pi.
