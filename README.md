# Are AI-led interviews comparable to human-led interviews?

This repository contains the data, code, and documentation for a study comparing AI-led and human-led semi-structured interviews on the topic of university students' academic help-seeking behavior. N=40 university students were randomly assigned to either a human-led or AI-led interview, both conducted via a text-based chat interface. The AI interview system used OpenHermes 2.5 (a fine-tuned version of Mistral 7B) hosted on GDPR-compliant servers, following the same interview guide as the human interviewers.

## Repository Contents

| File / Folder                          | Description                                                                 |
| -------------------------------------- | --------------------------------------------------------------------------- |
| `analysis.ipynb`                       | Jupyter notebook containing all analysis code                               |
| `requirements.txt`                     | Python dependencies                                                         |
| `agg.csv`                              | Aggregated behavioral metrics per interview (41 conversations)              |
| `relevance_and_specificity_scores.csv` | Aggregated coded quality scores per interview                               |
| `interview_guide.json`                 | Full interview guide including framing, questions, probes, and intro/outro  |
| `prompts/`                             | Jinja prompt templates used by the AI interview system                      |

## Data

### agg.csv

Aggregated behavioral metrics per conversation. Raw interview transcripts are not shared to protect participant confidentiality (see Data Availability).

| Column                                    | Description                                                            |
| ----------------------------------------- | ---------------------------------------------------------------------- |
| conversation_id                           | A unique ID per conversation                                           |
| Interviewer                               | If the interviewer was either `Human` or `AI`                          |
| total_duration(min)                       | The total duration of the interview                                    |
| total_duration(min)\_by_user              | The total duration of the interview spent by the user                  |
| avg_response_length                       | Average response length by the user                                    |
| avg_response_duration_by_user(sec)        | Average response duration by the user in seconds                       |
| avg_response_duration_by_interviewer(sec) | Average response duration by the interviewer in seconds                |
| total_character                           | Total number of characters for the entire conversation                 |
| total_character_by_user                   | Total number of characters for the entire conversation sent by the user|
| total_words                               | Total number of words for the entire conversation                      |
| total_words_by_user                       | Total number of words for the entire conversation sent by the user     |
| question_count                            | Number of questions asked during the interview                         |

### relevance_and_specificity_scores.csv

Aggregated coded quality scores per conversation. Scores represent averages across all answers coded within an interview.

| Column          | Description                                    |
| --------------- | ---------------------------------------------- |
| conversation_id | A unique ID per conversation                   |
| interviewer     | If the interviewer was either `Human` or `AI`  |
| relevance       | Average relevance score for the interview (1–5; 1 = very irrelevant, 5 = very relevant)  |
| specificity     | Average specificity score for the interview (1–5; 1 = very vague, 5 = very specific)     |

## Analysis Code

All analysis code can be found in [analysis.ipynb](./analysis.ipynb). Dependencies can be found in `requirements.txt`.

## AI Interview System — Prompts

The AI interviewer was implemented using two LLM agents working in tandem. Prompt templates are stored in the `prompts/` directory as Jinja (`.jinja`) files with named variable slots.

### Probing Agent

The probing agent is responsible for asking interview questions and follow-up probes. It receives the current question context, suggested probes from the interview guide, and the conversation transcript so far, and generates a single follow-up question.

**System prompt** (`prompts/probing_agent_system_prompt.jinja`):
> Establishes the agent as an experienced qualitative social science interviewer whose sole task is to ask relevant follow-up probes one question at a time.

**Probing prompt** (`prompts/probing_agent_probing_prompt.jinja`):

Template variables:
- `question_context` — the current main question and its associated metadata from the interview guide
- `interview_transcript` — the conversation transcript up to the current point
- `probes` — the list of suggested follow-up probes from the interview guide

Behavioral constraints encoded in the prompt:
- Ask only one question at a time; keep questions brief
- Ask open-ended questions; avoid yes/no questions
- Do not give advice or feedback to the interviewee
- When selecting a probe from the suggestions, paraphrase it to fit the conversation naturally

### Reformulation Agent

The reformulation agent is triggered when the classification agent determines that the current main question has not yet been adequately addressed. It rephrases the main question to incorporate information already provided by the interviewee — for example, to elicit clarifications, elaborations, or extensions of a prior answer — before re-presenting it.

> **Note**: The prompt file for the reformulation agent is not included in this repository. Based on the system design described in the paper, the agent receives the main question, the question context, and the relevant prior answers, and returns a reformulated version of the question tailored to the conversation so far.

### Classification Agent

The classification agent makes binary decisions that control interview flow — specifically, whether the interviewee's last response satisfies the condition required to advance to the next question (e.g., whether a yes/no question was answered affirmatively, triggering a section skip).

**System prompt** (`prompts/classification_agent_system_prompt.jinja`):
> Establishes the agent as a binary classifier that returns only `0` or `1`.

**Classification prompt** (`prompts/classification_agent_classification_prompt.jinja`):

