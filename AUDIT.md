# Audit — React Lab 2

Part of what you submit. Fill in sections 0–4 **before** you ask the AI to change anything:
deciding what's wrong is your job, not the AI's. The sheet you audit against is "The Shape
of a Good React Slice" — section 08 of the lab doc.

Point at code as `file:line`, e.g. `CoursesPage.jsx:14`.

> Note: the "AI's first draft" audited below is `ai-sample/CoursesPage.jsx`, the typical AI draft
> the lab provides. I did not generate a separate first draft from CONTRACT.md prompt 1. The
> "after" is my final code in `src/courses/`. Line numbers are in `ai-sample/CoursesPage.jsx`
> unless a file is named.

---

## 0. What came back

| File the AI gave you | What's in it (components, hooks) | Lines |
| --- | --- | --- |
| `ai-sample/CoursesPage.jsx` | One component, `CoursesPage`: 13 `useState`, 1 `useRef`, 2 `useCallback`, 1 `useMemo`, 1 `useEffect`; the list, the form, the fetching and the add all in it | 161 |

- Does it run? Yes. `test/courses/CoursesPage.test.jsx` passes against it (I ran it): list loads,
  a valid add appears, a duplicate code shows the 409 message, bad input shows the reasons under
  the inputs. I did not open it in the browser; check that yourself.
- Requests in the Network tab — on page load: 1 `GET /api/courses` (2 in dev, because StrictMode runs
  the effect twice) ; per add: 1 `POST` **plus** 1 more `GET` (`fetchCourses()` at line 80). This is
  from reading the code; confirm in the Network tab.
- `npm test` before any fix: 4 passed / 31 failed in `test/courses/` (the sample passes the page test only;
  the 31 tests of `useCourses`, `CoursesView` and `AddCourseForm` fail because those files don't
  exist). The search tests pass.

## 1. Part A — walk the allowed list

Is each element present, and in the right place?

| Element (the sheet, Part A) | Where it is in the AI's code (`file:line`, or "missing") | Right place? If not, where should it be? |
| --- | --- | --- |
| One View component that only renders | missing (lines 92–158 all inside `CoursesPage`) | No. Should be `CoursesView.jsx`, props only |
| Fetch + logic in a separate hook or module | missing (`fetchCourses` lines 31–43, `handleSubmit` 54–89 inside the component) | No. Should be `useCourses.js` |
| `useState` for the course list, at the right level | line 15 | Right idea, wrong place: belongs in `useCourses` |
| `useState` for the form's inputs, at the right level | lines 19–24 (six separate states), 25–27 | Right idea, wrong place: belongs in `AddCourseForm` (one object state) |
| `useEffect` — for the initial fetch only (with a cleanup) | lines 45–47, fetch only, but **no cleanup** | Half. Needs an `ignore` flag (StrictMode runs it twice) |
| Props down, callbacks up (the form raises "add this course") | missing | No. The form should get `onAdd` from the page |
| `.map` to render the list, with a stable `key` | line 105–106, `key={index}` | No. Should be `key={course.id}` |
| Loading / error / empty states, from one `status` | lines 16–17 (`loading` boolean + `error`) | No. Should be one `status`: "loading" / "ready" / "error" |

## 2. Part B — scan for "not yet"

Search the files (VS Code: Ctrl/Cmd+Shift+F) for each one.

| Not yet | Found? (`file:line`) | What it does in *this* slice | Cut, or keep — and why |
| --- | --- | --- | --- |
| `useMemo` | line 50–52 | Sorts a copy of the list by code on every change | Cut. The API already returns the list sorted, and the new course is inserted in order in the hook |
| `useCallback` | lines 31 and 54–89 | Keeps the function the same between renders, but nothing receives it as a prop or depends on it for speed | Cut both. A plain function works the same |
| `useReducer` | not found | — | — |
| `useRef` | line 28 (used at line 81) | Puts the focus back on Code after an add | Cut. The contract doesn't ask for it |
| Context (`createContext`, `useContext`) | not found | — | — |
| A data-fetching library, or `axios`, or raw `fetch` instead of `apiFetch` | not found: it uses `apiFetch` (lines 34, 61) | Good: this is correct | Keep |
| TypeScript (types, interfaces, `.ts`/`.tsx`) | not found | — | — |
| Anything else nobody taught you (name it) | line 18 and 95 (`courseCount`), line 11 (`import React`), lines 67–68 (`parseInt`) | `courseCount` is state that duplicates `courses.length` and adds a line the contract didn't ask for; `import React` isn't needed with the new JSX setup; `parseInt` without a radix instead of `Number` | Cut `courseCount`; drop the `React` default import; use `Number(...)` |

## 3. The rule — every hook and every component split

One row for **each** `useState`, `useEffect` and custom hook call, and for **each**
component. Say what it does *in this slice*. "The AI added it" and "just in case" are not
answers — if you can't name its job, the decision is "cut".

