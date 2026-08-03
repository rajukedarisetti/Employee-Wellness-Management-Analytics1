# Milestone 3 – Emotion Detection & Journal Analytics

**Project:** Employee Wellness Management Analytics
**Milestone:** 3 – Emotion Detection, Sentiment Scoring & Journal Management
**Program:** Infosys Springboard Internship

---

## 1. Project Objective

Building on the Milestone 2 multilingual NLP preprocessing pipeline, this milestone adds:

- A transformer-based **emotion detection** model integrated with the existing preprocessing pipeline, predicting the dominant emotion for every journal entry along with a confidence score.
- **VADER-based sentiment scoring** (Positive / Negative / Neutral / Compound) for each entry.
- A **Journal module** where an employee can write a daily entry, submit it for analysis, and view the detected emotion, sentiment, and a wellness recommendation.
- **Database persistence** of every analyzed entry (emotion, confidence, compound sentiment score, and journal text) so mood history can be tracked over time.
- A **FastAPI backend** + **Streamlit frontend** exposing all of the above, secured with JWT authentication.

---

## 2. Model Used

| Task | Model | Notes |
|---|---|---|
| Emotion detection | `bhadresh-savani/bert-base-go-emotion` (Hugging Face `transformers`) | Fine-tuned BERT classifier trained on the GoEmotions dataset (28 fine-grained emotion labels). Loaded via `transformers.pipeline("text-classification", ..., top_k=None)` so a score is returned for **every** label, not just the top prediction. |
| Sentiment scoring | VADER (`vaderSentiment`) | Rule/lexicon-based sentiment analyzer, run on the English-translated text. |
| Wellness chat assistant | `Qwen/Qwen2.5-0.5B-Instruct` | A small instruction-tuned causal LM used only for the free-form supportive chatbot (a generation task); it is **not** used for emotion classification. |
| Language detection | `langdetect` | Detects the source language of the raw journal text. |
| Translation | `deep_translator` (Google Translator) | Translates the cleaned/filtered text to English before sentiment and emotion inference, so both models operate on a consistent language. |
| Tokenization / lemmatization | spaCy multilingual pipeline (`xx_sent_ud_sm`) | Sentence segmentation, tokenization, and lemmatization. |
| Stopword removal | `stopwordsiso` | Provides stopword lists for 50+ languages keyed by ISO 639-1 code, so filtering works automatically for whichever language is detected. |

The 28 raw GoEmotions labels are mapped down to **6 application-level emotions** for a simpler, more actionable UI:

```
Happy, Sad, Stress, Angry, Fear, Neutral
```

(e.g. `joy`, `amusement`, `excitement`, `love`, `gratitude`, `optimism`, `relief`, `pride`, `admiration`, `approval`, `caring` → **Happy**; `nervousness`, `embarrassment`, `confusion` → **Stress**; etc.)

---

## 3. Emotion Detection Pipeline

Each journal entry passes through the following stages (implemented in `nlp_pipeline.py`, function `process_employee_feedback`):

1. **Text normalization** – `ftfy.fix_text()` repairs mojibake/encoding issues.
2. **Language detection** – `langdetect` identifies the source language (e.g. Telugu, Hindi, English, French, …).
3. **Cleaning** – removes URLs, emails, @mentions, #hashtags, and emoji (emoji are extracted separately first).
4. **Tokenization & sentence splitting** – via the spaCy multilingual model.
5. **Stopword filtering** – language-specific stopwords removed using `stopwordsiso`.
6. **Translation to English** – via `deep_translator`, so downstream models (VADER + BERT) always see English text.
7. **Lemmatization** – of the translated English text.
8. **Sentiment scoring** – VADER computes Positive/Negative/Neutral/Compound scores on the translated text.
9. **Emotion classification** – the fine-tuned BERT GoEmotions pipeline classifies the translated text; the 28 raw scores are summed into the 6 app-level emotion buckets.
10. **Result assembly** – dominant emotion, confidence, sentiment label, and all intermediate artifacts (cleaned text, tokens, translation, etc.) are returned as a single JSON-serializable result.

A **crisis-keyword check** also runs in the wellness chatbot path (separate from emotion detection): if a message contains language indicating self-harm risk, a fixed, resource-pointing safety message is returned instead of a model-generated reply.