Template variables:
- `conversation_history` — prior context of the interview
- `text` — the specific response to classify
- `next_question_instruction` — a natural-language description of the classification criterion
- `classification_examples` — optional few-shot examples (JSON)

The agent returns `1` if the text meets the criterion and `0` otherwise.

---

## Data Availability

All materials underlying this study are publicly available at:

**https://github.com/hjalmarzukile/Comparing_AI_Human_Interview_Reproduction**

The repository includes:
- **Aggregated data**: `agg.csv` (behavioral metrics) and `relevance_and_specificity_scores.csv` (coded quality scores)
- **Interview instrument**: `interview_guide.json` — the full semi-structured interview guide including framing, introductory and closing texts, all main questions, follow-up probes, and conditional skip logic
- **AI system prompts**: Jinja prompt templates for the probing and classification agents (`prompts/`)
- **Analysis code**: `analysis.ipynb` with all statistical analyses and `requirements.txt`

**Materials not shared**: Raw interview transcripts are not shared to protect the confidentiality of participants. Interview responses were collected from university students under a confidentiality agreement, and sharing verbatim transcripts would risk re-identification.

---

## Interview Guide

The following is the full interview guide used in both the human-led and AI-led interviews, reproduced from `interview_guide.json`.

### Study Context

This interview investigates university students' academic help-seeking behavior. The guide has two main sections and a closing: (1) a situation where the student needed and received academic help, (2) a situation where the student needed help but did not ask for it, and (3) a brief closing for any additional reflections. The interview examines the social processes and mechanisms surrounding help-seeking: why students need help, who they turn to, the social situation, their relationship to helpers, and their feelings, attitudes, and behaviors. A secondary aim is to understand how students navigate their social networks to access resources and cope with academic challenges.

### Introduction

> Thank you very much for taking part in our research. In this interview, we will ask you open-ended questions to investigate university students' academic help seeking behavior. It is very common for students to face various academic challenges in their studies such as understanding a concept, solving an exercise, doing a group project and so on.
>
> There are no right or wrong answers. We are very interested in hearing your experience and perspective. The only constraint is that you must write your answers in one go, not using multiple lines. Also, your responses will remain confidential and all personally identifiable information will be removed from publications.

---

### Section 1 — Needed Help and Got Help

*Section aim*: To understand the social situation in which university students needed and were able to get academic help — including reasons for needing help, how students identify when help is needed, what social competencies they use to decide who to approach, and the social processes involved.

**Q1.** Can you think of a situation where you needed academic help and got help with your studies? *(yes/no; if no, section is skipped)*

**Q2.** What did you specifically need help with?

**Q3.** How did you get help?
- Did you personally ask for help? If so, how?
- From whom did you get help? Can you describe the social situation?
- How would you describe your relation to them?
- How do you decide who can help best and how to approach them?

**Q4.** Who else did you consider asking for help?
- What factors influenced your decision?

**Q5.** How did you feel after getting the help?
- What are your expectations from the helper?

**Q6.** In general, what are the reasons you seek academic help?

**Q7.** Who do you ask to get help to resolve your academic challenges? How would you describe your relation to them?

---

### Section 2 — Needed Help but Did Not Ask

*Section aim*: To understand situations where university students refrained from asking for academic help — including why students hold back, their relation to potential helpers, what factors affect their decision, and the social processes involved.

**Q1.** Can you think of a situation where you needed help with your studies but did not ask for it? *(yes/no; if no, section is skipped)*

**Q2.** What did you specifically need help with?

**Q3.** Although you did not ask for help, did you consider asking anyone for help? And if so, who?
- How would you describe your relation to them?
- What were your considerations around asking for help?

**Q4.** Why did you not prefer asking for help?
- When do you feel the most comfortable to ask for academic help?
- When do you feel the least comfortable to ask for academic help?

**Q5.** How did you resolve the issue?
- Did you end up getting any help? Can you describe the social situation?
- How do you decide who can help best and how to approach them?
- When do you feel the most comfortable to ask for academic help?
- When do you feel the least comfortable to ask for academic help?
- Why did you not prefer asking for help?

**Q6.** How did you feel about this situation?
- Under what conditions would you seek help? Can you explain why?
- What changes would you suggest at the university to better meet your academic needs?

**Q7.** In general, in which cases or from whom do you refrain from asking academic help? Why so?

---

### Section 3 — Closing

**Q1.** Before we conclude our interview, is there anything you'd like to add or any perspectives you think we might have missed? *(yes/no; if no, interview closes)*

---

### Closing

> Thank you very much for sharing your insights and experiences. Your input is a valuable contribution to our research.

---

### Interview Guide Notes

- Each main question may be followed by a number of AI-generated follow-up probes (up to the `max_probes_n` limit specified per question). For human-led interviews, the probes listed above served as guidance for the interviewer.
- Conditional skip logic: if the interviewee answers "no" to the opening screening question of Sections 1 or 2, the remaining questions in that section are skipped.
- The probes listed under each question are suggestions; both human interviewers and the AI probing agent were expected to paraphrase and adapt them to fit the natural flow of the conversation.
