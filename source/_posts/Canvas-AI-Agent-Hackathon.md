---
title: Canvas AI Agent Hackathon
date: 2026-01-05 22:34:39
tags:
---
 
 {% asset_img "canvas agent.jpg" "chat window" %}

## Overview

At the **Skandalaris Hackathon 2025 (AI Track)**, my teammates **DeYu Zhang**, **Mark Song**, and I built **Canvas AI Agent**, an LLM-powered assistant designed to help students navigate course content directly within Canvas. The agent answers questions, summarizes materials, and reminds students of relevant deadlines using contextual course data. Our project earned a **Runner-Up Award (Top 5 overall)** and received $500 in cash prizes.

## Motivation

Canvas courses often contain fragmented information spread across announcements, files, assignments, and discussion boards. As a result, students spend unnecessary time searching for information instead of focusing on learning. Our goal was to build an agent that understands course-specific context, not just raw text, and provides answers through a clean and intuitive conversational copilot interface with little learning curve, especially benefiting accessibility-constrained and younger students.

## Highlights

- Integrated **authorized Canvas API actions**, allowing the agent to incorporate real student-accessible course operations directly into its responses  
- Implemented **retrieval-augmented generation (RAG)** using vector similarity search to accelerate discovery of relevant Canvas materials while filtering out unrelated content  
- Built a **real-time frontend** using WebSockets to stream incremental responses and progress updates, supporting concurrent multi-user sessions  

## Technical Architecture

**Frontend**

- **React + TypeScript**  
- Chat-style conversational interface  
- **WebSocket** powered real-time response streaming  

**Backend**

- **FastAPI** server with asynchronous request handling  
- **Azure OpenAI GPT-5** for query understanding, tool invocation, and response generation  
- **OpenAI Vector Stores** for semantic search across uploaded course materials  
- **Canvas LMS API** integration for student-accessible resources  



## Demo & Links

- [Github](https://github.com/Deyu-Zhang/canvas_ai)
- [Demo](https://docs.google.com/presentation/d/1RhQeK-0zsjlltyZdZD8q8MCvjk9Y4iDnbsi4hZDU2Ak/edit?usp=sharing)

