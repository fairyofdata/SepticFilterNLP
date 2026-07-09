# SepticFilter — Deep Learning-based Disguised Profanity Detection Filter 🤬🤖

**🌐 Available Versions:** [🇰🇷 한국어 (Korean)](/README_KR.md) | [🇯🇵 日本語 (Japanese)](/README_JP.md)

> 🏆 **Samsung SDS Multicampus Big Data Utilization Competition — Grand Prize (1st Place)** (Multicampus, 2022)  
> K-Digital Training Data Science Course Team Project

> 📌 **Successor Project:** The Korean-domain NLP expertise gained here — jamo-level tokenization, ensemble design, and community crawling — was directly applied to [AntiPhishingNLP](https://github.com/fairyofdata/AntiPhishingNLP) (KISA Grand Prize), which extended this work to smishing and voice phishing detection.

A deep learning-based profanity filter that goes beyond static blocklists to detect **disguised profanity** — such as jamo decomposition (`씨발 → ㅅㅂ`) and special character substitution (`미1친`).

---

## 🎯 Problem Definition

Conventional keyword-blocklist filters fail to detect character-level evasions like:

| Original | Disguised Form |
|---|---|
| 씨발 (expletive) | ㅅㅂ |
| 개새끼 (expletive) | 개ㅅㄲ |
| 미친 (expletive) | 미1친 |

Morphological analyzers also fail on these — they treat decomposed jamo as OOV (Out-of-Vocabulary) tokens, losing the offensive signal entirely.

**Research Question:**  
> *"How can we detect disguised profanity that morphological analyzers cannot even tokenize?"*

---

## 🏗️ Core Design: Dual Tokenization Strategy + Soft Voting Ensemble

The key insight: while `ㅅㅂ` is opaque to morphological analyzers, decomposing it into jamo units (`['ㅅ', 'ㅣ', 'ᴕ', 'ㅂ', 'ㅏ', 'ㄹ']`) preserves the sequence structure, allowing models to learn the abusive pattern.

### Stage 1 Filter — FastText + LSTM (Jamo-level)
- **FastText** handles rare words and typos via subword embeddings.
- Acts as a pre-filter; text not flagged here proceeds to Stage 2.

### Stage 2 Filter — Soft Voting Ensemble
| Model | Accuracy | Strength |
|---|---|---|
| Jamo-level LSTM | 93.22% | Long-range dependency and jamo sequence context |
| Jamo-level CNN | 93.70% | Local N-gram patterns (e.g., characters inserted mid-profanity like `시1발`) |

> Note: A morpheme-level LSTM was also built but excluded from the final ensemble. It incorrectly split strings like `시@발` into `시`, `@`, `발`, misclassifying neutral text as profanity.

---

## 📊 Data Pipeline & Troubleshooting

### Data Sources (56,816 comments after cleaning)

| Community | Volume |
|---|---|
| DCInside | 30,000 |
| FMKorea | 15,000 |
| Inven | 15,000 |
| Nate Pann | 15,000 |

### 🚨 Engineering Challenges

| Challenge | Solution |
|---|---|
| **IP Ban** | Dynamic sleep interval (min 60s × multiple random multipliers) + `User-Agent` header spoofing |
| **Korean Encoding** | `utf-8-sig` encoding for CSV output to prevent garbled characters |
| **Noise & Duplicates** | `emoji` library to strip emojis; regex for comment count artifacts and whitespace; deduplication after merge |

---

## 🖥️ Service Architecture (Django)

| Layer | Detail |
|---|---|
| **Backend** | Django MVT pattern (`urls.py`, `views.py` routing) |
| **Model Serving** | `.h5` LSTM/CNN weights + `.pkl` tokenizers loaded into `views.py` memory for real-time inference |
| **Database** | User input + prediction result (1=profanity, 0=normal) persisted to MySQL for future retraining |
| **Frontend** | HTML/JS web UI with live classification result display |

---

## 📁 Repository Structure

```
SepticFilterNLP/
├── 크롤링/                          # Per-community crawling scripts
├── 전처리+모델링/                    # Tokenization strategy model training notebooks
│   ├── 자모CNN 합친거.ipynb          # Jamo-level CNN
│   ├── 자모LSTM 합친거.ipynb         # Jamo-level LSTM
│   ├── 자모_FastText - LSTM 합친거.ipynb
│   └── 소프트보팅.ipynb              # Final ensemble integration
└── 딥러닝_알고리즘_기반_욕설_감지_필터_.pdf  # Award submission report (37 pages)
```

---

## 👥 Team & Contributions

**Team Name**: 정화조 (Septic Tank), K-Digital Training, Samsung SDS Multicampus

| Member | Role |
|---|---|
| Jeong Deok | Team Lead, data labeling, preprocessing, Django service |
| Kim Juyoung | Data labeling, Django service |
| Park Mihyeon | Modeling, preprocessing, labeling |
| **Baek Jiheon** | **Crawling, database (MySQL) design & pipeline, Django service, labeling** |
| Song Hyeonseung | Modeling, preprocessing, labeling |
| Lee Changyong | Crawling, database design, modeling, labeling |

**Jiheon Baek's Key Contributions:**
- Built multi-community crawler with Selenium + BeautifulSoup (IP ban evasion logic)
- Designed MySQL schema and data ingestion pipeline
- Integrated model serving API with Django backend and HTML/JS frontend

---

## 🔗 Connection to Successor Project

This project directly informed [AntiPhishingNLP](https://github.com/fairyofdata/AntiPhishingNLP) (KISA Cybersecurity Challenge Grand Prize, 2023):

| SepticFilter (2022) | → | AntiPhishingNLP (2023) |
|---|---|---|
| Jamo-level tokenization for disguised profanity | | Dual jamo/morpheme tokenization applied to Att-BiLSTM |
| CNN + LSTM + FastText soft voting | | KoBERT + KoELECTRA + Att-BiLSTM soft voting |
| Community comment crawling pipeline | | KorCCViD cleansing + SMOTE imbalance handling |
| Django-based web service | | Django real-time STT voice phishing server |

The ensemble design philosophy and Korean-specialized tokenization insight carried over directly.

---

## 📄 License

MIT License
