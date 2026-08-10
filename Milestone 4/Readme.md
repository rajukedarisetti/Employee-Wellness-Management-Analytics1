# 🧠 MoodMentor -- AI-Powered Emotional Wellness Assistant

MoodMentor is an **AI-powered emotional wellness application** developed
as part of the **Employee Wellness Management & Analytics** project.

The application helps employees understand, track, and reflect on their
emotional well-being through daily mood tracking, journal analysis,
sentiment analysis, emotion detection, wellness recommendations, and an
AI-powered Wellness Chat assistant.

The project combines **Natural Language Processing (NLP), Machine
Learning, Transformer-based Emotion Detection, Generative AI,
Authentication, PostgreSQL, Data Visualization, and Report Generation**
into one integrated application.

------------------------------------------------------------------------

## 📌 Project Objective

The main objective of MoodMentor is to provide an easy-to-use platform
that can:

-   Track an employee's daily mood.
-   Analyze emotions from journal text.
-   Detect the language of user input.
-   Clean and preprocess multilingual text.
-   Translate non-English text into English.
-   Perform sentiment analysis.
-   Detect emotions using a BERT-based model.
-   Generate personalized wellness recommendations.
-   Provide an AI-powered wellness chat assistant.
-   Store mood and journal information in PostgreSQL.
-   Display emotional trends using dashboards and charts.
-   Generate PDF and CSV wellness reports.
-   Provide managers with an overview of employee wellness data.

------------------------------------------------------------------------

# ✨ Main Features

## 1. 🔐 User Authentication

MoodMentor provides a complete user authentication system.

Users can:

-   Create an account.
-   Select an Employee or Manager role.
-   Login securely.
-   Verify their email using OTP.
-   Reset their password using OTP.
-   Logout securely.

### Security

-   Passwords are protected using **bcrypt hashing**.
-   Authentication uses **JWT tokens**.
-   User and authentication information is stored in PostgreSQL.
-   Sensitive information such as passwords, API keys, database
    credentials, and JWT secrets should not be committed to the
    repository.

------------------------------------------------------------------------

## 2. 😊 Daily Mood Tracking

Employees can manually record their current mood.

The application supports six mood categories:

-   😊 Happy
-   😐 Neutral
-   😢 Sad
-   😫 Stress
-   😠 Angry
-   😨 Fear

Each mood entry is associated with the user and the time/date of the
entry and is stored in PostgreSQL.

------------------------------------------------------------------------

## 3. 📅 Mood Calendar

The Home page provides a calendar-based view of mood history.

Each recorded day can display:

-   Date
-   Mood
-   Emoji
-   Mood indicator/color
-   Recorded time

Users can navigate through previous and upcoming months to review their
mood history.

------------------------------------------------------------------------

# 📝 4. Journal Analysis

Users can enter a journal entry describing how they are feeling.

### Example

``` text
I am feeling stressed because I have a lot of work today.
```

The journal text is sent to the FastAPI backend and processed through
the NLP pipeline.

### NLP Workflow

``` text
User Text
    ↓
Text Normalization
    ↓
Language Detection
    ↓
Text Cleaning
    ↓
Tokenization
    ↓
Stopword Removal
    ↓
Translation to English
    ↓
Lemmatization
    ↓
Sentiment Analysis
    ↓
Emotion Detection
    ↓
Recommendation
    ↓
Database / Dashboard
```

------------------------------------------------------------------------

# 🌐 5. Multilingual NLP Pipeline

MoodMentor is designed to support multilingual journal entries.

The pipeline can handle languages including:

-   English
-   Hindi
-   Marathi
-   Gujarati
-   Telugu
-   Kannada
-   Tamil
-   Malayalam
-   Bengali
-   French
-   German
-   Spanish
-   Portuguese
-   Arabic
-   Chinese
-   Japanese
-   Korean
-   Russian

The language is detected first. Non-English text is translated into
English before the main sentiment and emotion analysis.

The main NLP workflow is implemented in:

``` text
nlp_pipeline.py
```

------------------------------------------------------------------------

# 🧹 6. Text Preprocessing

The NLP pipeline performs multiple preprocessing operations to prepare
raw user text for analysis.

### Text Normalization

The **ftfy** library is used to fix problematic text encoding and
normalize text.

### URL Removal

URLs such as:

``` text
https://example.com
```

are removed from the analysis text.

### Email Removal

Email addresses are removed to reduce unnecessary personal information
from the NLP input.

### Mentions and Hashtags

Social-media-style mentions and hashtags are cleaned before analysis.

### Emoji Processing

Emojis are detected and can be preserved as separate information while
being removed from the text used by the NLP models.

### Tokenization

Text is divided into individual tokens for further processing.

### Stopword Removal

Language-specific stopwords are removed using **stopwordsiso**.

### Translation

Non-English text is translated into English using **GoogleTranslator**
through the `deep-translator` package.

### Lemmatization

