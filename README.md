# Movie Agent: AI-Powered Movie Discovery Using Google's Agent Development Kit (ADK)

## Problem Statement
Users often leave platforms like Netflix or YouTube frustrated, not because great content is missing, but because they simply don't know it exists. Discovering the right movie can be challenging with traditional search and recommendation systems.

## Why Agents?
AI agents excel at exploring complex connections between movies, explaining why a film is great, and recommending similar ones based on nuanced reasoning—something static algorithms struggle with.

## Solution
**Movie Agent** is a multi-agent system built around the `movie_catalog_agent`, demonstrating real-world Agent-to-Agent (A2A) collaboration using Google's Agent Development Kit (ADK) and the open A2A protocol.

This project showcases a production-grade pattern where a specialist backend agent securely shares its knowledge with a customer-facing agent.

## High-Level Architecture

| Component                  | Role                                                                 | Technology Used                          | Runs Where                  |
|----------------------------|----------------------------------------------------------------------|------------------------------------------|-----------------------------|
| **Movie Catalog Agent**    | Specialist agent owning the movie database and exact details         | ADK LlmAgent + Gemini-1.5-flash-lite     | Separate background process (port 8001) |
| **get_movie_info() tool**  | Simple Python function (mock DB) returning actor + one-line plot     | Plain Python function                    | Inside Catalog Agent        |
| **A2A Exposure Layer**     | Wraps Catalog Agent and exposes it via standard A2A protocol         | `to_a2a()` → auto-generates FastAPI + agent card | Uvicorn server @ localhost:8001 |
| **Agent Card**             | "Business card" of the agent (name, description, skills, endpoint)   | Auto-generated JSON                      | Served by Catalog server    |
| **RemoteA2aAgent**         | Client-side proxy making the remote Catalog Agent feel local         | ADK RemoteA2aAgent                       | Inside the notebook         |
| **Customer Support Agent** | Front-facing agent that interacts directly with users                 | ADK LlmAgent + Gemini-1.5-flash-lite      | Inside the notebook         |
| **ADK Runner + Session**   | Orchestrates execution and maintains conversation state              | Runner + InMemorySessionService          | Inside the notebook         |

## End-to-End Flow
When a user asks a question (e.g., "Tell me about Sholay"):

1. **User** → sends query to **Customer Support Agent** (Gemini)
2. Support Agent realizes it needs exact details → calls **RemoteA2aAgent** proxy
3. Proxy discovers the Agent Card at `http://localhost:8001/.well-known/agent-card.json`
4. Sends A2A task request to **Movie Catalog Agent** server
5. Catalog Agent executes `get_movie_info("sholay")` and returns result via A2A
6. Response flows back to Support Agent
7. Support Agent composes a friendly, accurate reply
8. **User** receives the answer with correct actors and plot

## Key Architecture Benefits
- **Clean separation of concerns**: Movie data lives only in the Catalog Agent (single source of truth)
- **Zero-code integration**: Support Agent uses the remote agent as if it were local—no manual HTTP/JSON handling
- **Interoperability by design**: Works with any A2A-compliant agent, regardless of host
- **Production-ready**: Easily deployable in real-world scenarios (e.g., Catalog on Cloud Run, Support on streaming platform backend)

### One-Sentence Summary
A scalable two-agent microservice architecture for LLMs: a specialist backend agent exposes knowledge via the open A2A protocol, while a customer-facing agent consumes it transparently—perfect for building modular, interoperable AI applications at scale.

## Future Improvements (If I Had More Time)
- Connect to a real movie database (e.g., TMDB API)
- Deepen story-type analysis for even better personalized recommendations

Built as part of the Google x Kaggle AI Agents Intensive Capstone Project 🚀
