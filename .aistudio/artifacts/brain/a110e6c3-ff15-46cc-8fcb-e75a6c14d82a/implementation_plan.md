# ICS Academy — AI Study & Kitab Research Assistant with Firebase Persistence

This upgrade introduces an interactive **AI Study & Kitab Research Assistant** directly inside the ICS Academy (Da'wa College) Student Dashboard, backed by server-side Gemini AI reasoning and real-time Firebase Firestore persistence so students can research Kitab passages, translate between Arabic, Malayalam, and English, draft Da'wa speeches/essays, and permanently save their study notes and dashboard records.

## User Review & Critical Decisions

> [!IMPORTANT]
> Please review the confirmed product choices below and click **Proceed** to begin provisioning Firebase and building the full-stack assistant.

- **Confirmed Decision 1 — Core Feature**: **AI Study & Kitab Research Assistant** tailored for Da'wa College students (covering Fiqh, Hadith, Tafsir, Arabic *Ibarat* analysis, Academic subjects, Trilingual Arabic/Malayalam/English translation, and Da'wa speech/essay outlines).
- **Confirmed Decision 2 — Placement**: **Integrated directly inside the existing Student Dashboard**, accessible via the sidebar and mobile navigation bar alongside Notice Board, Attendance, Leave Register, Time Table, Library, and Exam Results.
- **Confirmed Decision 3 — Data Persistence**: **Full database persistence with Firebase Firestore**, storing saved AI study notes, bookmarked Kitab explanations, and interactive student records across sessions.

---

## 1. Overview & Core Concept

- **What It Does**: Provides every logged-in ICS Academy student with a scholarly AI study companion capable of explaining classical Arabic texts (*Ibarat* with *Tashkeel* / grammatical breakdown), answering Islamic and academic subject questions in Malayalam, English, or Arabic, generating structured Da'wa speech/article outlines, and saving curated study notes to a personal cloud notebook.
- **Target Audience / Persona**: Da'wa College students at ICS Academy Manhappatta balancing classical Islamic curriculum (Fiqh, Usul, Hadith, Tafsir, Nahw) with modern academic studies.
- **Key Value**: Bridges classical *Kitab* study and modern digital research in one unified workspace while preserving every student's saved notes and records in Firebase Firestore.

---

## 2. User Experience & Visual Design

- **Key User Flows**:
  1. **Select Study Mode & Subject**: Inside the Student Dashboard, the student navigates to the **AI Kitab & Study Assistant** section and chooses a mode:
     - *Kitab & Ibarat Explainer* (vocalization, word-by-word meaning, and *Nahw/Sarf* analysis)
     - *Subject Q&A* (Fiqh, Hadith, Tafsir, History, Academic Science & Humanities)
     - *Trilingual Translator* (Arabic ↔ Malayalam ↔ English)
     - *Da'wa Speech & Essay Outliner* (structured points, Quran/Hadith references, and conclusion)
  2. **Interactive Query & Quick Prompts**: Students type or paste an Arabic *Ibarat*, Malayalam question, or English topic—or click curated quick-start scholarly prompts—and receive a clearly formatted response.
  3. **Save to Cloud Notebook**: With one click on **Save to Notebook**, any AI explanation or translation is saved to Firebase Firestore under the student's profile, complete with subject tags, search filtering, and delete/archive controls.
- **Visual Identity & Theme**:
  - *Aesthetic Direction*: Scholarly, refined institutional dashboard matching the existing ICS Academy emerald, deep teal (`#081E21`), and lime accent (`#84CC16`) palette across both Light and Dark themes.
  - *Color Palette & Mood*: 60% neutral canvas (`--background`), 30% structured card surfaces with 1px hairline borders (`--border`), and 10% semantic emerald/lime accents for active study modes and primary actions.
  - *Typography & Hierarchy*: `El Messiri` for display headings, `Plus Jakarta Sans` for crisp UI and Malayalam/English body prose, and clear Arabic script legibility with generous line height (`leading-relaxed`) for *Kitab* excerpts.
  - *Component Styling & Layout*: Two-zone educational workspace inside the dashboard—an interactive AI Research & Kitab Prompt stage on the left (`3 columns`) paired with the student's Saved Cloud Notebook & Quick Reference deck on the right (`2 columns`).
- **Interactive Feedback & Motion**: Smooth loading skeletons during AI generation, instant optimistic updates with Firestore real-time listeners (`onSnapshot`), and clear empty/error states.

---

## 3. Key Product Decisions & Trade-Offs

