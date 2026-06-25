# DL + Blockchain Pipeline for Diabetic Retinopathy Early Diagnosis

This repository contains the ongoing development of an internship project that integrates Deep Learning and Blockchain technology for the secure and automated diagnosis of Diabetic Retinopathy (DR). The goal is to use a Deep Learning model for multi-class retinal image classification and an Ethereum smart contract to maintain a tamper-proof, access-controlled ledger of diagnosis records.

---

## Current Implementation Status

### 1. Deep Learning Model
- **Environment & Dataset:** Developed using Google Colab with GPU acceleration on the APTOS 2019 dataset (2,930 training images across 5 DR severity classes: No DR, Mild, Moderate, Severe, Proliferative).
- **Class Imbalance Handling:** Applied targeted augmentation on minority classes (Mild, Severe, Proliferative) to balance the dataset to ~600 samples per class. Custom class weights were also applied to prioritize early-stage and severe DR detection.
- **Architecture:** Transfer learning using EfficientNetB3 pretrained on ImageNet, with fine-tuning of the last 30 layers. Custom classification head: GlobalAveragePooling2D → Dropout(0.3) → Dense(5, softmax).
- **Training:** Adam optimizer, sparse categorical crossentropy loss, EarlyStopping with patience=5. Model saved to Google Drive.
- **Evaluation Results (Validation Set — 586 images):**

| Class | Precision | Recall | F1-Score |
|---|---|---|---|
| No DR | 0.95 | 0.97 | 0.96 |
| Mild | 0.47 | 0.60 | 0.52 |
| Moderate | 0.69 | 0.82 | 0.75 |
| Severe | 0.83 | 0.17 | 0.28 |
| Proliferative | 0.82 | 0.29 | 0.43 |
| **Overall Accuracy** | | **0.79** | |

### 2. Smart Contract
- **Contract Name:** `DRDiagnosisResuls` — deployed on Sepolia testnet.
- **Core Structures:** Implements `Patient` registry and `Diagnosis` records containing cryptographic image hashes (`bytes32`) and multi-class DR category (`uint`).
- **Access Control:**
  - `onlyOwner` modifier ensures only the hospital admin can register patients and assign doctors.
  - `uploadDiagnosis` restricts diagnosis uploads exclusively to the doctor assigned to that patient.
  - `viewRecords` allows the admin and assigned doctor to query historical diagnosis records.
  - Duplicate patient registration prevented via timestamp-based existence check.
- **Events:** `PatientRegistered` and `DiagnosisUploaded` emitted for on-chain activity tracking.

### 3. Python-Blockchain Bridge
- **Web3.py Integration:** Connected to Sepolia testnet via Alchemy RPC endpoint.
- **Patient Registration:** Automated pipeline — generates synthetic patient profiles using Faker, persists records to JSON, and registers patients on-chain via smart contract.

---

## Repository Structure
- `DR-Diagnosis.sol` — Solidity smart contract managing patient registry, diagnosis records, and access control.
- `Copy_of_DR_DL_training.ipynb` — Jupyter Notebook covering data preprocessing, augmentation, model training, and evaluation.
- `DR_Blockchain_Web3.ipynb` — Jupyter Notebook covering Web3.py integration, patient registration pipeline, and blockchain interaction with the deployed smart contract.

---

## Next Steps
- **Diagnosis Upload:** Integrate model inference with `uploadDiagnosis` — compute `keccak256` hash of retinal image, run classification, and log result on-chain.
- **End-to-End Pipeline:** Connect full flow — test image input → model prediction → hash generation → blockchain upload → on-chain verification.
- **Persistent Storage:** Move JSON patient store and dataset to Google Drive to survive Colab session resets.
- **Model Improvement:** Improve recall for Severe (0.17) and Proliferative (0.29) classes through further fine-tuning.
- **Validation:** End-to-end testing with mock patient profiles to verify access control, record retrieval, and tamper detection.
