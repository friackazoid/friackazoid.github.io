---
layout: post
title: Vibe Coding a Personal App
categories: [LLM]
---

I've tried to create an App for personolised calendar with vague understanding of mobile platform programming. And somehow… the result is personally satisfying.

# Introduction

I am a fan of various GTD systems, journaling, etc. Moleskine is my love, but paper feels old-school, especially when you already have a tablet (Boox NoteAir3C in my case). Because I was always missing a perfect calendar, I used PDF templates and handwritten notes. They work, but they have clear disadvantages, which eventually pushed me to create my own “perfect” app.

First, a confession: I don't know Dart and have only a vague understanding of mobile platform programming. During this project, I barely learned any of it.

# Technical Setup

The project took me about 4 hours of vibe coding and about an hour to set up the environment (this blog post took me longer!).

Despite my engineering background, in this project I haven’t touched a line of code. The prompts describing HOW to implement things were also generated with the help of an LLM.

My technical setup is:

- OpenAI Plus and Codex
- NeoVim

My hardware is an E‑Ink tablet, Boox NoteAir3C. Since the tablet runs on Android, I used Flutter for local testing and installation via the debug interface.


# Vibe Coding Process and Result

As I really don’t know anything about coding applications for mobile platforms, I used ChatGPT to generate prompts for Codex. In those prompts, I described WHAT I wanted to see in my application, and it generated very detailed prompts with technical details on HOW to implement it. Example:

<details>
<summary>Prompt: "Create links between the monthly, weekly, and daily planners. ...</summary> When I click on a day in the monthly planner, it should open the daily planner for that specific date. The same behavior should apply to the weekly planner."
</details>
<details>
<summary>ChatGPT Response: You are a senior Flutter engineer. ...</summary>

Task:
Create navigation links between Monthly, Weekly, and Daily planner pages.

User flows:
1) Monthly -> Daily
- When user taps a day cell in the MONTHLY calendar grid, open the DAILY planner for that exact date.

2) Weekly -> Daily
- When user taps a day header/row in the WEEKLY planner (Mon–Sun list), open the DAILY planner for that exact date.

3) Optional nice-to-have (implement if trivial):
- Daily page has buttons to jump to the containing week/month (or a menu), but do NOT overcomplicate.

Constraints:
- Flutter stable, Dart only.
- No external navigation libraries required (use Navigator 1.0 or Router 2.0 already used in the project).
- Do not break persistence (daily/week/month data must still save/load as before).
- No new features beyond linking and routing.

Routing requirements:
- Define a single source of truth for date navigation:
  - DailyPlannerPage must accept a DateTime initialDate OR a dateKey "yyyy-MM-dd".
  - WeeklyPlannerPage accepts weekStart (Monday) OR a date inside the week.
  - MonthlyPlannerPage accepts year/month OR a date inside the month.
- Implement typed route helpers:
  - routeToDaily(BuildContext, DateTime date)
  - routeToWeekly(BuildContext, DateTime weekStartOrAnyDay)
  - routeToMonthly(BuildContext, DateTime anyDay)

- If using Navigator 1.0:
  - Use named routes with arguments OR MaterialPageRoute.
- If using Router 2.0:
  - Add route paths:
    - /daily?date=yyyy-MM-dd
    - /weekly?start=yyyy-MM-dd (Monday)
    - /monthly?month=yyyy-MM
  Choose ONE approach consistent with the existing project and implement cleanly.

Monthly planner changes:
- In the calendar grid cell widget, add InkWell/GestureDetector:
  - Only allow tapping valid dates (including out-of-month cells if you show them, but pass correct date anyway).
  - On tap: open Daily planner for that date.

Weekly planner changes:
- In Mon–Sun list:
  - Make each day header (or entire day section) tappable.
  - On tap: open Daily planner for that day’s date.

Daily planner changes:
- Must load the correct plan based on passed date.
- Ensure date switching inside Daily planner (prev/next day buttons) still works.

Persistence constraints:
- DO NOT store navigation state in persistence.
- Ensure any pending edits are saved before navigating away:
  - On tap from monthly/weekly -> daily, call a saveCurrentPlan() if necessary (or rely on debounced autosave but ensure no loss).

Deliverables:
- Updated MonthlyPlannerSpread / MonthlyPlannerPage with tap-to-open-day behavior.
- Updated WeeklyPlannerSpread / WeeklyPlannerPage with tap-to-open-day behavior.
- Updated routing setup + route helper functions.
- DailyPlannerPage updated to accept initialDate/dateKey.
- Minimal tests or debug prints are optional, but code must be clean.

Acceptance checklist:
- Tapping a day in MONTHLY opens DAILY for that date.
- Tapping a day in WEEKLY opens DAILY for that date.
- No data loss when navigating (notes/tasks remain persisted).
- Works across month/year boundaries (e.g., tapping Jan 1 from Dec grid if shown).
</details>

My personal impression is positive; the most fun part was that both LLMs generated what you actually said, not what you meant to say. For example, with my first request Codex generated an application with four identical views, each containing all elements. I had to regenerate the app, as prompts intended to change the layout were not successful. 

You can find the resulting app [here](https://github.com/friackazoid/DreamNotebook). And here are some screenshots of the app in action:

| ![Creating a new activity](/images/2026-01-11-vibe-coding/1.png) | ![Daily Schedule](/images/2026-01-11-vibe-coding/2.png) |
|--------------------------------------------------------------|--------------------------------------------------------------|
| ![Monthly Planner](/images/2026-01-11-vibe-coding/3.png) | ![Collections](/images/2026-01-11-vibe-coding/4.png) |


# What I Know Is Wrong

Let’s be explicit so nobody has to write an angry comment explaining the obvious.

- *Security & authentication:* I would never use this code for anything involving accounts, personal data syncing, or auth of any kind. That also means no synchronisation with popular calendars from Google or Apple, which I personally miss.
- *Battery efficiency:* I have no idea how good or bad my application is in this regard.
- *Integration with native note applications:* handwriting on a custom canvas is very slow, and doing this inside a native note app would be much easier.

# Why I Still Think This Was Worth Doing

This project wasn’t about learning Dart properly. It was about:

- Lowering the barrier to experimentation
- Prototyping ideas that would never survive a formal design process
- Reclaiming joy in making tools for myself

There is value in building things that are allowed to be imperfect.

# Where I Want Professional Feedback

This is the real reason for this post. I’m especially curious about:
- What parts of this code are dangerous vs merely ugly
- Where battery inefficiency usually hides in Flutter apps
- Which mistakes are common beginner traps vs truly bad ideas