- **Decision 1 — Server-Side Gemini API Architecture**:
  - *Chosen Approach*: Route all AI requests through an Express backend endpoint (`/api/ai/study-assistant`) using the `@google/genai` SDK and `gemini-3.8-flash` with domain-specific system instructions for Da'wa College scholarship.
  - *Why*: Keeps the API key strictly on the server, ensures fast low-latency responses in Malayalam, Arabic, and English, and structures responses with clear headings, vocalized Arabic quotes, and study takeaways.
  - *Alternatives Considered*: Client-side SDK calls were rejected to protect credentials and adhere to full-stack security standards.
- **Decision 2 — Firebase Firestore + Auth Sync**:
  - *Chosen Approach*: Provision Firebase Firestore and Google Authentication, using hardened security rules (`firestore.rules`) that isolate each user's saved study notes and student records by `request.auth.uid` and strict schema validation.
  - *Why*: Guarantees zero-trust security, real-time cross-device synchronization, and permanent storage for student study notebooks.

---

## 4. Technical Architecture & Data Strategy *(Technical Reference)*

### Architecture & Component Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     ICS Academy Student Dashboard                       │
│                                                                         │
│  ┌───────────────────────────────┐   ┌───────────────────────────────┐  │
│  │  Sidebar & Mobile Navigation  │   │  Header & Student Switcher    │  │
│  │  (+ AI Kitab Assistant Link)  │   │  (+ Firebase Auth Status)     │  │
│  └───────────────┬───────────────┘   └───────────────┬───────────────┘  │
│                  │                                   │                  │
│                  ▼                                   ▼                  │
│  ┌───────────────────────────────────────────────────────────────────┐  │
│  │            AI Study & Kitab Research Workspace (2-Zone)           │  │
│  │                                                                   │  │
│  │  ┌─────────────────────────────┐  ┌────────────────────────────┐  │  │
│  │  │ Zone 1: AI Research Stage   │  │ Zone 2: Saved Cloud Notes  │  │  │
│  │  │ • Mode Selector (4 Modes)   │  │ • Real-time Note List      │  │  │
│  │  │ • Language Selector         │  │ • Category / Search Filter │  │  │
│  │  │ • Prompt Input & Presets    │  │ • View / Copy / Delete     │  │  │
│  │  │ • Formatted Response Viewer │  │   Handlers                 │  │  │
│  │  └──────────────┬──────────────┘  └──────────────┬─────────────┘  │  │
│  └─────────────────┼────────────────────────────────┼────────────────┘  │
└────────────────────┼────────────────────────────────┼───────────────────┘
                     │ POST /api/ai/study-assistant   │ Firestore SDK
                     ▼                                ▼
      ┌──────────────────────────────┐  ┌──────────────────────────────┐
      │   Express Backend Server     │  │     Firebase Firestore       │
      │   • @google/genai SDK        │  │   • /studyNotes/{noteId}     │
      │   • Model: gemini-3.8-flash  │  │   • /leaveRequests/{reqId}   │
      │   • Da'wa Curriculum Prompt  │  │   • Hardened Security Rules  │
      └──────────────────────────────┘  └──────────────────────────────┘
```

### Data Model & State

1. **`StudyNote` Entity (`/studyNotes/{noteId}`)**:
   - `uid` (string, required): Authenticated Firebase user ID (`request.auth.uid`).
   - `studentId` (string, required): Selected student profile ID (`st-101`, etc.).
   - `mode` (enum: `'kitab' | 'qa' | 'translation' | 'dawa'`, required): Research mode used.
   - `language` (enum: `'ml' | 'en' | 'ar'`, required): Output response language.
   - `prompt` (string, 1–2000 chars, required): The student's question or Arabic *Ibarat* passage.
   - `response` (string, 1–15000 chars, required): The structured AI explanation or outline.
   - `createdAt` (timestamp, immutable): Server timestamp (`request.time`).
2. **`LeaveRequest` Entity (`/leaveRequests/{requestId}`)**:
   - Allows students to submit new leave applications directly from the Leave Register card and persist them in Firestore alongside their existing records.

### Interactive Component & State Mapping

- **Mode & Language Controls**: Switching between *Kitab & Ibarat Explainer*, *Subject Q&A*, *Trilingual Translator*, and *Da'wa Speech Outliner* updates the prompt placeholder and curated preset chips immediately.
- **Ask Assistant Handler**: Validates non-empty input ($\le 2000$ chars), transitions to a structured loading state, posts to `/api/ai/study-assistant`, and renders the formatted scholarly answer.
- **Save to Cloud Notebook Handler**: Persists the active prompt, response, mode, and student ID to Firestore `/studyNotes/{noteId}` using `serverTimestamp()` and structured `handleFirestoreError` diagnostics.
- **Notebook Search & Filter**: Filters saved notes in real time by mode or keyword and supports one-click copy and deletion.
