# Project 1 Planning: The Unofficial Guide

> Write this document before you write any pipeline code.
> Your spec and architecture diagram are what you'll use to direct AI tools (Claude, Copilot, etc.) to generate your implementation — the more specific they are, the more useful the generated code will be.
> Update the Retrieval Approach and Chunking Strategy sections if you change your approach during implementation.
> Update this file before starting any stretch features.

---

## Domain

To celebrate the World Cup starting this Thursday, June 11th, the domain I chose is the World Cup! 
The plan is create an Unofficial guide for visitors and anyone looking to go to a soccer game this world cup. 

This domain covers alot. Topics include; accomodation cost, buying tickets safely online, FanFest information for Miami, first time WC fan tips, travel readiness for visitors coming to the US for the World Cup among other topics. This information is scattered and not central to one location.  
---

## Documents

| # | Source | Description | URL or location |
|---|--------|-------------|-----------------|
| 1 | Reddit |Accommodation Costs for WC | https://www.reddit.com/r/WorldCup2026Tickets/comments/1tkgter/how_are_people_handling_accommodation_costs_for/
| 2 | Reddit |Buying/Trading Tickets online|https://www.reddit.com/r/WorldCup2026Tickets/comments/1r6ixzv/how_to_buytrade_a_ticket_safely_off_of_someone/
| 3 |Website|Fan Fest Miami |https://miamifwc26.com/fan-festival/ |
| 4 |Reddit |First time WC tips| https://www.reddit.com/r/worldcup/comments/1u08jfq/whats_one_mistake_firsttime_world_cup_2026_fans/ |
| 5 |Reddit|Getting to Hard Rock Stadium|https://www.reddit.com/r/Miami/comments/1pncikp/getting_to_hard_rock_stadium/ |
| 6 |Reddit |What to expect in opening matches|https://www.reddit.com/r/worldcup/comments/1ts998n/to_those_who_have_attended_opening_matches_of_the/|
| 7 |Website|Ranking the 16 FIFA World Cup Host Cities |https://www.si.com/soccer/ranking-16-fifa-world-cup-host-cities|
| 8 |Reddit |SF WC Stadium Info |https://www.reddit.com/r/worldcup/comments/1q999bd/world_cup_2026_information_for_people_going_to_sf/|
| 9 |Website|Attending a WC |https://theworldcupguide.com/steps-to-attend-world-cup/ |
| 10 |Website |Travel Readiness for Travelers entering the US|https://www.envoyglobal.com/insight/fifa-world-cup-2026-final-travel-readiness-guide-for-visitors-entering-the-us/ |
| 11 | Website| What to know before going to a WC game| https://seatgeek.com/blog/going-to-a-world-cup-match-heres-what-to-know-before-you-go

---

## Chunking Strategy
I will use the fixed-sized chunking with overlap strategy. 

**Chunk size:**
The chunk size I will use is 180 words.

**Overlap:**
30 word overlap (~ 17%)

**Reasoning:**
The MiniLM-L6-v2, only reads the first 256 tokens (~ 190 words) of any chunk. My documents are World Cup guides full of proper nouns (e.g. Estadio Azteca, MetLife Stadium) so I figured 180 is better than 190 to leave a cushion and doesn't pass the limit. 

My documents are long form guides with threads in some cases. One complete thought runs about a paragraph (~ 100 - 180 words). Chunks smaller than that don't work that well for this project.  The 30 word overlap allows for full sentences and facts to not get split in half.

**Corpus Check:**

My source files total ~ 24,900 words. At a 150 word effective stride ( 180 word chunks minus 30 word overlap), I expect ~165 chunks which is within the range (50-2,000), confirming the chunk size is not producing too few oversized chunks or too many fragments.
---

## Retrieval Approach

**Embedding model:**

The embedding model I'm using is all-miniLM-L6-v2. This model will run locally on my machine and is a solid option for semantic search retrieval. I would load all-miniLM-L6-v2 via the sentence transformers library, which is the tool that runs the model and produces the embeddings.

**Top-k:**
For Top K, I'd select 5. Selecting too few risks the card not being in the set, too many cards buries the good card among loosely related ones and distracts the LLM. I selected 5 because my questions sometimes pull from different files so a K=5 gives a little extra coverage without flooding the context.

**Production tradeoff reflection:**
For production, I would definitely make it multilingual, we have visitors coming from all over the world and if we could use sources in other languages and be able to use multilingual embedding models to be able to serve non-english knowledge, it would be production ready.

