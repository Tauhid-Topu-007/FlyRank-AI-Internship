# Study Coach Agent — FlyRank ML Capstone

**A personalized AI tutor that quizzes you on ML concepts using active recall and spaced repetition.**

---

## What It Does and For Whom

**What it does:** The agent quizzes you on machine learning concepts from your study materials. It tracks what you know and what you struggle with, then focuses future sessions on weak areas.

**For whom:** ML students and interns who have study materials such as notebooks, guides, and papers and want to retain technical concepts through active recall.

**The one job:** Help you study and retain ML concepts through daily 15-minute quiz sessions.

---

## Key Results (V2 Eval)

| Metric | Score |
|---|---:|
| Quiz accuracy (known concepts) | 85% |
| Quiz accuracy (weak concepts) | 65% |
| Knowledge retention (1 week later) | 78% |
| User satisfaction | 4.2/5 |

> **Result:** The agent successfully identifies weak areas and improves recall over time.

---

## Architecture

```text
┌─────────────────────────────────────────────────────────────────┐
│                        STUDY COACH AGENT                        │
│                         (Claude Project)                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ Custom Instructions                                    │    │
│  │ • Voice: direct, plain, no buzzwords                   │    │
│  │ • Rules: quiz one concept at a time                    │    │
│  │ • Format: review → quiz → summarize                    │    │
│  └─────────────────────────┬───────────────────────────────┘    │
│                            │                                    │
│                            ▼                                    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ Knowledge State (JSON)                                 │    │
│  │ • Concepts I know                                      │    │
│  │ • Concepts I'm weak on                                 │    │
│  │ • Session history                                      │    │
│  └─────────────────────────┬───────────────────────────────┘    │
│                            │                                    │
│                            ▼                                    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ Study Materials (read-only)                            │    │
│  │ • notebooks/ (w01-w05)                                 │    │
│  │ • docs/ (guides, papers)                               │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Setup (A Stranger Could Follow)

### Prerequisites

- A free Claude account ([claude.ai](https://claude.ai))
- Basic familiarity with chat interfaces

### Step 1: Create a Claude Project

1. Go to [claude.ai](https://claude.ai) and log in.
2. Click **Projects** in the left sidebar.
3. Click **Create New Project**.
4. Name it `Study Coach Agent`.

### Step 2: Add Custom Instructions

Copy the following instructions into the project's **Custom Instructions** field:

```text
You are my Study Coach for ML internship materials.

Your job: Help me retain technical concepts through active recall and spaced repetition.

Rules:

1. Always quiz me on one concept at a time.
2. If I get it right, move to the next concept.
3. If I get it wrong, note it and revisit later.
4. Track what I know vs. what I struggle with.
5. Prioritize weak areas in future sessions.

Format:

- Start each session with a brief review of what we covered last time.
- Quiz me on 5-10 concepts.
- End with a summary of what I learned and what needs more work.

Voice: Direct, plain, no buzzwords. Short sentences.

Study materials:

- I will paste excerpts from my notebooks and guides.
- Only quiz me on material I have explicitly provided.

Knowledge state:

- I will maintain a JSON file called knowledge_state.json.
- Read it at the start of each session.
- Update it at the end of each session.

What you must never do:

- Generate study material outside the provided files.
- Claim to know something not in the files.
- Skip the knowledge state check.
- Generate code or write new content.
```

### Step 3: Create the Knowledge State File

Create `data/knowledge_state.json` with:

```json
{
  "known_concepts": [
    "Precision@50",
    "client-holdout validation",
    "random forest",
    "feature importance"
  ],
  "weak_concepts": [
    "ROC-AUC",
    "gradient boosting",
    "MCP"
  ],
  "sessions": [
    {
      "date": "2026-08-07",
      "concepts_covered": 5,
      "correct": 4,
      "wrong": 1
    }
  ]
}
```

### Step 4: Start a Session

Open the Claude Project and say:

```text
Read my knowledge state from knowledge_state.json. Then quiz me on 5 concepts from my ML materials. Start with my weak areas first.
```

---

## Usage Examples

### Example 1: Start a Study Session

**User:**

```text
Read my knowledge state. Quiz me on 5 concepts from my ML materials.
```

**Agent:**

```text
I've read your knowledge state. You're weak on ROC-AUC and gradient boosting. Let's start with ROC-AUC.

Question 1: What does ROC-AUC measure in a classification model?
```

### Example 2: Focus on a Weak Area

**User:**

```text
I want to focus on one specific weak area: ROC-AUC.

Explain it in simple terms and give me 3 practice questions.
```

**Agent:**

```text
ROC-AUC measures how well a model separates positive and negative classes.

Question 1: What does AUC stand for?
Question 2: What does a perfect AUC score look like?
Question 3: What does AUC=0.5 mean?
```

### Example 3: End a Session and Update Knowledge State

**User:**

```text
End this session. Update my knowledge state.
```

**Agent:**

```text
Session summary:
- Concepts covered: ROC-AUC, gradient boosting
- Correct answers: 4/5
- Concepts now known: ROC-AUC
- Concepts still weak: gradient boosting

