---
name: teach-me
description: |
  Delivers a ground-up, structured lesson from unstructured input — words, concepts, phrases, and sentences dumped without organization. Use this skill whenever the user invokes /teach-me, lists a bunch of concepts or terms they want to understand, says things like "explain X, Y, Z to me", "teach me about...", or just brain-dumps a set of related terms. The skill identifies the overarching concept landscape, builds an axiomatic lesson from first principles, and teaches coherently — not as a glossary, but as a unified conceptual narrative.
---

# Teach Skill

The user has dumped a collection of words, phrases, sentences, and concepts — probably without structure. Your job is to teach them.

## Step 1: Find the conceptual landscape

Read all the input. Don't treat it as a list to define — treat it as a window into what they're trying to understand. Ask:
- What is the smallest set of core concepts that, together, fully contains everything mentioned?
- What foundational concepts are needed to understand those core concepts, even if not mentioned?

That's your syllabus. You're building the minimal covering set — not a surface-level glossary, but the conceptual landscape a learner needs to navigate this space.

## Step 2: Structure the lesson axiomatically

Order the concepts so each one builds on what came before. Think of it like a dependency graph — teach the leaves before the nodes that depend on them. The learner should never encounter a term that hasn't been explained yet.

A rough heuristic for ordering:
1. The most primitive, self-contained concept first
2. Then the next concept that only depends on what's been established
3. And so on upward toward the most complex or applied ideas

Don't force a bullet-point list. Write in flowing prose that naturally introduces each idea in context. Definitions should feel like they emerge from understanding, not be recited. Use analogies liberally — especially for abstract or academic concepts.

## Step 3: Enrich where it counts

### For abstract or academic concepts
Give a trivial concrete example — something so simple it almost feels condescending, but illuminates the idea clearly. The best examples are ones where the reader thinks "oh, *that's* what that means."

### For technical tools, frameworks, or systems
Include a brief orienting section early in the explanation:
- What problem did this exist to solve?
- What was annoying or painful before it existed?
- Why was it designed the way it was? (e.g., React's component model wasn't arbitrary — it was a response to the chaos of mutating the DOM manually)

This context transforms a tool from "a thing to memorize" into "a logical response to real constraints." It sticks better.

### Must-know adjacent concepts
If something important is clearly adjacent to what was mentioned but wasn't included — and a learner would be confused or incomplete without it — weave it in. Don't pad with trivia; include what genuinely completes the picture.

## Step 4: Write the lesson

Write as a teacher who cares, not as a textbook. The tone should be clear, direct, and a little conversational. Headers are fine for navigation but don't over-structure — the ideas should flow.

The lesson doesn't need to explicitly define every word the user mentioned. If the concept is better taught through a unified explanation that covers all the terms without calling them out one by one, do that. The goal is understanding, not coverage checkboxes.

---

## Example: what good input and output look like

**Input (brain dump):**
> monads, functors, pure functions, side effects, map, flatMap, Haskell

**What not to do:** define each term in order, like a dictionary.

**What to do:** recognize this is about functional programming and category theory as applied to FP. Start from "what is a pure function and why does it matter," build up to functors as "things you can map over," then monads as "functors that can be chained without nesting," and weave in Haskell as the language designed around these constraints and why. `map` and `flatMap` become concrete examples, not separate entries.

---

Teach as if the user is intelligent but new to this topic. Assume curiosity, not prior knowledge.
