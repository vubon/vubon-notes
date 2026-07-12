---
title: "Introducing Kolom: A Native Bengali Keyboard for macOS"
description: "Discover Kolom, a free, open-source, native Bengali phonetic keyboard for macOS (Apple Silicon). Learn how Kolom brings fast and private Bengali typing to Mac."
keywords:
  - "Kolom"
  - "kolom"
  - "kolom keyboard"
  - "Bengali keyboard macOS"
  - "Mac Bengali typing"
date: 2026-07-08T13:30:00Z
draft: false
tags:
  - "Kolom"
  - "macOS"
  - "Swift"
  - "Open Source"
  - "Bengali"
  - "Input Method"
categories:
  - "Projects"
  - "Development"
---

I've been using Avro Keyboard for over a decade. It was the absolute easiest way to write Bengali digitally. However, with Apple eventually phasing out support for Rosetta and Intel-based applications on Apple Silicon Macs, I realized something important: the Bengali community needed a modern, future-proof, native alternative.

So, I decided to build one. Meet **Kolom**! 🚀

## What is Kolom?

Kolom (কলম, meaning "Pen") is a free and open-source Bengali Input Method designed specifically for modern macOS architectures (Apple Silicon M1/M2/M3/M4). It is written entirely in Swift using Apple's native `InputMethodKit`, making it blazing fast and deeply integrated with the operating system.

### Key Features

1. **Fully Native & Apple Silicon Ready**: Built from the ground up for modern macOS, it uses zero legacy frameworks and doesn't rely on Rosetta translation.
2. **Avro-Compatible Phonetic Typing**: If you are used to typing `ami bangla bhalobashi`, you'll feel right at home. The phonetic transliteration engine uses the exact same mapping rules you already know and love.
3. **Intelligent Word Suggestions**: Kolom includes a robust Candidate Engine that suggests correct spellings as you type using a lightweight offline Bengali dictionary.
4. **Privacy First**: Everything runs locally on your machine. There is no cloud telemetry, no internet requirements, and no data collection.

## The Technical Journey

Building an Input Method for macOS is a uniquely challenging software engineering task. Apple’s `InputMethodKit` is incredibly powerful, but notoriously under-documented.

To make Kolom work seamlessly, I had to architect several distinct engines from scratch:

- **Transliteration Engine**: A complex state machine that converts phonetic English keystrokes into Bengali characters on the fly. It intelligently handles complex conjuncts (যুক্তাক্ষর) automatically.
- **Candidate Engine**: A fast lookup system that searches through a local dictionary file in real-time to offer spelling suggestions in a floating UI window.
- **Composition Engine**: The orchestrator that manages the state of the text currently being typed (the "marked text") and interacts directly with macOS text fields across different applications.

One of the coolest parts of the Transliteration Engine is how it handles conjuncts natively. For example, if you type `strI`, the engine translates `s`, `t`, and `r` into consonants and intelligently inserts the invisible Hasanta (`্`) between them to seamlessly render the conjunct **স্ত্রী**. If you need to force a separation, you simply type `o` as an inherent vowel break (e.g., `boNgg` for **বঙ্গ**).

## Open Source for the Community

I built Kolom because I needed it for my daily workflow, but I open-sourced it because I want the Bengali community to thrive on modern computing platforms. 

Kolom is completely free and licensed under the permissive MIT License. We officially released **v1.0.0** today!

If you are a Mac user who writes in Bengali, I'd love for you to give it a spin. And if you are a Swift developer, I warmly welcome contributions, pull requests, and bug reports.

You can check out the source code, read the documentation, and download the latest release directly from GitHub:

👉 **[Kolom GitHub Repository](https://github.com/vubon/Kolom)**

Lastly, I want to give a massive thank you to the creators of Avro Keyboard (OmicronLab). They pioneered phonetic typing and completely revolutionized how we write our mother tongue digitally. Kolom stands on the shoulders of giants. 🙏

---
*আমি আমার মাতৃভাষা "কলম" দিয়ে লিখি!*