| `useState` courses (15) | Holds the list the screen shows | Move to `useCourses.js` |
| `useState` loading (16) + error (17) | Say what the list is doing | Replace with one `status`; move to `useCourses.js` (error is a message string) |
| `useState` courseCount (18) | Nothing: it is just `courses.length` | Cut |
| `useState` code…description (19–24) | Hold what is typed in the six controlled inputs | Move to `AddCourseForm.jsx` as one `values` object |
| `useState` submitting (25) | Disables the button while the server answers | Move to `AddCourseForm.jsx` |
| `useState` formError (26) + fieldErrors (27) | Show what the server said went wrong | Move to `AddCourseForm.jsx` as one `error` (fieldErrors read from it, so it can't go out of sync) |
| `useRef` codeInputRef (28) | Focus after add, not asked for | Cut |
| `useCallback` fetchCourses (31) | Nothing needs a stable function | Cut; the effect calls `apiFetch` directly |
| `useEffect` (45) | Loads the list once when the page appears | Keep, with `[]` and an `ignore` cleanup; move to `useCourses.js` |
| `useMemo` sortedCourses (50) | Nothing: the API sorts and the hook inserts in order | Cut |
| `useCallback` handleSubmit (54) | Nothing | Cut; plain function in `AddCourseForm.jsx` |
| Component `CoursesPage` (14) | Does everything | Split: keep a thin `CoursesPage` that calls the hook and shows the other two |
| (missing) `CoursesView` | Only renders the list/states | Add (`CoursesView.jsx`, props only) |
| (missing) `AddCourseForm` | The form and its own state | Add (`AddCourseForm.jsx`, `onAdd` prop) |
| (missing) `useCourses` | Fetch + add logic, away from the JSX | Add (`useCourses.js`) |

## 4. Decisions — before any fix prompt

- I will **cut**: `useMemo`, both `useCallback`s, `useRef` and the focus, `courseCount`, the `React` import, the "N courses in the catalogue" line.
- I will **move** (from → to): fetching and the add logic → `useCourses.js`; the list and states
  → `CoursesView.jsx`; the form, its inputs and its errors → `AddCourseForm.jsx`.
- I will **add** (missing from the AI's version): a single `status`; the `ignore` cleanup in the effect; an
  `onAdd` callback from page to form; `key={course.id}`; `aria-invalid`, `aria-describedby` and
  `role="alert"` on the errors; `Number()` for credits and capacity.
- Other smells I found (state stored that could be computed, a reload after add, browser
  validation, `key={index}`, swallowed errors…): `courseCount` is state that could be computed
  (line 18); the list is reloaded after every add (line 80), so each add is two requests instead
  of one and the list flickers to "Loading…"; `key={index}` (106); loading and error as separate
  values that can disagree; no cleanup on the effect, so a stale answer could win; the field
  errors are shown only for four inputs and without `aria-*`; the form-level error has no
  `role="alert"`; the button text changes to "Adding..." (fine, but the test finds it by role).

---

## 5. Strip check (section 14 of the doc)

For each "not yet" item you removed — one at a time:

| Removed | `npm test` after | The app after (loads / adds / shows errors) |
| --- | --- | --- |
| `useRef` + the focus call | `CoursesPage.test.jsx`: 4 passed | Loads, adds and shows errors as before (only the focus-back-to-Code is lost) |
| `useMemo` (use `courses` directly) | 4 passed | Same: the API returns the list in code order, and the reload keeps it that way |
| `courseCount` state and its line | 4 passed | Same, minus the "N courses" line |
| `useCallback` on `handleSubmit` | 4 passed | Same |

I ran these one at a time, each on top of the previous, against a scratch copy of the sample
(not the final code). Nothing broke, so none of them was earning its place. The `useCallback` on
`fetchCourses` I did not strip on its own: it only goes away together with the reload after add.

(Browser check of "the app after": do it yourself with `npm run dev` before submitting.)

## 6. Before / after

|  | Before (the AI's first version) | After (yours) |
| --- | --- | --- |
| Files and components | 1 file, 1 component (161 lines) | 4 files: `useCourses.js`, `CoursesView.jsx`, `AddCourseForm.jsx`, `CoursesPage.jsx` (162 lines in all) |
| `useState` calls (and where) | 13, all in `CoursesPage` | 6: 3 in `useCourses` (courses, status, error), 3 in `AddCourseForm` (values, submitting, error) |
| `useEffect` calls | 1 (no cleanup) | 1 (`[]`, with `ignore` cleanup) |
| "Not yet" items | `useMemo` 1, `useCallback` 2, `useRef` 1 | none |
| Requests: on load / per add | 1 / 2 (POST + GET) | 1 / 1 (POST only) |
| `npm test` | 4 of 35 `test/courses/` tests pass (before the split) | 62 passed / 0 failed |

## 7. Two sentences

- One thing the AI got right that you might not have thought of: It reads `err.fieldErrors` straight off the thrown `ApiError` (one `error` state), instead of copying them into a second state, so the message and the field errors can never disagree.
- One thing you will never accept from an AI without checking again: "It works." The sample passed the page test and still had a reload after every add, `key={index}` and four hooks nobody needed. A passing test is not the same as a good slice, so I check the code against the sheet, not just the test.
