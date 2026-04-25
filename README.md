# A Mental Health Screener (Android)

A speech-driven Android app for mental health self-assessment in Urdu and English.

## Why we built this

Mental health resources in Urdu barely exist. The few apps that do almost always require English fluency which excludes a huge portion of the people who actually need them. I wanted to see if I could build something that could meet people from my community.

The app screens for stress, anxiety, depression, and suicidal ideation using clinically-grounded instruments — DASS-42 for the first three, and a regression model trained on Reddit's r/SuicideWatch for the fourth. It speaks each question out loud, listens to the answer, scores it, and produces a severity band the user can take to a clinician.

It's not a diagnostic tool. It's an early-warning self-check.

## Highlights

- **Voice-driven UX in Urdu and English** — TTS for questions, speech recognition for answers, with separate language handling per assessment track
- **Four assessments** built on validated instruments — Stress, Anxiety, Depression (DASS-42), and a custom suicidal-ideation classifier
- **Five-band severity output** (normal → mild → moderate → severe → extremely severe) mapped from raw Likert scores
- **Per-user history** so users can see how their scores trend over time
- **Therapist directory** so a concerning result can immediately route to a real person

## The model selection (the part I'm proudest of)

For the suicidal ideation classifier I compared nine vectorizer × classifier combinations:

| Vectorizer | × Classifier |
|---|---|
| Count, TF-IDF, Hashing | Multinomial Naïve Bayes, KNN, Logistic Regression |

Hashing + Multinomial NB had the best AUC (0.77). TF-IDF + Multinomial NB scored slightly lower on AUC (0.75 after tuning) but had **better recall on the suicidal class** — meaning it was better at catching true positives.

I chose recall over AUC. In a screening context, a false negative (missing someone genuinely at risk) is far more costly than a false positive. So the deployed model is TF-IDF + Multinomial Naïve Bayes (tuning 2): AUC 0.75, test accuracy 0.70, recall 0.70 on suicidal class.


## How it works

**Stack**
- Android (Java, AndroidX, minSdk 19, targetSdk 30)
- Python (scikit-learn, pandas, NumPy) + Flask for the ML service
- Firebase Auth + Realtime Database for users, history, questions, therapists
- Android `SpeechRecognizer` and `TextToSpeech` for voice I/O
- AnyChart for severity gauges
- Volley for HTTP

**Scoring**
- DASS-42: rule-based, sum of Likert responses mapped to severity bands
- Suicidal ideation: TF-IDF + Multinomial Naïve Bayes classifier served from a Flask endpoint

**Speech-to-score**
- Android Intent recognizes Urdu speech, returns Urdu text
- Mapped to 0–4 Likert scores via keyword matching against DASS-42 response anchors
- e.g. *نہیں / کبھی نہیں* → 0, *انتہائی شدید* → 4

**Datasets**
- DASS-42: 39,776 publicly available instances (online survey data)
- Suicidal ideation classifier: combined Reddit corpus from r/SuicideWatch (suicidal posts) and r/depression (depression but not necessarily suicidal posts), labelled binary

## What the next version would look like

This was built to demo the idea, not to ship. Years later, here's what I'd design differently:

- **Local-first by default** — assessments are personal data; the next version would store them on-device with opt-in encrypted cloud sync, rather than as the default path
- **A clean language layer** — adding a new language should be a config change (questions, TTS voice, recognition target) rather than touching code in three places. The Urdu work taught me how much friction the second language adds when localization isn't a first-class concern
- **One scoring framework** instead of two — the DASS tracks use numeric Likert sums; the suicide track uses an opaque classifier output. Unifying them would make trends, comparisons, and explanations far cleaner
- **Manual input as the default, voice as accessibility** — voice was the novel idea but it's also the most fragile path. Speech-to-score via keyword matching is brittle across dialects; a structured fallback would catch what the recognizer misses
- **Architecture separation** — pull the speech, scoring, and API logic out of the activity layer into a repository + use-case structure, so the same logic could power a web version

## Repository status

Archived. I'm not actively maintaining this code, but I left it public because the engineering decisions. The model selection in particular and the Urdu localization work were worth documenting. If you're a researcher or student building something similar, feel free to reach out. I'm happy to share what I learned.
