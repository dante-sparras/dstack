# Asking the user

Finding facts is your job; decisions are the user's. Never ask for something you can look up: the stack, whether a license exists, the current version, the visibility of an existing repo. Ask only for what the repo can't tell you.

## Format

Every question this skill asks uses this format, whether it's one question or a whole message of them.

```markdown
<context the user needs, if any>

❓ **Q<n>** - **<question title>**: <question body; may span paragraphs and list the choices>

➡️ <your recommended answer>

---

❓ **Q<n>** - **<question title>**: <question body>

➡️ <your recommended answer>

---

Reply **yes** to accept every recommendation, or answer by number (`Q2 open`).
```

- **Word each question so "yes" accepts the recommendation.** Ask "Use the MIT license? Or name another SPDX id, or none for proprietary code.", not "Which license?". When the recommendation flips, so does the wording: "Skip versioned releases?" when you recommend no.
- **Number** the questions Q1, Q2, … within each message, counting only the questions you actually ask, and put `---` between them.
- **Add the ➡️ line only when there is a recommendation**, and keep it to the answer itself (`➡️ MIT`, `➡️ All of them`). The reasoning goes in the body. Open questions, such as a project name, have none.
- **List choices** in the body, one per line, with the name in bold when it's a short label (`- **Strict**: …`).
- **Put context above Q1**, such as the private-repo limits in step 3, never between questions.
- **Show dependencies.** When a question only matters for one answer to another question in the same message, start its body with that condition in bold: "**If public:** …". Its answer is dropped when the condition doesn't hold.
- **End with the reply line** when the message has more than one question with a recommendation. When some questions are open, say which ones still need an answer: "Answer Q1 to Q4; for the rest, reply **yes** to accept every recommendation."

## Reading the answers

| The user replies | It means |
| --- | --- |
| "yes" to a question | Its recommendation. |
| "yes" to the whole message, or "yes to all" | Every recommendation in it. Open questions still need their own answers; ask only for those. |
| A choice's name (`Q2 open`) | That choice. |
| Free text | Use it as written. If it fits none of the choices and its meaning is unclear, ask a follow-up about that question only. |
| Nothing for a question | Ask again for that question only; never assume the recommendation. |