---

## 4. Confidence Score Calculation

The BERT emotion pipeline returns a probability for each of the 28 GoEmotions labels. These are:

1. Mapped to one of the 6 app-level emotions (summing scores of all raw labels that map to the same bucket).
2. Normalized so the 6 bucket scores sum to 1.
3. The bucket with the highest normalized score is chosen as the **final predicted emotion**.
4. Its normalized score (a value between 0 and 1) is reported as the **confidence score** for that prediction.

```python
app_scores = {label: 0.0 for label in EMOTION_LABELS}
for pred in raw_predictions:
    app_label = GOEMOTIONS_TO_APP_LABEL.get(pred["label"].lower(), "Neutral")
    app_scores[app_label] += pred["score"]

total = sum(app_scores.values()) or 1.0
app_scores = {label: round(score / total, 4) for label, score in app_scores.items()}

final_emotion = max(app_scores, key=app_scores.get)
confidence = app_scores[final_emotion]
```

The confidence score is displayed to the user as a percentage (e.g. `Confidence: 82%`) and stored alongside the predicted emotion in the database.

---

## 5. Sentiment Analysis

VADER (`SentimentIntensityAnalyzer`) is run on the **translated English text** and returns four scores:

| Score | Meaning |
|---|---|
| Positive | Proportion of text classified as positive |
| Negative | Proportion of text classified as negative |
| Neutral | Proportion of text classified as neutral |
| Compound | A single normalized score in `[-1, 1]` summarizing overall sentiment |

The compound score is thresholded into a final label:

```python
if compound_score >= 0.05:
    final_sentiment = "Positive 😊"
elif compound_score <= -0.05:
    final_sentiment = "Negative 😔"
else:
    final_sentiment = "Neutral 😐"
```

All four scores are shown to the user; only the **compound score** is persisted to the database (mapped onto a 5-point mood scale — see schema below — for calendar and dashboard visualizations).

---

## 6. Journal Module & Frontend

The Streamlit frontend (`app.py`) exposes a **Journal** page (alongside Home, Analyze Text, Wellness Chat, and Dashboard) where an employee can:

- Type a free-text daily journal entry and submit it for analysis (`/analyze-text`), or upload a `.csv`/`.txt` file of feedback (`/analyze`).
- View, per entry:
  - **Journal Text**
  - **Detected Language**
  - **Predicted Emotion** (with emoji) and **Confidence Score**
  - **Sentiment Scores** (Positive / Negative / Neutral / Compound) and final sentiment label, plus a bar chart of the per-emotion score distribution
  - **Wellness Recommendation** — surfaced through the built-in **Wellness Chat** assistant (Qwen-based), which offers short, supportive, non-clinical coping suggestions (e.g. breathing exercises, taking a break, talking to a trusted colleague/manager) based on how the employee is feeling, and escalates to crisis-helpline information if self-harm language is detected.
- Browse **past entries** in an expandable history list (sentiment, emotion, confidence, timestamp, and full text).

A separate **Home** page lets employees log a quick mood via emoji picker (independent of NLP), view a mood calendar, current streak, and an overall wellness score. A **Reports** page (manager role) aggregates mood logs across all employees.

---

## 7. Database Schema

PostgreSQL (hosted on Neon/Supabase), accessed via `psycopg2`. Tables created/maintained by `db.py`:

**`users`**
| Column | Type | Notes |
|---|---|---|
| id | SERIAL PK | |
| username | VARCHAR(50) UNIQUE | |
| email | VARCHAR(255) UNIQUE | |
| password_hash | VARCHAR(255) | bcrypt hash |
| is_verified | BOOLEAN | OTP email verification flag |
| role | VARCHAR(20) | `employee` or manager role, default `employee` |

**`otp_codes`**
| Column | Type | Notes |
|---|---|---|
| id | SERIAL PK | |
| email | VARCHAR(255) | |
| code | VARCHAR(6) | One-time verification code |
| purpose | VARCHAR(20) | `signup` / `reset` |
| expires_at | TIMESTAMP | 10-minute expiry |
| used | BOOLEAN | |

