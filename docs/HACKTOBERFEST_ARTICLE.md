# FlatHike: Open-Source AI and On-Device Terrain Intelligence for Real-World Trails

*This is a submission for the [Hacktoberfest Open-Source AI Challenge Week 1: Touch Grass](https://dev.to/challenges/hacktoberfest-week1-2026-10-05)*

---

## What I Built

Most modern apps are designed to keep your eyes glued to glass. **FlatHike** is built for the exact opposite: to get you out into the woods, onto the ridges, and back home safely, keeping your screen time down to a few seconds of situational clarity.

When you are planning or navigating a hike in the mountains, conventional map apps usually treat distance as flat lines or give naive time estimates based on standard walking speed. But in real backcountry terrain, **grade and elevation change dictate everything**. A 5-kilometer flat stroll in the park takes an hour; a 5-kilometer mountain scramble with a 25% incline can take half a day.

**FlatHike** is an open-source, privacy-first Android application designed for hikers, trail runners, and alpine adventurers. It analyzes real GPS telemetry (GPX, KML, GeoJSON, CSV/TXT) and elevation profiles using digital elevation models (DEM) and biomechanical velocity algorithms, combined with **on-device open-weight AI (Google Gemma)** that runs completely offline with zero internet access.

<p align="center">
  <img src="https://raw.githubusercontent.com/torrua/FlatHike/main/docs/screenshots/01_overview_metrics.png" alt="Flat Surface Model vs Recorded Backcountry Data" width="360" />
</p>

*Route overview: contrasting idealized flat-ground velocity against real recorded alpine speed (1.8 km/h on steep terrain) and pacing telemetry.*

### Key Capabilities:
- **Interactive Elevation & Waypoint Profiler:** Visualizes trail elevation profiles with interactive crosshairs, intermediate landmarks, pass/summit waypoints, and pinpoint inspection along continuous track arc-lengths.
- **Biomechanical Slope Speed Modeling (Tobler's Hiking Function):** Automatically categorizes trails into 5 terrain slope classes (*Steep Up*, *Moderate Up*, *Flat / Gentle*, *Moderate Down*, *Steep Down*) and calculates actual speed versus empirical human hiking physiology.
- **Kilometer & Waypoint Split Analysis:** Breaks tracks down automatically by kilometer splits or manually between marked landmarks (e.g., *Start → Mountain Pass → Alpine Lake*).
- **100% Offline Trail AI Companion (Powered by Google Gemma):** Powered by open-weight Gemma models executed locally on-device via Google MediaPipe LLM Inference—delivering trail safety assessments, weather gear checklists, and pacing advice with zero cell signal.
- **Adaptive Alpine UI:** Complete Material 3 Light and Dark alpine themes, plus full bilingual support (English and Russian).

---

## Demo & Visual Walkthrough

The app is published and ready to install on Android devices.

- **GitHub Release:** [FlatHike v1.5.1 on GitHub Releases](https://github.com/torrua/FlatHike/releases/tag/v1.5.1)
- **Direct Download:** [app-debug.apk](https://github.com/torrua/FlatHike/releases/download/v1.5.1/app-debug.apk)

### 1. 3D Mountain Geodetics & Ellipsoid Altitude Correction
When navigating high-altitude ridges (ascending past 4,000 meters), FlatHike computes real 3D arc lengths and applies WGS-84 geodetic corrections to account for the difference between sea-level map projections and real physical trail length at high elevations:

<p align="center">
  <img src="https://raw.githubusercontent.com/torrua/FlatHike/main/docs/screenshots/02_geodetic_parameters.png" alt="WGS-84 Geodetic Parameters and Altitude Correction" width="360" />
</p>

*WGS-84 Geodetic parameters showing elevation reduction to ellipsoid (-13.2 m) and 3D chord adjustments above geoid.*

### 2. Interactive Elevation Profiler & Grade Speed Breakdown
Hikers can tap anywhere along the elevation chart to inspect instantaneous altitude and gradient, and view pace broken down by grade:

<p align="center">
  <img src="https://raw.githubusercontent.com/torrua/FlatHike/main/docs/screenshots/03_elevation_and_slope_speed.png" alt="Elevation Profile and Speed vs Slope Gradient" width="360" />
</p>

*Interactive elevation profile with crosshair (3644m at km 16.73) and five-tier slope categorization: Steep Ascent (>15%), Moderate Ascent (5–15%), Flat/Gentle (±5%), Moderate Descent (-15…-5%), and Steep Descent (<-15%).*

### 3. Kilometer Splits & On-Device Gemma Trail Assistant
The trail is dissected into segments with detailed climb rates, accompanied by the built-in AI Gemma trail assistant ready to answer pacing questions:

<p align="center">
  <img src="https://raw.githubusercontent.com/torrua/FlatHike/main/docs/screenshots/04_splits_and_gemma_assistant.png" alt="Splits and AI Gemma Trail Assistant" width="360" />
</p>

*Kilometer splits (5 km segments shown) showing grade and elevation change, alongside the on-device AI Gemma Trail Assistant ready to generate full route safety reports.*

### 4. Edge AI Settings: Local Gemma Inference Modes
FlatHike gives hikers full control over how Gemma operates, defaulting to an instant offline expert mode with zero data downloads:

<p align="center">
  <img src="https://raw.githubusercontent.com/torrua/FlatHike/main/docs/screenshots/05_gemma_settings.png" alt="Gemma Neural Model Modes" width="360" />
</p>

*Settings modal showing Gemma execution modes: Built-in instant offline expert, on-device MediaPipe GPU/CPU neural model, and cloud fallback.*

---

## Code

FlatHike is fully open-source under the Apache-2.0 License:

{% github torrua/FlatHike %}

- **Repository:** [https://github.com/torrua/FlatHike](https://github.com/torrua/FlatHike)
- **CI/CD Pipeline:** Fully automated [GitHub Actions Build & Test Workflow](https://github.com/torrua/FlatHike/actions), ensuring every commit passes all 47 unit tests and compiles clean Android binaries.

---

## How I Built It

Building software for the outdoors imposes strict constraints: **battery efficiency, offline resilience, and mathematical precision**.

### 1. Open-Source AI on the Edge: Google Gemma & Local MediaPipe Inference
Cloud AI is useless at 2,500 meters altitude where cell service vanishes. To give hikers an intelligent guide that works without the internet, FlatHike integrates **Google's open-weight Gemma model family** (Gemma-2B and Gemma-3B / PaliGemma) executed directly on the user's mobile device via the **Google MediaPipe Tasks GenAI API** (`com.google.mediapipe:tasks-genai`).

- **Architecture:** The model weights (`.bin` / `.task`) live locally in app storage.
- **Inference Pipeline:** Runs on-device hardware accelerators (GPU/NPU via OpenCL/Vulkan backend) with low thermal overhead.
- **Trail System Prompts:** Structured domain prompts evaluate trail difficulty, elevation gain, slope gradients, and biomechanical energy baselines, generating concise, safety-critical advice without sending a single byte over the wire.
- **Pre-quantized Weights:** FlatHike supports pre-quantized model weights (int4 / int8) to match the RAM budget of mobile devices while preserving reasoning quality for trail decision-making.

### 2. Biomechanical Velocity & Geodetic Algorithms
Instead of simplistic averages, FlatHike models hiking velocity based on Waldo Tobler's Hiking Function:

$$W = 6 \cdot e^{-3.5 \cdot \left|\tan(\theta) + 0.05\right|}$$

where:
- $\theta$ is the slope angle of the trail.
- $\tan(\theta)$ is the slope gradient ($dh / dx$).
- $W$ is the predicted walking speed in $\text{km/h}$.

The app calculates exact cumulative arc lengths along the GPS track using spherical Haversine trigonometry with chord interpolation. It segments the path into continuous grade classes:
- **Steep Ascent ($> +15\%$)**
- **Moderate Ascent ($+5\%$ to $+15\%$)**
- **Flat / Gentle ($-5\%$ to $+5\%$)**
- **Moderate Descent ($-15\%$ to $-5\%$)**
- **Steep Descent ($< -15\%$)**

This allows hikers to diagnose exactly where their energy was spent and compare their recorded telemetry to theoretical physiological baselines.

### 3. Modern Reactive UI & Clean Android Architecture
- **Language & UI:** 100% Kotlin with Jetpack Compose and Material Design 3.
- **State Management:** MVVM with unidirectional data flow via Kotlin Coroutines `StateFlow`.
- **Canvas Rendering:** High-performance custom hardware-accelerated Compose `Canvas` drawing the elevation profile, gradient shading, and touch-drag crosshairs.
- **Localization:** Dynamic runtime language switching (English and Russian) without requiring an app restart.

---

## Why Does Open Innovation Matter?

This year's Hacktoberfest theme—*Touch Grass*—highlights the true potential of open-source artificial intelligence: **empowering software that operates in the real physical world, not just inside a server rack.**

1. **Safety and Autonomy in the Backcountry:** Closed AI APIs require persistent internet connectivity, monthly subscription fees, and reliable cloud infrastructure. In the wilderness, those assumptions collapse. Open-weight models like Gemma allow developers to decouple intelligence from cloud servers, bringing life-saving analysis to any pocket anywhere on Earth.
2. **Data Privacy & Location Sovereignty:** GPS tracks reveal intimate details about where you live, when you leave your house, and where you camp. Using closed proprietary cloud models means uploading your sensitive geotagged location history to third-party data centers. Open-source models running locally keep 100% of your location data on your phone.
3. **Scientific Transparency:** The algorithms that estimate hiking time, calorie burn, and trail hazard should not be a proprietary black box. By keeping FlatHike open-source, the outdoor community can verify, audit, and improve the geodetic math and biomechanical models collaboratively.

---

## Prize Categories

- **Best Use of Gemma** (Featured Category)
  FlatHike makes central, real-world use of Google's open-weight **Gemma** model family (Gemma 2B and Gemma 3B / PaliGemma) running 100% locally on Android devices via Google MediaPipe Tasks GenAI. This demonstrates how open-weight foundation models can empower life-safety applications in disconnected environments without depending on cloud APIs.

---

## What's Next

- **Field Testing on Autumn Trails:** Taking FlatHike onto regional ridge trails to benchmark Gemma 2B on-device battery drain under freezing conditions.
- **Offline Topographic Contour Maps:** Pairing the elevation profile with offline vector contour tiles (OpenMapTiles / Mapsforge).
- **Wear OS Companion App:** A lightweight wrist glance showing immediate slope angle and next waypoint distance, letting you keep the phone zipped in your backpack.

---

*Get outside, stay safe on the trail, and enjoy the mountains!*
