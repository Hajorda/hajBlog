---
external: false
title: "Open Source Turkish TTS: A Developer's Guide to Every Viable Option"
description: "Building speech applications in Turkish? Here's a thorough breakdown of every open-source model worth your time — what works, what doesn't, and why Turkish is harder than you think."
date: 2026-05-19
---

![cHeaderBanner](/images/turkish-tts.png "Banner")

*Building speech applications in Turkish? Here's a thorough breakdown of every open-source model worth your time — what works, what doesn't, and why Turkish is harder than you think.*

---

Turkish is a fascinating language for speech synthesis. It's almost perfectly phonetic — letters map consistently to sounds, unlike English with its notorious irregularities. You'd think that would make Turkish TTS a solved problem. And yet, finding a truly high-quality, open-source Turkish TTS model in 2025 still requires some digging. The reason: Turkish is agglutinative. Words grow long chains of suffixes, and a model needs to handle a massive vocabulary of unique word forms to synthesize natural-sounding speech. That's a data problem, and data is exactly what Turkish has historically lacked.

This article maps every serious open-source option available today — from multilingual giants to Turkish-specific fine-tunes — so you can pick the right tool for your use case.

---

## Why Turkish TTS Is Harder Than It Looks

Before diving into the models, it's worth understanding the core challenge. Turkish's agglutinative structure means that a single root word can spawn hundreds of grammatically valid forms. To synthesize speech that sounds natural, a model needs exposure to a wide variety of these word forms — otherwise, it stumbles on rare but perfectly valid compound suffixes. Researchers have found that training corpora for Turkish TTS need to prioritize vocabulary diversity (maximizing unique words voiced) over sheer hour count. A 10-hour dataset with high lexical variety will outperform a 30-hour monotone corpus.

Vowel harmony — where suffixes change their vowels to match the root — adds another wrinkle for phoneme-based systems. And prosody (the rhythm and intonation of spoken Turkish) is subtly distinct from Indo-European languages, making models pre-trained on European speech less immediately transferable.

With that context, let's look at what's available.

---

## 1. Coqui XTTS-v2 — The Community's First Choice

**GitHub / HuggingFace:** `coqui/XTTS-v2`
**License:** Coqui Public Model License (CPML) — non-commercial by default
**Architecture:** GPT-based autoregressive + HiFi-GAN vocoder

XTTS-v2 is arguably the most capable open-source multilingual TTS system for Turkish right now. It supports 17 languages out of the box — Turkish included — and its standout feature is **zero-shot voice cloning**: give it just 6 seconds of any speaker's audio and it will synthesize new speech in that voice.

The model uses a GPT-style autoregressive backbone to generate audio tokens conditioned on both text and a speaker embedding, then decodes them through a HiFi-GAN neural vocoder. For Turkish, this architecture handles the language's phonetic regularity well, producing fluid, natural-sounding output with correct vowel harmony in most cases.

The bad news: Coqui AI shut down in early 2024, leaving XTTS as a community-maintained project. Development has slowed, but the codebase is stable and actively used.

```python
from TTS.api import TTS

tts = TTS("tts_models/multilingual/multi-dataset/xtts_v2", gpu=True)
tts.tts_to_file(
    text="Merhaba, bu bir Türkçe ses sentezi testidir.",
    file_path="output.wav",
    speaker_wav="reference_speaker.wav",
    language="tr"
)
```

**Best for:** Applications needing voice cloning, conversational agents, content localization.
**Watch out for:** CPML restricts commercial use; check licensing carefully.

---

## 2. Coqui TTS — Glow-TTS on Turkish Common Voice

**Model string:** `tts_models/tr/common-voice/glow-tts`
**License:** Mozilla Public License 2.0
**Architecture:** Glow-TTS (flow-based) + MelGAN vocoder

Before XTTS, the Coqui TTS toolkit shipped a dedicated Turkish model trained on Mozilla's Common Voice dataset. Glow-TTS is a non-autoregressive, flow-based architecture that generates mel spectrograms in parallel — meaning faster inference than autoregressive models.

The Turkish Common Voice training data is community-contributed, which means variable recording quality. The resulting model is intelligible but noticeably more robotic than XTTS-v2. Still, it's a solid baseline for low-latency applications and requires no reference audio — just text in, speech out.

```bash
tts --text "Bugün hava çok güzel." \
    --model_name tts_models/tr/common-voice/glow-tts \
    --out_path output.wav
```

**Best for:** Fast prototyping, embedded systems where voice cloning isn't needed.
**Watch out for:** Lower naturalness than XTTS; Common Voice quality varies.

---

## 3. Facebook MMS-TTS (Turkish)

**HuggingFace:** `facebook/mms-tts-tur`
**License:** CC-BY-NC 4.0
**Architecture:** VITS (Variational Inference with adversarial learning for end-to-end TTS)