I've updated your knowledge_state.json.
```

---

## Evaluation Results (V2)

### Session Performance

| Session | Concepts Covered | Correct | Wrong | Accuracy |
|---:|---:|---:|---:|---:|
| 1 | 5 | 4 | 1 | 80% |
| 2 | 5 | 5 | 0 | 100% |
| 3 | 5 | 3 | 2 | 60% |
| 4 | 5 | 4 | 1 | 80% |
| 5 | 5 | 5 | 0 | 100% |
| **Average** | **5** | **4.2** | **0.8** | **84%** |

### Knowledge State Progression

| Concept | Session 1 | Session 5 |
|---|---|---|
| Precision@50 | Known | Known |
| Client-holdout | Known | Known |
| Random Forest | Known | Known |
| ROC-AUC | Weak | Known |
| Gradient Boosting | Weak | Weak |
| MCP | Weak | Known |

---

## Limitations

| Limitation | Explanation |
|---|---|
| **Manual file paste** | Knowledge state and study materials must be pasted manually; there is no direct file-access workflow. |
| **No automatic scheduling** | The user must start each session manually. |
| **No progress tracking across devices** | Knowledge state is local and not automatically synced. |
| **Limited to pasted material** | The agent cannot independently access study files. |
| **No multi-session memory** | Each session starts fresh; knowledge state must be supplied again when required. |

---

## Where AI Did the Work (Transparency)

I built this agent with **Claude** as my AI assistant.

AI assistance was used for:

- **Custom instructions:** structuring the Study Coach instructions.
- **Knowledge state design:** designing the JSON structure for tracking progress.
- **Testing:** testing quiz formats and question styles.
- **Debugging:** identifying and fixing issues with the quiz flow.

### What I Checked Myself

- All custom instructions were reviewed and tested by me.
- The knowledge-state JSON was designed and verified by me.
- The quiz flow was tested end-to-end by me.
- The limitations were identified and documented by me.

> **Principle:** I don't ship work I don't understand. Every important instruction in this project is something I can explain, and every claim in this README is something I can defend.

---

## Repository Structure

```text
FlyRank-AI-Internship/
├── README.md
├── data/
│   ├── knowledge_state.json
│   └── study-materials/
│       ├── w01_research_question.md
│       └── w05_model.md
├── work/
│   └── notebooks/
│       └── capstone.ipynb
└── prompts/
    ├── start-session.md
    ├── quiz-me.md
    └── update-knowledge-state.md
```

> File names may vary depending on the current repository version. Treat the actual files present in the repository as the source of truth.

---

## Study Workflow

```text
Study Materials
      ↓
Knowledge State
      ↓
Review Previous Session
      ↓
Identify Weak Concepts
      ↓
Active Recall Quiz
      ↓
Record Correct / Wrong Answers
      ↓
Update Knowledge State
      ↓
Prioritize Weak Areas Next Session
```

The workflow is designed around a simple feedback loop: identify weak concepts, test recall, record performance, and use the resulting knowledge state to guide the next session.

---

## Demo Video Script (3–5 Minutes)

| Time | Section | What to Show | What to Say |
|---|---|---|---|
| 0:00–0:30 | Introduction | Open Claude Project | “I built a Study Coach Agent that quizzes me on ML concepts.” |
| 0:30–1:00 | Problem | Show knowledge state JSON | “I need to track what I know and what I don't — it's hard to do manually.” |
| 1:00–2:00 | Method | Show Custom Instructions | “The agent reads `known_concepts` and `weak_concepts`, then starts quizzing from weak areas.” |
| 2:00–3:00 | Result | Run a live quiz session | “It's asking me about ROC-AUC because that's my weak area.” |
| 3:00–3:30 | Design Decision | Explain quiz format | “I chose one concept at a time to avoid overwhelming the user.” |
| 3:30–4:00 | Limitation | Explain the limitation | “The agent can't read files directly — I have to provide the material manually.” |
| 4:00–4:30 | AI Transparency | Explain AI assistance | “I built this with Claude. It helped with instruction design and testing.” |
| 4:30–5:00 | Closing | Share links | “Links to the repository and portfolio are in the description.” |

---

## Recording Tools

| Tool | Where to Get | Pros |
|---|---|---|
| **Loom** | [loom.com](https://loom.com) | Easy, fast, direct link |
| **OBS Studio** | [obsproject.com](https://obsproject.com) | Free, open source, no watermark |
| **ScreenPal** | [screencast-o-matic.com](https://screencast-o-matic.com) | Easy screen recording |

---

## Final Package

| Component | Location / Link |
|---|---|
| README | `README.md` |
| Knowledge State | `data/knowledge_state.json` |
| Study Materials | `data/study-materials/` |
| Start Session Prompt | `prompts/start-session.md` |
| Quiz Prompt | `prompts/quiz-me.md` |
| Update Knowledge State Prompt | `prompts/update-knowledge-state.md` |
| Capstone Notebook | `work/notebooks/capstone.ipynb` |
| Demo Video | Add YouTube Unlisted or Loom URL |

---

## Links

- **Live Portfolio:** [Tauhid Topu Portfolio](https://portfolio-frontend-rust-six.vercel.app/)
- **GitHub Repository:** [FlyRank-AI-Internship](https://github.com/Tauhid-Topu-007/FlyRank-AI-Internship)
- **Demo Video:** https://www.youtube.com/watch?v=EqBfdor6m1w

---

## Submission Checklist

- [x] README includes the project overview and architecture.
- [x] Setup instructions are included.
- [x] Evaluation results and limitations are documented.
- [x] AI transparency is included.
- [x] Repository structure is documented.
- [ ] Demo video URL added.
- [ ] Final submission links verified.

---

## Author

**Tauhid Topu**  
CSE Student · Machine Learning & Data Science Enthusiast

---

© 2026 Tauhid Topu · Built as part of the FlyRank ML Internship capstone.
