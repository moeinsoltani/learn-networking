# CLAUDE.md — Linux Networking Learning Project

## Student Profile
- Experience level: very limited (beginner)
- Platform: Windows 11, with access to a Linux VM and WSL2
- Use the Linux VM for labs (WSL2 as fallback for early lessons)

## Teaching Methodology
For each lesson:
1. Create a lesson file at `lessons/lesson-NN-topic.md` before teaching
2. The file contains: mental model, mechanics, lab commands with expected output, and checkpoint questions with blank answer fields
3. Student reads the file, asks questions in chat if needed
4. Student fills in their answers to checkpoint questions directly in the file
5. Student reports back (pastes answers or says "done") — I read and correct
6. Only move to the next lesson when checkpoints pass
7. At the end of each Phase, give a written test covering all lessons in that phase

## Lesson File Format
Each lesson file must contain:
- **Concept** — mental model, analogy, ASCII diagram
- **How it works** — mechanics (brief, focused on "why")
- **Lab** — exact commands with expected output ($ prompt = run this, # = comment)
- **Checkpoint** — questions with a `**Your answer:**` field left blank for the student to fill in
- **Homework** — one task slightly harder than the lab, done independently before next lesson

## Lesson Index
- Lesson 01: `lessons/lesson-01-namespaces-intro.md` — What a network namespace is
- Lesson 02: `lessons/lesson-02-host-as-namespace.md` — The host is just another namespace
- Lesson 03: `lessons/lesson-03-build-namespaces.md` — Build two isolated namespaces

*(Update this index as lessons are created)*

## Progress Tracking
- Current lesson: 01
- Completed lessons: none yet
- Completed phases: none yet