Meta's Massively Multilingual Speech (MMS) project set out to build speech technology for over 1,000 languages. The Turkish checkpoint is part of this initiative and uses the **VITS** architecture — a fully end-to-end model that jointly trains acoustic modeling and waveform generation. VITS eliminates the two-stage pipeline of older systems, reducing compounding errors.

The model uses a unified multilingual architecture based on Transformer and HiFi-GAN technologies, which allows it to handle typologically diverse languages including agglutinative ones like Turkish. Cross-lingual phoneme representations and joint acoustic modeling make it particularly effective for lower-resource languages.

```python
from transformers import VitsModel, AutoTokenizer
import torch

model = VitsModel.from_pretrained("facebook/mms-tts-tur")
tokenizer = AutoTokenizer.from_pretrained("facebook/mms-tts-tur")

inputs = tokenizer("Açık kaynaklı Türkçe ses sentezi.", return_tensors="pt")
with torch.no_grad():
    output = model(**inputs).waveform
```

**Best for:** Research, applications where a single-speaker, stable neutral voice is sufficient.
**Watch out for:** Non-commercial license; single speaker only (no voice cloning).

---

## 4. Microsoft SpeechT5 — Turkish Fine-Tune

**HuggingFace:** `deryauysal/speecht5_tts_common_voice_tr`
**License:** MIT (base model) — check fine-tune card
**Architecture:** SpeechT5 (encoder-decoder Transformer)

Microsoft's SpeechT5 is a versatile speech pre-trained model that handles both TTS and ASR tasks in a unified architecture. Community members have fine-tuned it on Turkish Common Voice data, creating a viable Turkish TTS checkpoint.

SpeechT5 conditions speech generation on speaker embeddings from a separate embedding dataset, giving you some speaker variability without full voice cloning. Performance is on par with the Coqui Common Voice model — solid intelligibility, reasonable prosody, not quite at XTTS-v2 naturalness.

```python
from transformers import SpeechT5Processor, SpeechT5ForTextToSpeech, SpeechT5HifiGan
from datasets import load_dataset
import torch

processor = SpeechT5Processor.from_pretrained("deryauysal/speecht5_tts_common_voice_tr")
model = SpeechT5ForTextToSpeech.from_pretrained("deryauysal/speecht5_tts_common_voice_tr")
vocoder = SpeechT5HifiGan.from_pretrained("microsoft/speecht5_hifigan")

inputs = processor(text="Türkçe metin okuma sistemi.", return_tensors="pt")
embeddings_dataset = load_dataset("Matthijs/cmu-arctic-xvectors", split="validation")
speaker_embeddings = torch.tensor(embeddings_dataset[7306]["xvector"]).unsqueeze(0)

speech = model.generate_speech(inputs["input_ids"], speaker_embeddings, vocoder=vocoder)
```

**Best for:** Projects already using the HuggingFace Transformers ecosystem.

---

## 5. TurkicTTS — Multi-Language Turkic Family Model

**GitHub:** `IS2AI/TurkicTTS`
**License:** Apache 2.0
**Architecture:** Tacotron2 + Parallel WaveGAN

If your application needs to span multiple Turkic languages — Azerbaijani, Kazakh, Uzbek, and others alongside Turkish — TurkicTTS is a purpose-built solution. Developed by the Institute of Smart Systems and Artificial Intelligence (ISSAI) in Kazakhstan, it covers ten Turkic languages in a single model.

It builds on ESPnet2's TTS infrastructure with Tacotron2 for acoustic modeling and a Parallel WaveGAN neural vocoder. The model leverages shared phonological features across Turkic languages, which benefits lower-resource variants. For Turkish specifically, quality is comparable to other community models — usable, though not state-of-the-art.

**Best for:** Multilingual Turkic-region applications, accessibility tools, educational software serving Central Asian and Turkish audiences together.

---

## 6. FastPitch + HiFi-GAN from Scratch

**GitHub:** `Rumeysakeskin/Turkish-Text-to-Speech`
**License:** Open (check repo)
**Architecture:** FastPitch (non-autoregressive) + HiFi-GAN vocoder

This repository demonstrates training a Turkish TTS system from scratch using NVIDIA's FastPitch — a parallel spectrogram predictor that also explicitly models pitch. Rather than relying on multilingual transfer, it trains entirely on Turkish data, giving the model a stronger grounding in Turkish prosody.

This approach is more of a starting point for practitioners who want to train their own models on custom Turkish speech data. The repo includes data preprocessing pipelines, training recipes, and inference code. For teams with proprietary Turkish voice data, this is a valuable reference implementation.

**Best for:** Organizations that need to fine-tune on proprietary speaker data; researchers studying Turkish prosody.
**Watch out for:** Requires more setup than the HuggingFace plug-and-play options.

