# Cog.ai Technical Stack & Architecture

This document outlines the detailed architecture and technology choices behind Cog.ai, a platform designed for the early detection and monitoring of cognitive decline using multimodal AI analysis.

---

## 1. System Architecture Overview

Cog.ai utilizes a modern, decoupled client-server architecture:
*   **Client (Frontend)**: A React-based Single Page Application (SPA) responsible for presenting a premium UI, handling interactive cognitive games, and securely capturing raw user data (audio recordings, input timings, and form responses).
*   **Server (Backend)**: A high-performance Python FastAPI service dedicated to processing complex audio/text data streams, interacting with machine learning pipelines, and securely storing patient data.

---

## 2. Frontend (Client-Side Interface)

The frontend prioritizes accessibility, speed, and a high-end "cyberpunk medical" aesthetic to instill both trust and technological superiority.

### Core Frameworks
*   **React 18**: The fundamental UI library, utilizing hooks for state management.
*   **TypeScript**: Enforces strict typing across the entire frontend, drastically reducing runtime errors and improving developer experience, especially when dealing with complex API payloads and assessment states.
*   **Vite**: The build tool and bundler. Chosen over Webpack/CRA for near-instant Hot Module Replacement (HMR) and highly optimized production builds.

### UI & Styling
*   **Tailwind CSS**: The primary styling engine for rapid, utility-first UI development.
*   **Radix UI / shadcn/ui**: Provides highly accessible, unstyled UI primitives. `shadcn/ui` is used to assemble these primitives into customized, theme-able components (Dialogs, Accords, Dropdowns, Progress bars) without the bloat of traditional component libraries.
*   **Lucide React**: The icon library of choice for clean, scalable, and modern iconography.
*   **Next-Themes**: Manages seamless switching between Dark and Light color palettes.

### State, Routing & Data
*   **React Router DOM v6**: Handles complex client-side routing, protected routes (e.g., locking the assessment workflow behind authentication), and smooth page transitions.
*   **React Hook Form + Zod**: `React Hook Form` handles form state efficiently without unnecessary re-renders, while `Zod` defines rigid schema validation for user inputs (like login, signup, and clinician configurations).

### Visualizations & Media
*   **Recharts & Chart.js**: Used in the Clinician Dashboard and "AI Risk Summary" to render complex, multi-dimensional risk analysis data into digestible radar charts, bar graphs, and line trends.
*   **HTML2Canvas & jsPDF**: Enables the crucial "Export to PDF" functionality for medical reports, allowing families to easily share AI findings with healthcare providers.

---

## 3. Backend (API & Processing Server)

The backend is built for asynchronous speed and heavy computational lifting, specifically tailored for ML model inference and audio processing.

### Core Framework & Server
*   **FastAPI**: A modern, extremely fast Python web framework. Chosen for its native support for asynchronous programming (`async/await`), which is critical when handling long-running tasks like audio processing without blocking the main server thread.
*   **Uvicorn**: The lightning-fast ASGI web server implementation that runs the FastAPI application.

### Database & ORM
*   **PostgreSQL** (via `psycopg`): A robust, scalable relational database for storing patient records, assessment histories, and clinician data.
*   **SQLAlchemy 2.0**: The Object-Relational Mapper (ORM), allowing developers to interact with the database using Python objects rather than raw SQL strings.
*   **Alembic**: Handles database schema migrations linearly and safely as the data models evolve.
*   **Pydantic**: Deeply integrated with FastAPI, Pydantic ensures that all incoming API requests and outgoing responses strictly adhere to defined data schemas before any logic is executed.

### Security & Authentication
*   **PyJWT**: Generates and decodes JSON Web Tokens (JWT) for stateless, secure API authentication.
*   **Passlib (with bcrypt)**: Securely hashes user passwords before database storage, protecting against database breaches.

---

## 4. Artificial Intelligence & Machine Learning Pipeline

This is the core intellectual property of Cog.ai. It takes raw multimodal data (speech and text) and transforms it into clinical insights.

### Speech Transcription & NLP
*   **Faster-Whisper**: A highly optimized version of OpenAI's Whisper model. This is used to transcribe the patient's audio recordings locally and rapidly.
*   **Language & Translation**: `langcodes` and `googletrans` are used to detect language mismatches and handle multilingual assessments, vital for accessibility in diverse regions (e.g., detecting if a user spoke Tamil instead of English).

### Acoustic Feature Extraction (Vocal Biomarkers)
The system doesn't just analyze *what* is said, but *how* it is said. Subtle vocal tremors, micro-pauses, and pitch variations are powerful early indicators of cognitive decline.
*   **Librosa**: The primary library for audio and music analysis in Python.
*   **OpenSMILE**: A specialized tool for extracting acoustic features (like MFCCs, pitch, jitter, and shimmer) from speech signals.
*   **Praat-Parselmouth**: Used for precise phonetic and voice acoustic analysis.
*   **TorchAudio & PyTorch**: Deep learning audio libraries used for complex tensor operations and feature extraction on the audio waveforms.

### Predictive Modeling Engine
*   **XGBoost**: Extreme Gradient Boosting. This is the primary algorithm used for the final risk prediction. It excels at handling tabular data (the extracted acoustic features combined with text metrics) and provides highly accurate classification.
*   **Scikit-Learn**: Used for data preprocessing (StandardScaler) and dimensionality reduction (PCA) before feeding the data into the XGBoost model. Pre-trained artifacts (`scaler.pkl`, `pca_transform.pkl`, `model.pkl`) are loaded into memory on server boot alongside joblib.
*   **Pandas & NumPy**: Used for heavy array manipulations and dataframe structuring during the data pipeline.

---

## 5. Typical Assessment Data Flow

1.  **Capture**: The user performs a task (e.g., "Story Recall") on the React frontend. The browser's MediaRecorder API captures the audio as a `webm/ogg` blob.
2.  **Transmission**: The frontend submits the audio blob and task metadata to a FastAPI endpoint (`/api/upload-audio`).
3.  **Preprocessing**: The backend receives the file, converts it into a standardized WAV format using `soundfile` or `torchaudio`.
4.  **Transcription**: The audio is passed through `Faster-Whisper` to get the exact text.
5.  **Acoustic Profiling**: The raw audio is simultaneously passed through `OpenSMILE/Librosa` to extract hundreds of acoustic features (jitter, shimmer, speech rate).
6.  **Inference**: The combined features (Text metrics + Acoustic metrics) are scaled using `scaler.pkl` and transformed using `pca_transform.pkl`. The final array is fed into the `XGBoost` model (`model.pkl`).
7.  **Result**: The model returns a continuous risk score and feature importance break-down, which is saved to PostgreSQL via SQLAlchemy.
8.  **Render**: The FastAPI server returns the structured JSON to the React frontend, which renders the comprehensive "AI Risk Summary" using Recharts.
