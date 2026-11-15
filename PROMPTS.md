# Prompt log — React Lab 2

Part of what you submit. It shows *how* you used AI: did you hand it the contract and the
rules, or type "make it work"? Fill it in **as you go**, not from memory at the end.

- Name and roll number:Anthati Madhu Vikas (MT2026010)
- AI tool and plan (e.g. ChatGPT free, Gemini, Claude free, Copilot):Claude free 

**Log every prompt you send, in order — including the ones that didn't help.** Paste each
prompt in full. One exception: if you pasted CONTRACT.md's prompt 1 unchanged, write
"CONTRACT.md prompt 1, unchanged". Summarise each reply; don't paste whole files.

---

## Prompt 1 — generate

**Sent:**

```text
CONTRACT.md prompt 1, unchanged
```

**What came back:** The AI implemented the Lab 2 pre-lab search functionality in SearchResults.jsx, useCourseSearch.js and CourseSearch.jsx.

It used useState and useEffect, with an ignore flag in the cleanup function to prevent stale API responses.

**Did it run?** (`npm run dev`: list loads? add works? 409 shown? fieldErrors shown?
`npm test`: _20_ passed / _0_ failed)

**What I did with it:**

---Ran the tests locally, corrected the working directory when the initial test execution failed, and verified that the search tests passed.

## Prompt 2 —

**Why I sent it:** (for a fix prompt: which decision in AUDIT.md it carries out) To implement the remaining Courses functionality according to the README, CONTRACT.md and provided test cases.

**Sent:**

```
pasted the README sections "Core and stretch", the pre-lab and lab-day tables, "The API", "Rules" and "Done means"

```

**What came back:**The AI implemented:
- useCourses.js
- CoursesView.jsx
- AddCourseForm.jsx
- CoursesPage.jsx
The implementation followed the required restrictions:
- Used only useState and useEffect.
- Used apiFetch instead of raw fetch.
- Maintained a single status field.
- Used an ignore flag to handle stale API responses.
- Updated the list using the successful 201 response instead of reloading it.
- Avoided useMemo, useCallback, useRef, useReducer and Context.

**What I kept, and what I rejected:**

---Kept the implementation after reviewing it against CONTRACT.md §4. The initial implementation had a missing closing brace in AddCourseForm.jsx, which the AI corrected after I reported the issue.

<!-- Copy the "Prompt 2" block for each further prompt. -->

## Afterwards

- How many prompts did the whole feature take?Two main implementation prompts.
- Which of your prompts would you write differently now, and how?I would provide the exact contract requirements, API specifications and implementation restrictions upfront to make the initial prompt more precise.
- One thing the rules in prompt 1 prevented (or failed to prevent):They also prevented unnecessary abstractions and ensured that the implementation followed the assignment requirements.
