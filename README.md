# Signa

**Real-time ASL fingerspelling recognition in your browser.** Signa uses your camera to recognize static handshapes and build a text transcript. Hand tracking and recognition run locally in the browser; camera frames are not uploaded.

> **Language note:** This version recognizes the **American Sign Language (ASL) fingerspelling alphabet**. It does not currently recognize Indian Sign Language (ISL), full signed sentences, or general sign vocabulary.

## Features

- Live hand tracking with MediaPipe and a TensorFlow.js landmark classifier.
- Letter predictions with confidence display, temporal smoothing, and a hold-to-commit indicator.
- Transcript controls for spaces, delete, copy, and text-to-speech.
- Train a personal model in the browser using webcam samples, or train offline with landmark CSV data.
- Store samples and personal models locally in the browser.
- Camera selection, mirrored view, adjustable confidence and hold time, and responsive dark interface.

## Run locally

Camera access requires `localhost` or HTTPS. On Windows, open PowerShell in the project folder and run:

```powershell
powershell -ExecutionPolicy Bypass -File .\serve.ps1
```

Then open [http://localhost:8080](http://localhost:8080) and allow camera access. Keep the terminal window open while using the app.

Alternatively, if Python is installed:

```powershell
python -m http.server 8080
```

Open [http://localhost:8080](http://localhost:8080).

## Use the app

1. Open **Signa** in a modern browser and select **Start camera**.
2. Hold one ASL letter handshape in view until the progress ring completes.
3. Relax your hand briefly before signing the same letter again. Use the **Space** and **Delete** controls to edit the transcript.
4. Open **Train** to collect your own examples and train a model saved in this browser.

The app loads TensorFlow.js and MediaPipe code from jsDelivr and the hand-landmarker asset from Google on the first visit. The camera stream and live hand landmarks are processed on-device.

## Train with your own examples

For the best match to your camera, lighting, hand, and signing style, use the **Train** tab:

1. Select a label and start the camera.
2. Record examples while varying hand distance, position, and angle. Aim for at least 100 varied samples per label.
3. Train the model and check its validation results.
4. Export the model JSON to keep a backup or bundle it at `model/weights.json` for deployment.

Samples and trained models stay in the browser's IndexedDB unless you explicitly export them. They are not shared between browsers or devices automatically.

## Train from the ASL Alphabet dataset

The repository does **not** include a dataset or a pre-trained model. The training script can use the [ASL Alphabet dataset on Kaggle](https://www.kaggle.com/datasets/grassknoted/asl-alphabet), uploaded by Akash under the username `grassknoted`. Kaggle lists 87,000 images across A–Z, SPACE, DELETE, and NOTHING, and a GPL-2 license. The original image collection process is not described in this project, and this repository does not claim the dataset was used to produce a bundled model.

Download the dataset from Kaggle and extract it so each label has its own folder. Then, from the project root, install the offline training requirements and extract hand landmarks:

```powershell
python -m pip install -r training\requirements.txt
python training\extract_landmarks.py path\to\asl_alphabet_train data\asl-landmarks.csv --per-class 600
python training\train.py data\asl-landmarks.csv --out model\weights.json
```

Replace `path\to\asl_alphabet_train` with the directory containing the label folders (A, B, C, …, Z, `space`, `del`, and `nothing`). The extractor skips `nothing`; J and Z are excluded by default because they are motion signs. The training script reports held-out accuracy and per-class results, and writes a confusion matrix when Matplotlib is available.

The held-out split is drawn from the same downloaded image collection, so its accuracy is not a guarantee of performance with new signers or webcam conditions. Add varied webcam samples and test with people who were not part of training. Follow the dataset's license and attribution requirements if you use its images or publish a model trained from them.

## How it works

```text
Camera frame → MediaPipe hand landmarks → normalized 63-value feature vector
             → starter rules or MLP classifier → temporal smoothing → transcript
```

The browser and Python training pipeline share the same landmark normalization. A bundled `model/weights.json` is optional; without one or a model trained in the current browser, Signa uses its built-in starter rules.

## Current limitations

- Recognizes static ASL fingerspelling handshapes, not continuous signing, sentence grammar, or general vocabulary.
- J and Z require motion over time and are not included in the static-pose classifier.
- Similar handshapes, occlusion, camera angle, lighting, and differences between training data and live webcam video can affect predictions.
- A model trained from a single image dataset may not generalize to different signers. Validate it with diverse, separate webcam recordings.

## Project structure

```text
index.html                 App shell
css/styles.css             Responsive dark interface
js/app.js                  Camera, UI, and application flow
js/hand.js                 MediaPipe hand tracking
js/features.js             Landmark normalization and augmentation
js/heuristic.js            Starter rule-based classifier
js/classifier.js           TensorFlow.js MLP classifier
js/committer.js            Prediction smoothing and letter commit logic
js/dataset.js              Browser sample recording and CSV import/export
js/storage.js              Browser storage and preferences
model/                     Optional bundled model weights
training/                  Python landmark extraction and model training
tests/                     Browser self-test
```

## Credits

Signa is based on [Scarlet-Thunder/sign-language-translator](https://github.com/Scarlet-Thunder/sign-language-translator). The example offline dataset is [ASL Alphabet by grassknoted on Kaggle](https://www.kaggle.com/datasets/grassknoted/asl-alphabet); it is separate from the application source and is not bundled here.