The translated English text is lemmatized using **spaCy**.

### Final Pipeline

``` text
normalize
    ↓
detect language
    ↓
clean
    ↓
tokenize
    ↓
stopword filtering
    ↓
translate to English
    ↓
lemmatize
    ↓
sentiment
    ↓
emotion
```

------------------------------------------------------------------------

# 😊 7. Sentiment Analysis

MoodMentor uses **VADER Sentiment Analysis** to determine the overall
sentiment of journal text.

The three sentiment categories are:

-   Positive
-   Negative
-   Neutral

VADER produces a compound sentiment score approximately between:

``` text
-1  → Negative
 0  → Neutral
+1  → Positive
```

The application uses the following thresholds:

``` text
Compound >= 0.05
        ↓
    Positive

Compound <= -0.05
        ↓
    Negative

Otherwise
        ↓
    Neutral
```

The sentiment result and score can be displayed in the Journal and
Dashboard sections.

------------------------------------------------------------------------

# 😃 8. Emotion Detection

MoodMentor uses the following BERT-based transformer model:

``` text
bhadresh-savani/bert-base-go-emotion
```

The model produces emotion predictions and scores from journal text.

The application maps the model outputs into six application-level
emotion categories:

``` text
Happy
Sad
Stress
Angry
Fear
Neutral
```

### Example Mapping

``` text
Joy
Love
Excitement
Gratitude
Optimism
       ↓
    Happy
```

Other related emotion outputs are grouped into the application's six
categories. For example, sadness-related emotions can be mapped to
**Sad**, nervousness or confusion can contribute to **Stress**, and
anger-related emotions can be mapped to **Angry**.

The final application emotion is selected using the highest aggregated
emotion score.

------------------------------------------------------------------------

# 🤖 9. AI Wellness Chat

MoodMentor includes an AI-powered Wellness Chat feature.

The conversational assistant uses:

``` text
Qwen/Qwen2.5-0.5B-Instruct
```

The model is used locally to generate supportive conversational
responses.

The Wellness Chat is designed to provide:

-   Supportive responses
-   General coping suggestions
-   Encouragement
-   Non-judgmental conversation
-   General wellness guidance

The system prompt prevents the assistant from presenting itself as a
doctor or therapist.

The chatbot also checks for crisis-related keywords before generating an
AI response.

> The Wellness Chat is intended for general wellness support and is not
> a replacement for professional medical or psychological care.

------------------------------------------------------------------------

# 💡 10. Wellness Recommendation System

After sentiment and emotion analysis, MoodMentor provides
recommendations based on the detected emotional state.

### Example Recommendation Flow

``` text
Happy
   ↓
Positive and encouraging recommendation

Sad
   ↓
Journaling, breathing, or social-support recommendation

Stress
   ↓
Short break, breathing, or workload-management recommendation

Angry
   ↓
Pause, step away, and reflection recommendation

Fear
   ↓
Grounding, breathing, and support recommendation
```

Recommendations can also consider the model's confidence.

### Confidence-Based Recommendations

``` text
Low Confidence
      ↓
Light / General Suggestion

Medium Confidence
      ↓
Matched Coping Suggestion

High Confidence
      ↓
More Structured Support Suggestion
```

This approach helps avoid giving overly specific recommendations when
the emotion prediction is uncertain.

------------------------------------------------------------------------

# 📊 11. Employee Dashboard

Employees have access to a personal wellness dashboard.

The dashboard can display:

-   Current mood
-   Mood distribution
-   Mood trend over time
-   Detected emotions
-   Sentiment distribution
-   Recent activity
-   Mood calendar
-   Historical analysis
-   Date filtering
-   Mood filtering
-   Source filtering
-   Journal text search

The dashboard uses data retrieved from the backend and PostgreSQL
database.

------------------------------------------------------------------------

# 📈 12. Mood and Emotion Visualization

MoodMentor uses charts to make emotional data easier to understand.

### Mood Distribution

A donut chart displays the distribution of recorded moods.

### Mood Trend

A line chart shows how the user's mood changes over time.

### Emotion Distribution

A bar chart displays the frequency of emotions detected from journal
entries.

### Sentiment Distribution

A chart displays the number or proportion of:

``` text
Positive
Negative
Neutral
```

sentiment results.

These visualizations help employees identify emotional patterns over
time.

------------------------------------------------------------------------

# 📄 13. Report Generation

MoodMentor provides export functionality for wellness analytics.

## PDF Report

The PDF report can contain:

-   Username
-   Selected date range
-   Mood summary
-   Wellness recommendation
-   Mood entries
-   Detected emotion
-   Confidence score
-   Data source

## CSV Report

The CSV export can contain:

``` text
Date
Time
Mood
Emotion
Confidence
Source
Journal Text
```

These reports allow users to retain and review their wellness
information.

------------------------------------------------------------------------

# 👨‍💼 14. Manager Dashboard

