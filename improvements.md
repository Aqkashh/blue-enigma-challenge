# Project Improvements Writeup

This document outlines the bugs I fixed to get the baseline system running and the "bonus innovation" improvements I implemented to build a more robust, modern, and user-friendly system.

## 1. Core Bug Fixes (Task 2)

I identified and fixed several critical bugs in the provided scripts to achieve baseline functionality:

* **Neo4j Connection Error:** The `load_to_neo4j.py` (and all other scripts) failed with an `ssl.SSLCertVerificationError`. I diagnosed this as an SSL certificate validation issue with the Neo4j Aura cloud instance.
    * **Fix:** I updated the `NEO4J_URI` in the configuration to use the `neo4j+ssc://` scheme. This tells the driver to establish a secure connection but to skip the certificate chain verification that was failing on my local machine.

* **Pinecone Client Error:** The `pinecone_upload.py` script failed with an `ImportError`. The project's `requirements.txt` used the deprecated `pinecone-client` package.
    * **Fix:** I uninstalled the old `pinecone-client` and installed the modern `pinecone` library. I then refactored `pinecone_upload.py` to use the new client syntax, fixing the initialization and upsert logic.

* **Baseline Prompt Engineering:** The original `hybrid_chat.py` prompt explicitly instructed the AI to "Cite node ids." This resulted in a poor user experience (e.g., "Visit Attraction 250").
    * **Fix:** I re-engineered this prompt to give the AI a persona ("expert Vietnam travel guide") and instructed it to **use the human-readable "name"** of attractions, not their IDs, which dramatically improved the quality of the baseline response.

## 2. Bonus Innovation (Task 3)

For the "Bonus Innovation" and "Design" points, I went beyond small fixes and refactored the entire chat logic into a new, superior `hybrid_chat_langchain.py` script.

* **Framework:** It's built on **LangChain Expression Language (LCEL)**, the modern, declarative way to build AI chains.
* **Secure Config:** It uses a `.env` file and the `dotenv` library to securely load all API keys into the environment, which is a best practice.
* **Hybrid RAG Pipeline:** The chain is a true hybrid retriever:
    1.  A `vector_retriever` fetches semantically similar docs from Pinecone.
    2.  A `RunnableLambda` function (`get_graph_context`) takes those docs, extracts their IDs, and queries Neo4j for related "graph facts."
    3.  A `RunnableParallel` block runs these context-gathering steps in parallel for efficiency.
    4.  The final prompt, LLM, and output parser combine everything into a high-quality, readable answer.
* **Modularity:** This new design is cleaner, more readable, maintainable, and scalable than the original monolithic script.