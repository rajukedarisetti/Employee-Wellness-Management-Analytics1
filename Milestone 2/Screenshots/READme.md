# Employee Wellness Management Analytics
## Milestone 2 - NLP Text Preprocessing Pipeline

## 📌 Project Objective

The objective of this milestone is to develop a robust **Natural Language Processing (NLP) Text Preprocessing Pipeline** capable of preparing multilingual text for sentiment and emotion analysis.

The pipeline accepts text from multiple input sources, detects the input language, performs a sequence of preprocessing operations, and produces clean text suitable for machine learning and deep learning models.

---

# NLP Pipeline Overview

The implemented pipeline performs the following operations:

1. Text Input
   - Direct text input
   - Upload `.txt` files
   - Upload `.csv` files

2. Language Detection
   - Automatically detects the language of the input text.
   - Displays the detected language before preprocessing.

3. Unicode Normalization
   - Normalizes Unicode characters into a standard format.

4. Text Cleaning
   - Removes unnecessary whitespace.
   - Removes special characters where applicable.

5. URL Removal
   - Removes web links from the text.

6. Email Removal
   - Removes email addresses.

7. HTML Tag Removal
   - Removes HTML tags from the text.

8. Emoji Extraction
   - Extracts emojis separately.
   - Removes emojis from the text.

9. Punctuation Removal

10. Number Removal

11. Lowercase Conversion
   - Applied for languages where lowercase conversion is meaningful.

12. Tokenization
   - Splits text into individual words/tokens.

13. Stop-word Removal
   - Removes common language-specific stop words.

14. Lemmatization
   - Converts words to their base/root forms.

15. Noise Filtering
   - Removes unwanted or invalid tokens.

16. Final Preprocessed Text
   - Produces clean text ready for sentiment and emotion analysis.

---

# Technologies and Libraries Used

## Programming Language

- Python 3

## Development Environment

- Google Colaboratory (Google Colab)

## Python Libraries

- pandas
- numpy
- regex
- re
- nltk
- spacy
- langdetect
- emoji
- unicodedata
- html
- string

---

# Google Colab Setup Instructions

## Step 1

Open the notebook in **Google Colaboratory**.

## Step 2

Install the required libraries.

```python
!pip install langdetect
!pip install emoji
!pip install spacy
!python -m spacy download en_core_web_sm
```

## Step 3

Download required NLTK resources.

```python
import nltk

nltk.download('punkt')
nltk.download('stopwords')
nltk.download('wordnet')
nltk.download('omw-1.4')
```

## Step 4

Run all notebook cells sequentially.

---

# Preprocessing Workflow

```
Text Input
      │
      ▼
Language Detection
      │
      ▼
Unicode Normalization
      │
      ▼
Text Cleaning
      │
      ▼
URL Removal
      │
      ▼
Email Removal
      │
      ▼
HTML Tag Removal
      │
      ▼
Emoji Extraction
      │
      ▼
Emoji Removal
      │
      ▼
Punctuation Removal
      │
      ▼
Number Removal
      │
      ▼
Lowercase Conversion
      │
      ▼
Tokenization
      │
      ▼
Stop-word Removal
      │
      ▼
Lemmatization
      │
      ▼
Noise Filtering
      │
      ▼
Final Preprocessed Text
```

---

# Sample Input and Output

## Sample Input

```
Hello 😊

Visit https://example.com

Contact me at employee@gmail.com

This is an <b>Employee Wellness</b> Project 2026!
```

---

## Detected Language

```
English
```

---

## Extracted Emojis

```
😊
```

---

## Cleaned Text

```
Hello

Visit

Contact me at

This is an Employee Wellness Project
```

---

## Tokens

```
['hello',
'visit',
'contact',
'me',
'at',
'this',
'is',
'an',
'employee',
'wellness',
'project']
```

---

## Tokens after Stop-word Removal

```
['hello',
'visit',
'contact',
'employee',
'wellness',
'project']
```

---

## Lemmatized Tokens

```
['hello',
'visit',
'contact',
'employee',
'wellness',
'project']
```

---

## Final Preprocessed Text

```
hello visit contact employee wellness project
```

---

# Observations

- The preprocessing pipeline effectively cleans noisy text by removing URLs, email addresses, HTML tags, punctuation, numbers, and unnecessary symbols.
- Language detection enables language-aware preprocessing, improving text quality for multilingual datasets.
- Emoji extraction preserves useful emotional information before removing emojis from the text.
- Tokenization and stop-word removal reduce irrelevant information while retaining meaningful words.
- Lemmatization converts words into their root forms, reducing vocabulary size and improving consistency.
- The final processed text is well-suited for downstream tasks such as sentiment analysis, emotion classification, and other NLP applications.

---

# Repository Structure

```
Employee Wellness Management Analytics/

└── Milestone2/
    ├── NLP_Preprocessing_Pipeline.ipynb
    ├── screenshots/
    └── README.md
```

---

# Future Scope

This preprocessing pipeline serves as the foundation for upcoming milestones, including:

- Sentiment Analysis
- Emotion Classification
- Employee Feedback Analysis
- Mental Wellness Prediction
- AI-powered Employee Wellness Dashboard

---

# Author

**Project:** Employee Wellness Management Analytics

**Milestone:** NLP Text Preprocessing Pipeline

Developed as part of the Infosys Internship Program.