# Khmer Text-to-Speech (VITS)

**ITM 454 Natural Language Processing · Fall 2026 · American University of Phnom Penh**
**Group 5:** AN Sovannchakriya · CHAN Natalie · KUYSENG Chesda · LY Mengchou
**Instructor:** Monyrath Buntoun

We built a system that reads Khmer text out loud. We took a pretrained **VITS** speech model and fine-tuned it on about 20 hours of recordings from one native Khmer speaker. Then we compared it with real recordings and with Meta's public **MMS-TTS Khmer** model.

---

## Results

Evaluated model: `best_model.pth` (epoch 13). 399 unseen test sentences, plus 10 new everyday sentences.

| | CER ↓ | CER, new sentences ↓ | WER ↓ | CER, 2nd judge (Wav2Vec2) ↓ | MCD ↓ | Duration ratio |
|---|---|---|---|---|---|---|
| Real recordings | 0.7% | – | 2.6% | 12.9% | – | – |
| **Our model** | **6.7%** | **4.7%** | **16.3%** | **20.3%** | **4.90 dB** | **1.01** |
| MMS-TTS Khmer | 20.2% | 24.3% | 41.5% | 32.2% | 7.30 dB | 1.21 |

- **CER** = how many characters a speech-recognition model gets wrong when it listens to the audio (lower is better). The main judge is `seanghay/Qwen3-ASR-0.6B-Khmer`; the second judge is `gagan3012/wav2vec2-xlsr-khmer`.
- **WER** = the same at word level. Khmer has no spaces between words, so sentences are first split into words with `khmercut`.
- **MCD** = how far the sound is from the real recording. **Duration ratio** 1.00 = the real speaker's pace.

What this shows:

- Our model makes about **3× fewer errors** than MMS-TTS Khmer, and both speech-recognition judges agree on that ranking.
- **76%** of test sentences are near-perfect (CER ≤ 10%). Only **8 of 399** have clear mistakes, mostly rare words such as farm-tool and ingredient names.
- It speaks at the speaker's natural pace (1.01), while MMS is about 21% slower.
- The hardest character is the repeat mark **ៗ** (52% error): the model often doesn't repeat the word.

---

## Graphs

**Error rate by system:** our model vs MMS-TTS Khmer vs real recordings, on test and new sentences.
<p align="center"><img src="figures/cer_by_system.png" width="70%"></p>

**Two speech-recognition judges:** Qwen3-ASR and Wav2Vec2 give the same ranking.
<p align="center"><img src="figures/cer_two_asr_judges.png" width="70%"></p>

**Error rate by topic:** Traditional Home Appliances is the hardest, Etiquettes the easiest.
<p align="center"><img src="figures/cer_by_topic.png" width="70%"></p>

**Error rate by sentence length:** short sentences are slightly harder.
<p align="center"><img src="figures/cer_by_length.png" width="70%"></p>

**Sound difference (MCD) and speaking speed:** our model is closer to the real voice and speaks at a natural pace.
<p align="center"><img src="figures/mcd_duration_hist.png" width="85%"></p>

**Spectrograms:** the same sentence spoken by the real speaker, our model and MMS.
<p align="center"><img src="figures/spectrogram_comparison.png" width="85%"></p>

**Training loss:** validation loss for each epoch. The best model was at epoch 13.
<p align="center"><img src="training/loss_curve.png" width="70%"></p>

---

## Folder structure

```
khmer-tts-nlp/
├── notebooks/
│   ├── 1_text_preprocessing.ipynb   Step 1: clean the Khmer text
│   ├── 2_audio_preprocessing.ipynb  Step 2: check and prepare the audio
│   ├── 3_vits_training.ipynb        Step 3: train the model
│   └── 4_evaluation.ipynb           Step 4: test the model (CER, WER, MCD, two ASR judges)
├── training/    training summary and loss curve
├── results/     all result tables (CSV), transcripts and the Wav2Vec2 vocab.json
├── figures/     charts
└── docs/        project proposal and progress report
```

The dataset, audio files and trained model are too large for GitHub. They are kept on Google Drive in `NLPfinalproject/`. In Colab, Step 4 is the notebook `step4_fine-tuning_updated.ipynb`, and it saves to `NLPfinalproject/step4_fine-tuning_updated/`.

---

## How it works

1. **Text:** use one speaker (`f-adt2-0002`), clean the Khmer text and remove bad sentences → 10,621 sentences.
2. **Audio:** remove noisy or broken clips, convert to 22 kHz and split by paragraph into train / validation / test → 9,616 / 395 / 399 clips.
3. **Training:** fine-tune VITS (Coqui TTS) from the English LJSpeech model on a Colab GPU. Training stopped after 20 epochs; the best result came at epoch 13.
4. **Evaluation:** generate speech for the test sentences and 10 new sentences, then measure:
   - **CER and WER** with two Khmer speech-recognition models (Qwen3-ASR and Wav2Vec2)
   - **MCD**, which compares the sound with the real recordings
   - **speed** (duration ratio and real-time factor)
   - **errors**, sorted by topic, sentence length and character

   The notebook also prepares a blind listening test (MOS, 65 clips).

**Dataset:** Khmer ASR Cultural Dataset V2 by Digital Divide Data (CC BY-SA 4.0).

---

## How to run

1. In Google Drive, add a shortcut to the `NLPfinalproject` folder in **My Drive**.
2. Open the notebooks in order in Google Colab.
3. For Steps 3 and 4, choose **Runtime → Change runtime type → T4 GPU**, then **Runtime → Run all**.

---

## Still to do

- **Listening test (MOS):** native Khmer speakers rate the audio from 1 to 5. The clips are ready; the ratings aren't collected yet.
- **Train longer:** the model trained for 20 epochs. More data and longer training should make it better.
- **Fix ៗ and numbers:** write the repeated word out in full and spell out digits before training.

## Limitations

- One speaker, and mostly cultural topics.
- CER and WER depend on speech-recognition models (Wav2Vec2 gets real speech 12.9% wrong), so listener ratings (MOS) are still needed.
- The new-sentence test has only 10 sentences.
- Sentences with digits or Latin letters were removed, so the model can't read them.

## References

- Kim et al. (2021). *VITS: Conditional Variational Autoencoder with Adversarial Learning for End-to-End Text-to-Speech.* ICML.
- Coqui TTS (2021). doi:10.5281/zenodo.6334862
- Digital Divide Data (2024). *Khmer ASR Cultural Dataset.*
- Pratap et al. (2023). *Scaling Speech Technology to 1,000+ Languages (MMS).* Meta AI.
- Seanghay (2025). *Qwen3-ASR-0.6B-Khmer.* Hugging Face.
- gagan3012. *wav2vec2-xlsr-khmer.* Hugging Face.
