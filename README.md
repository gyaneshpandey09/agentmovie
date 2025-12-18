#Problem Statement -- Users often bounce off from platforms like Netflix/YouTube for lack of good content to watch. Often the problem is not that interesting content is not there but the user does not know about it.
#Why agents?# -- Agents can really help mapping all possible connections between movies or answer questions about why a movie is great and what other movies might exist like one.
#Solution# -- Core to Movie Agent is movie_catalog_agent, an example of a multi-agent system leveraging a2a protocol.
#The Build# -- This agent has been built using Google's ADK.
This notebook demonstrates a real-world, production-grade Agent-to-Agent (A2A) collaboration pattern using Google’s Agent Development Kit (ADK) and the open A2A protocol.

#High-Level Architecture Overview of the Movie Agent Notebook#
This notebook demonstrates a real-world, production-grade Agent-to-Agent (A2A) collaboration pattern using Google’s Agent Development Kit (ADK) and the open A2A protocol.

#Component	Role	Technology Used	Runs Where#
Movie Catalog Agent	Specialist agent that owns the movie database and knows exact movie details	ADK LlmAgent + Gemini-2.5-flash-lite	Separate background process (port 8001)
get_movie_info() tool	Simple Python function (mock DB) that returns actor + one-line plot	Plain Python function	Inside Catalog Agent
A2A Exposure Layer	Wraps the Catalog Agent and exposes it via the standard A2A protocol	to_a2a() → auto-generates FastAPI + agent card	Uvicorn server @ localhost:8001
Agent Card	“Business card” of the agent (name, description, skills, endpoint)	Auto-generated JSON at /.well-known/agent-card.json	Served by Catalog server
RemoteA2aAgent	Client-side proxy that makes the remote Catalog Agent feel local	ADK RemoteA2aAgent (reads the agent card)	Inside the notebook
Customer Support Agent	Front-facing agent that talks directly to humans	ADK LlmAgent + Gemini-2.5-flash-lite	Inside the notebook
ADK Runner + Session	Orchestrates execution, maintains conversation state	Runner + InMemorySessionService	Inside the notebook

#End-to-End Flow# (what actually happens when a user asks a question)
User
  ↓ (text query)
Customer Support Agent (Gemini)
  ↓ “I don’t know the exact plot → I need the catalog”
→ calls RemoteA2aAgent (proxy)
      ↓ discovers agent card at http://localhost:8001/.well-known/agent-card.json
      ↓ sends A2A task request (HTTP POST /tasks)
          Movie Catalog Agent Server (uvicorn @ port 8001)
              ↓ executes get_movie_info("sholay")
              ↓ returns result via A2A protocol
      ← response travels back
  ← RemoteA2aAgent gives result to Customer Support Agent
  ↓ Support Agent composes friendly reply
User gets accurate answer with correct actor + plot

#Architecture#
Clean separation of concerns
Movie data lives only in the Catalog Agent → single source of truth.

Zero-code integration
The Support Agent never imports HTTP calls or parses JSON — it just uses remote_movie_catalog_agent as if it were a local sub-agent.

Interoperability by design
Any other company or team could host their own Catalog Agent and the Support Agent would work instantly just by pointing to a different agent-card URL.

Production-ready pattern
In real deployments:

Catalog Agent → hosted by the movie vendor (Cloud Run, Agent Engine, etc.)
Support Agent → hosted by the streaming platform
They collaborate securely over the internet using only the open A2A protocol.
One-sentence summary
It’s a two-agent microservice architecture for LLMs: one specialist backend agent (Catalog) exposes its knowledge via the standard A2A protocol, and one customer-facing agent (Support) agent consumes it transparently — exactly how modern LLM applications will be built at scale.

If I had more time, this is what I'd do
Connect to a real movie database
Go deeper into story types to recommend better movies