I would also use a larger model with a larger token ceiling that allows larger chunks without slicing so much. 

I could also use larger API models that generally do better retrieval on domain specific text. 
---

## Evaluation Plan

| # | Question | Expected answer |
|---|----------|-----------------|
| 1 |What do fans say about accommodation costs during WC 2026?|Fans recommend staying farther from the stadium and using rideshare or public transit to save money, since places within walking distance charge a premium. A 20–30 minute Uber or public transit ride is considerably cheaper than the markups on nearby lodging. Most venues open hours before kickoff, so arriving early avoids the worst price pressure|
| 2 |How do fans recommend getting to Hard Rock stadium?|Fans recommend using Uber/rideshare rather than driving to the stadium directly. The common advice is to Uber to a parking lot or shuttle point and take a shuttle in, because catching an Uber from the stadium after the match is difficult (everyone leaves at once). Leaving from a lot is easier since crowds disperse gradually rather than all at once.|
| 3 |What precautions do fans suggest when buying resale tickets?|Fans suggest a verification-heavy process when buying resale tickets from individuals: do a live video call (no text-only deals), confirm the seller's ID matches the ticket account, have them screen-share to show the tickets live inside the portal (not screenshots), check their social media and LinkedIn for a legitimate identity, and stay on the call through the full payment-and-transfer so money and ticket exchange happen together. Record the process for proof. Avoid irreversible payment methods (Zelle, e-transfer, PayPal Friends & Family). Walk away if the seller refuses transparency — legitimate sellers will verify themselves.|
| 4 |What should international visitors prepare before entering the US?|International visitors from Visa Waiver Program (VWP) countries can enter without a traditional visa if they hold an approved ESTA linked to a valid passport and limit their stay to 90 days for tourism. Travelers not eligible for the VWP need a B-1/B-2 visitor visa. Even with a valid visa, entry isn't guaranteed — all travelers are subject to CBP inspection on arrival. The FIFA Pass lets 2026 ticket holders interview for a U.S. visa before the tournament.|
| 5 |How do the 16 host cities compare for fans deciding where to go? |A complete comparison of all 16 host cities and the factors used to rank them (e.g., atmosphere, accessibility, things to do, match significance).|

---

## Anticipated Challenges

1. City name collisions: Cities like Miami, Dallas, etc appear across many files, so a Miami query might pull a chunk that mentions Miami but it really is about Dallas. 

2. Sarcasm in fan text/comments: If a comment on Reddit is sarcastic (e.g. OH great, $1,000 resale tickets) it may be embedded as positive misleading a cost effectiveness query.

3. For opinion based questions, there is no single ground truth, fans give varied suggestions. I evaluate these by whether the system faithfully represents the consensus/main recommendations in my documents, rather than matching one exact answer.Conflicting advice across sources may produce an incomplete or one sided answer depending on which chunks are retrieved. 

---

## Architecture
````mermaid
flowchart LR
    A[Document Ingestion: 11 txt files + cleaning] --> B[Chunking: fixed-size 180 words 30 overlap]
    B --> C[Embedding + Vector Store: all-MiniLM-L6-v2 sentence-transformers ChromaDB]
    C --> D[Retrieval: semantic search top-k 5]
    D --> E[Generation: Groq llama-3.3-70b-versatile + source citation]
````
---

## AI Tool Plan

**Milestone 3 — Ingestion and chunking:**
For this milestone, I plan on using Claude. I'll give Claude the chunking strategy section and a sample document and ask it to write chunk_text() function. I expect code that produces 165 source tagged chunks. I'll verify by running the code and check the count is ~165 and read 5 chunks. 
**Milestone 4 — Embedding and retrieval:**
For this milestone, I plan on using Claude. I will give it my Retrieval Approach section and ask it to write the embedding step and a retrieve() function. I expect code that embed my chunks, stores them in ChromaDB with source metadata and returns the top 5 chunks with their source and distance score. I'll verify by running 3 of my evaluation questions and checking the returned chunks are on topic with distances under 0.5
**Milestone 5 — Generation and interface:**
For this milestone, I'll use Claude. I'll give it my grounding requirement and ask it to wire the Groq llama-3.3-70b-versatile call plus a Gradio interface. I expect a system prompt that enforces grounding, a function that returns an answer with its source attribution and a Gradio UI with a question box and answer plus sources output. I'll verify by asking an out-of-scope question and confirm the system refuses instead of inventing an answer. 