**`mood_logs`** *(core table for this milestone)*
| Column | Type | Notes |
|---|---|---|
| id | SERIAL PK | |
| user_id | INTEGER FK → users(id) | `ON DELETE CASCADE` |
| mood_date | DATE | defaults to `CURRENT_DATE` |
| sentiment | VARCHAR(20) | Mapped to the 5-point mood scale: Amazing / Happy / Normal / Sad / Angry |
| emotion | VARCHAR(30) | Predicted dominant emotion (e.g. `Happy 😊`) |
| compound_score | REAL | VADER compound sentiment score |
| confidence | REAL | Emotion model's confidence (0–1) |
| journal_text | TEXT | Full entry text |
| source | VARCHAR(10) | `manual` (emoji picker) or `nlp` (journal/file analysis) |
| created_at | TIMESTAMP | defaults to `NOW()` |

An index `idx_mood_logs_user_date` on `(user_id, mood_date)` supports fast calendar/history lookups.

---

## 8. API Endpoints

FastAPI backend (`backend.py`), all routes except `/health` require a `Authorization: Bearer <JWT>` header:

| Method | Endpoint | Description |
|---|---|---|
| GET | `/health` | Health check |
| POST | `/upload` | Upload a `.csv`/`.txt` file and preview its rows/columns (no NLP run yet) |
| POST | `/analyze` | Run the full NLP pipeline on an uploaded `.csv`/`.txt` file (optionally targeting a specific CSV column) |
| POST | `/analyze-text` | Run the full NLP pipeline on raw text typed into the Journal tab |
| POST | `/chat` | Wellness chatbot — sends a message + recent conversation history, returns a supportive reply (or a crisis-resource message if flagged) |

Authentication (signup, login, OTP verification, password reset) is handled directly inside the Streamlit app via `auth.py` (bcrypt password hashing + PyJWT tokens) rather than as separate REST endpoints, since the frontend and business logic run in the same Colab process.

Interactive API documentation is available at `/docs` (Swagger UI) and `/redoc` once the backend is running.

---

## 9. Sample Input & Output

**Input** (Journal entry, Hindi):
> "आज ऑफिस में बहुत काम था, थोड़ा थका हुआ महसूस कर रहा हूँ लेकिन टीम का साथ अच्छा लगा।"

**Output:**
```json
{
  "detected_language": "Hindi",
  "translated_text": "There was a lot of work in the office today, feeling a little tired but enjoyed the team's company.",
  "sentiment_scores": {"neg": 0.111, "neu": 0.63, "pos": 0.259, "compound": 0.4019},
  "final_sentiment": "Positive 😊",
  "emotion_scores": {"Happy": 0.42, "Sad": 0.18, "Stress": 0.21, "Angry": 0.03, "Fear": 0.05, "Neutral": 0.11},
  "final_emotion": "Happy 😊",
  "emotion_confidence": 0.42
}
```

*(Illustrative values — exact scores will vary by model run.)*

---

## 10. Observations

- Running translation before sentiment/emotion inference standardizes downstream model input, but any translation errors on colloquial or code-mixed text can shift both sentiment and emotion results — this is a known limitation of the pipeline.
- Mapping the 28-label GoEmotions taxonomy down to 6 app-level buckets makes results easier to act on but does lose some nuance (e.g. "disappointment" and "grief" both collapse into "Sad").
- VADER performs sentence-level lexicon matching and works well on the translated English text, but was originally tuned for social-media-style English, so nuanced workplace language may not always score as expected.
- Storing only the compound sentiment score (rather than all four VADER scores) keeps the schema simple for calendar/dashboard visualizations while still surfacing the full breakdown to the user at analysis time.
- The Wellness Chat model (Qwen2.5-0.5B-Instruct) is intentionally small for fast, low-cost inference in Colab; a crisis-keyword safety net bypasses the model entirely for high-risk messages rather than relying on the model to handle them safely.

---

## 11. Repository Structure

```
Employee Wellness Management Analytics/
└── Milestone3/
    ├── Emotion_Detection.ipynb
    ├── backend/
    ├── frontend/
    ├── screenshots/
    └── README.md
```

## 12. Submission

- **Milestone:** 3 – Emotion Detection & Journal Analytics
- **Deadline:** 02 August 2026, 09:00 PM
- **Submitted by:** _[Your Name]_
- **Repository Link:** _[Add your GitHub repo link here before submitting]_
