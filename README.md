# Technical Audit and Pipeline Architecture: Marine Acoustic Monitoring - Galápagos

## 1. Project Overview
This repository contains the processing pipeline for analyzing the underwater soundscape in the Bay of San Cristóbal, Galápagos. The objective is to process unlabeled hydrophone recordings to detect biological events (cetacean vocalizations, invertebrates, fish) in the presence of anthropogenic noise (boat engines).

The analysis is divided into two main approaches:
* **Tier 1 (Classical Ecoacoustics):** Extraction of mathematical indices (ACI, NDSI, Spectral/Temporal Entropy) via Digital Signal Processing (DSP).
* **Tier 2 (Deep Learning):** Latent feature extraction (embeddings) using pre-trained models (Transfer Learning) and topological dimensionality reduction.

---

## 2. Data Flow Architecture

The current system operates under the following sequential flowchart:

1.  **Data Acquisition:** Parallel download from Cloudflare R2 of data from three hydrophone units with different capabilities (Pilot at 48 kHz, 6478 at 96 kHz, 5783 at 144 kHz).
2.  **Preprocessing & Tier 1:** Audio loading, high-pass filtering, and spectrogram (STFT) calculation to derive complexity and power density indices.
3.  **Tier 2 (AI Inference):** Audio resampling and passing through the Perch 2.0 convolutional model to extract high-dimensional embedding tensors.
4.  **Dimensionality Reduction & Visualization:** Projection of the latent space to 2D using UMAP and generation of statistical and temporal dashboards.

---

## 3. Recommendations and Remediation Roadmap

To endow this pipeline with research-grade rigor, the following adjustments must be implemented:

1.  **Fundamental Arithmetic Correction:** Refactor all acoustic energy aggregation functions to operate on the **Power Spectral Density (PSD)** in a linear scale.
2.  **Zero-Phase Filtering:** Replace the causal filter with a forward-backward filter (`scipy.signal.sosfiltfilt`) to preserve waveform morphology.
3.  **Time Window Calibration:** Regardless of the hydrophone's sample rate, force the block size ($N_{FFT}$) to represent the exact same physical time (e.g., strict 10 ms or 20 ms windows).
4.  **Dynamic Extraction (Sliding Windows):** Implement sliding windows across the entire audio file so the neural network extracts local embeddings, using temporal max-pooling techniques to capture the peak of the transient event.
5.  **Timezone Alignment:** Adjust the master DataFrame using the local conversion function `pd.Timedelta(hours=6)`.
