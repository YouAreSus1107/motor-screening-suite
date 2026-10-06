# Motor Screening Suite

Six short motor tests that run on an ordinary webcam and microphone. A local web hub launches them, keeps each person's results, and charts them over time.

The tests target the fine motor, eye movement and speech changes that research links to early cognitive decline and Parkinson's disease.

## Status

- This is a research prototype. It is not a diagnostic or screening device.
- Only the finger tapping test has been checked against outside data.
- Every other result band is provisional. The bands come from published papers or from a small number of our own recordings.
- The tremor thresholds were set on two public wrist sensor datasets. That is calibration and not an independent check.
- The walking test is built and unit tested but has not been run live.
- Most runs so far come from a few people on a few cameras.

## The tests

- **Finger tapping.** Tap thumb and index finger together, big and fast or in time with a beep. The result is how even the rhythm is.
- **Spiral.** Trace a spiral in the air with a fingertip. The result is two scores, how close to the line and how much the fingertip shakes.
- **Eye movement.** Look toward a dot, then away from it, then hold still. The result is how often the eyes go the wrong way.
- **Speech.** Say "pa-ta-ka" repeatedly, then hold an "ahh". The results are syllable rhythm and voice steadiness.
- **Tremor.** Rest both hands in the lap, then hold the arms out. The result is tremor frequency and size for each hand. This is a supporting check and not a cognitive marker.
- **Walking, seated part.** Stamp each leg, then stand up from a chair five times. The result is the time for five stands.

## Privacy

- Everything runs on your own machine.
- Audio and video are never saved.
- Only the measurements and the traces derived from them are kept, in the `results` folder.

## Validation

The tapping test was run unchanged on two public datasets.

[EHWGesture](https://github.com/smilies-polito/EHWGesture) has tapping videos filmed beside a 120 fps motion capture system, so the true time of every tap is known.

- 99.7% of taps found on the volunteers held back for testing
- 0.7 points median error in rhythm variability
- 45 of 47 recordings placed in the same result band as motion capture

[HUBU-FIS](https://zenodo.org/records/17738775) has 234 phone videos from 75 people with Parkinson's disease and 43 controls. A neurologist rated every hand on the UPDRS finger tapping item. The analysis plan was written before any result was seen.

- Rank correlation with the neurologist's grade 0.47 (95% interval 0.33 to 0.59)
- AUC 0.86 for clearly impaired hands against unimpaired hands (95% interval 0.77 to 0.94)
- 93% of unimpaired hands graded Typical
- 73% of clearly impaired hands flagged
- 229 of 234 videos scored

These numbers describe a motor impairment in Parkinson's disease. They say nothing yet about cognitive decline.

## Setup

- Windows with Python 3.9 to 3.12
- Run `python install.py` once. It creates the environment, installs the packages and downloads the models.
- Run `.venv/Scripts/python launcher.py` to start the hub, then open port 8770 on your own machine in a browser.
- On Windows, `setup.bat` and `run_hub.bat` do the same by double click.
- `python install.py --speech-ml` adds an optional phoneme model for the speech test, about 1.4 GB.

## Using it

- Pick the camera, the language (English or 繁體中文) and the person being tested in the hub.
- Each test picks those up when it starts.
- In any test, `q` quits, `s` skips a practice phase and `v` hides the overlay so a helper can check the person's position.
- A test can also be run directly, for example `.venv/Scripts/python screening_tests/finger_tapping.py`. It then asks for a camera in the console.

## How the measurement works

- Hand, face and body points come from Google's MediaPipe models. This project is the measurement layer on top.
- Shake and tremor are measured on the raw points. The smoothing filter is only for drawing, because it would erase the signal.
- A hand partly out of the picture is not trusted. Tapping pauses and the spiral skips those frames.
- Tapping thresholds follow the size of the taps being made over the last 3 seconds.
- Each tapping run is also compared with the same person's earlier runs.
- The eye test measures every trial from the eye's own resting position, so a slow drift of the head is not counted as an eye movement.
- The tremor test records each 20 second hold first and measures every frame afterwards. It follows points on the skin of each hand, because the hand landmarks jitter on a flat resting hand.
- Every result carries a confidence score beside the verdict.

## The published dashboard

- The same dashboard is published as a static site.
- The site cannot run the tests itself. It connects to the hub on your own machine.
- Start `run_hub.bat`, open the site and press Connect.
- The browser talks only to your own machine, so recordings never reach the host.
- Chrome or Edge only. Safari blocks a local connection from a secure page.
- Build with `python tools/build_web.py`, then deploy with `firebase deploy --only hosting`.
- The download bundle is built from the current commit, so commit first.

## Tests for the code

- The engines are pure Python and their tests need no camera.
- `python screening_tests/tests/test_tapping.py` runs one engine's tests.
- `python screening_tests/tests/test_run_loops.py` drives every camera test's run loop without a camera, in about 3 minutes.
- `screening_tests/tests` has one file per engine and a few for the shared parts.

## Layout

- `launcher.py` is the hub, a local web server on the standard library only
- `launcher_web` is the hub frontend in plain JavaScript with no build step
- `screening_tests` has one entry script per test, plus the unit tests
- `core` has one engine per test and the shared camera, results, profile, language and drawing code
- `core/glove` and `firmware/glove` are the optional sensor glove
- `core/remote` and `participant` are remote sessions on a phone, a paused prototype
- `tools` has the site build, the release bundle and the dataset evaluations
- `model` holds the MediaPipe model bundles
- `assets` holds the 3D hand model, fonts and screenshots

## Sources

- Tapping. Roalf et al. 2018, Suzumura et al. 2022, Kwon et al. 2022, Li et al. 2024 (TapTalk)
- Spiral. Kachouri et al. 2021, Schroter et al. 2003, Balasubramanian et al. 2015
- Eye movement. Opwonya et al. 2022, Crawford et al. 2005, Antoniades et al. 2013
- Voice. Praat's standard measures through praat-parselmouth
- Tremor data. PADS (Varghese et al. 2024) and Parkinson@Home (Radboud University 2025)

## Origins

- The tracking pipeline began from [imadeddinedjekoune/Hand-Detection-3D](https://github.com/imadeddinedjekoune/Hand-Detection-3D), which mirrored a hand onto a rigged model in Unity.
- This project rewrote the tracker around the MediaPipe Tasks API and turned it toward motor measurement.
- The Unity side is retired.