---

## 7. eSpeak NG — The Lightweight Legacy Option

**Website:** espeak-ng.org
**License:** GPL v3
**Architecture:** Formant synthesis

eSpeak NG is a compact, rule-based text-to-speech engine that predates neural TTS by decades. It supports Turkish through phoneme rules and a built-in pronunciation dictionary. The output is distinctly robotic by modern standards — unmistakably synthetic — but it runs on virtually any hardware, including microcontrollers.

MaryTTS, a Java-based multilingual synthesis platform, also includes Turkish support and offers slightly higher quality through unit-selection synthesis, while still being installable in Java applications via API.

Neither will satisfy users expecting human-like speech, but for embedded systems, screen readers on constrained hardware, or applications where latency absolutely cannot exceed a few milliseconds, they remain relevant.

**Best for:** Embedded devices, legacy system integration, accessibility tools on constrained hardware.

---

## Comparison at a Glance

| Model | Naturalness | Voice Cloning | License | Inference Speed | Best Use Case |
|---|---|---|---|---|---|
| **XTTS-v2** | ⭐⭐⭐⭐⭐ | ✅ Zero-shot | CPML (non-commercial) | Medium | Voice cloning apps |
| **MMS-TTS (Facebook)** | ⭐⭐⭐ | ❌ | CC-BY-NC 4.0 | Fast | Research, neutral voice |
| **Coqui Glow-TTS** | ⭐⭐⭐ | ❌ | MPL 2.0 | Very fast | Quick prototyping |
| **SpeechT5 (fine-tuned)** | ⭐⭐⭐ | Partial | MIT | Medium | HuggingFace pipelines |
| **TurkicTTS** | ⭐⭐⭐ | ❌ | Apache 2.0 | Medium | Multi-Turkic language |
| **FastPitch + HiFi-GAN** | ⭐⭐⭐⭐ | ❌ (trainable) | Open | Fast | Custom training |
| **eSpeak NG** | ⭐ | ❌ | GPL v3 | Extremely fast | Embedded / constrained |

---

## Practical Recommendations

**If you need voice cloning or the most natural output:** XTTS-v2 is the clear winner. Just verify your use case against the CPML license terms — for commercial applications, you may need to fine-tune an alternative or seek a commercial license.

**If you need a commercially permissive license:** Look toward TurkicTTS (Apache 2.0) or the FastPitch training approach with your own data. The SpeechT5 base model is MIT-licensed; verify the fine-tune card.

**If you're building for a Turkic multilingual audience:** TurkicTTS was specifically designed for this scenario and handles the shared phonological space of the Turkic family elegantly.

**If you want to train your own model:** The FastPitch + HiFi-GAN repository provides a well-documented recipe. Pair it with Common Voice Turkish data (available from Mozilla) or TTTS-style custom recording sessions if you need a specific speaker voice.

**If you're deploying to constrained hardware:** eSpeak NG is the only realistic option at the extreme end; MMS-TTS or Coqui Glow-TTS work well on CPU for less constrained setups.

---

## What's Still Missing

Turkish TTS remains an underserved space compared to English, French, or German. A few things the community would benefit from:

- **A large, high-quality, open Turkish single-speaker corpus** — something equivalent to LJSpeech but in Turkish. Most existing models rely on Common Voice, which has variable recording quality.
- **Emotion-conditioned Turkish TTS** — no open model currently handles expressive, emotion-aware synthesis for Turkish.
- **Streaming-optimized Turkish TTS** — real-time applications (voice assistants, telephony) need chunk-by-chunk audio generation; XTTS has some streaming support but Turkish-optimized streaming models are rare.

The field moves fast. Given how quickly multilingual foundation models are improving, it's plausible that a future XTTS iteration or a derivative model will raise the bar considerably.

---

## Getting Started Today

For most developers, the fastest path to production-quality Turkish speech is:

```bash
pip install TTS
python -c "
from TTS.api import TTS
tts = TTS('tts_models/multilingual/multi-dataset/xtts_v2', gpu=False)
tts.tts_to_file(
    text='Merhaba dünya.',
    file_path='merhaba.wav',
    speaker_wav='your_reference.wav',
    language='tr'
)
"
```

Or if you prefer the HuggingFace ecosystem:

```bash
pip install transformers torch
# then use facebook/mms-tts-tur as shown above
```

Turkish speech synthesis has come a long way. It's no longer a niche research problem — it's an engineering choice. Pick the model that fits your latency budget, license requirements, and naturalness bar, and you'll be generating Turkish speech in minutes.

---

*Have you built something with Turkish TTS? Tried a model not listed here? Drop a comment — the Turkish NLP community is small and benefits enormously from shared experiences.*