MoodMentor provides a separate Manager role.

Managers can view employee wellness information such as:

-   Employee name
-   Employee email
-   Latest mood
-   Date
-   Time
-   Mood
-   Emotion

Managers can:

-   Search employees.
-   Filter employee records by mood.
-   View recent employee wellness information.
-   Export employee wellness information as CSV.
-   Review team-level mood trends.

The manager dashboard also provides a **team mood trend for the last 30
days**.

------------------------------------------------------------------------

# 🗄️ 15. PostgreSQL Database

MoodMentor uses **PostgreSQL** for persistent data storage.

The database stores user, authentication, mood, and analysis
information.

## Users Table

Stores information such as:

``` text
User ID
Username
Email
Password Hash
Verification Status
Role
```

## OTP Codes Table

Stores information such as:

``` text
Email
OTP Code
Purpose
Expiration Time
Used Status
```

## Mood Logs Table

Stores information such as:

``` text
User ID
Mood Date
Sentiment
Emotion
Compound Score
Confidence
Journal Text
Source
Creation Time
```

The database allows the application to maintain user history and
retrieve information for dashboards and reports.

------------------------------------------------------------------------

# ⚙️ 16. Backend

The backend is developed using **FastAPI**.

### Main responsibilities

-   Handle frontend API requests.
-   Manage authentication.
-   Process journal text.
-   Handle CSV/TXT uploads.
-   Connect the NLP pipeline to the application.
-   Perform sentiment and emotion analysis.
-   Generate recommendations.
-   Provide Wellness Chat functionality.
-   Store and retrieve data from PostgreSQL.
-   Validate requests.
-   Handle API errors.

### Main APIs

  Endpoint          Method   Purpose
  ----------------- -------- ---------------------------
  `/health`         GET      Backend health check
  `/upload`         POST     Upload and preview files
  `/analyze`        POST     Analyze uploaded files
  `/analyze-text`   POST     Analyze direct text input
  `/chat`           POST     Wellness Chat

------------------------------------------------------------------------

# 🧠 17. Models and NLP Components

  ----------------------------------------------------------------------------
  Component                                Purpose
  ---------------------------------------- -----------------------------------
  `langdetect`                             Detects the language of journal
                                           text

  `deep-translator` / Google Translate     Translates non-English text to
                                           English

  `spaCy`                                  Sentence splitting, tokenization,
                                           and lemmatization

  `VADER`                                  Positive, Negative, Neutral
                                           sentiment analysis

  `bhadresh-savani/bert-base-go-emotion`   Transformer-based emotion
                                           classification

  `Qwen/Qwen2.5-0.5B-Instruct`             Local Wellness Chat assistant

  `ftfy`                                   Fixes problematic text encoding

  `stopwordsiso`                           Removes language-specific stopwords
  ----------------------------------------------------------------------------

------------------------------------------------------------------------

# 🔄 18. Complete Application Architecture

``` text
                    ┌─────────────────┐
                    │      User       │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │    Frontend     │
                    │ MoodMentor UI   │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │     FastAPI     │
                    │     Backend     │
                    └────────┬────────┘
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
          ▼                  ▼                  ▼
   ┌────────────┐     ┌─────────────┐    ┌────────────┐
   │ NLP / ML   │     │   JWT Auth  │    │ Wellness   │
   │ Pipeline   │     │             │    │    Chat    │
   └─────┬──────┘     └─────────────┘    └─────┬──────┘
         │                                      │
         ▼                                      ▼
 ┌───────────────┐                    ┌─────────────────┐
 │ Sentiment +   │                    │ Qwen2.5-0.5B    │
 │ Emotion Model │                    │ Instruct Local  │
 └───────┬───────┘                    └─────────────────┘
         │
         ▼
 ┌─────────────────┐
 │ Recommendation  │
 │     System      │
 └────────┬────────┘
          │
          ▼
 ┌─────────────────┐
 │   PostgreSQL    │
 │    Database     │
 └────────┬────────┘
          │
          ▼
 ┌─────────────────┐
 │ Dashboard /     │
 │ Reports / CSV   │
 │      / PDF      │
 └─────────────────┘
```

------------------------------------------------------------------------

# 🛠️ 19. Technology Stack

  Layer             Technology
  ----------------- -----------------------------------------
  Frontend          Streamlit / Web UI
  Backend           FastAPI
  Language          Python
  NLP               spaCy, langdetect, ftfy, stopwordsiso
  Translation       Google Translate / deep-translator
  Sentiment         VADER
  Emotion           BERT / GoEmotions
  Generative AI     Qwen2.5-0.5B-Instruct
  Authentication    JWT + bcrypt
  Database          PostgreSQL
  Data Processing   Pandas
  Visualization     Charts / Dashboard components
  Reports           PDF and CSV
  Version Control   Git / GitHub
  Development       Google Colab / Local Python Environment

------------------------------------------------------------------------
