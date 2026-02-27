---
name: Presentation and Project Showcase
dependsOn:
  - documentation-and-architecture-decisions
tags: [chatbots, presentation, demo, capstone]
learningOutcomes:
  - Structure a 10–15 minute capstone presentation that demonstrates technical depth and practical impact.
  - Prepare answers for common technical questions about architecture, cost, scaling, and differentiation.
  - Identify what makes domain-specific chatbot projects successful as portfolio pieces.
---

## Learning Outcomes

- Structure a 10–15 minute capstone presentation that demonstrates technical depth and practical impact.
- Prepare answers for common technical questions about architecture, cost, scaling, and differentiation.
- Identify what makes domain-specific chatbot projects successful as portfolio pieces.

A capstone presentation is not a lecture about chatbot theory — it's a demonstration that you can build, deploy, and explain a production system. The audience wants to see it work, understand how it works, and be convinced that you made thoughtful engineering decisions. The most common mistake is spending too much time on slides and too little on the live demo. Lead with the demo, then explain the engineering behind it.

## Presentation Structure

A 10–15 minute presentation should follow this structure, with approximate time allocations.

**Introduction (2 minutes).** Start with the problem, not the technology. What gap does your chatbot fill? Who are the target users? What was their workflow before your chatbot existed, and how does it change with your chatbot? A concrete scenario is more compelling than an abstract description — "A new hire at a 500-person company spends their first week searching Confluence for answers to basic questions about benefits, IT setup, and HR policies. My onboarding assistant answers those questions instantly, with source citations" tells a story that the audience can immediately grasp.

**Live Demo (4 minutes).** This is the centrepiece of the presentation. Show a realistic conversation — not a contrived example, but a scenario that a real user would encounter. Demonstrate the key capabilities: ask a domain-specific question and show the RAG-powered response with source citations. Follow up with a related question and show that the chatbot remembers the conversation context. If you have an analytics dashboard, switch to it briefly to show real-time metrics.

Prepare a demo script with specific messages to send, so you're not improvising under pressure. Have a backup plan: a screen recording of the demo that you can play if the live version has technical issues. Nothing derails a presentation faster than a broken live demo with no fallback.

**Technical Deep Dive (5 minutes).** Walk through your architecture diagram. Explain the data flow: a user sends a message, it hits the API, the RAG engine retrieves relevant documents, the LLM generates a response grounded in those documents, the response is returned with streaming. Highlight 1–2 key technical decisions and why you made them — "I chose ChromaDB over Pinecone because the project has under 100,000 vectors and ChromaDB eliminated infrastructure complexity" shows engineering judgement. Discuss 1–2 challenges you faced and how you solved them — this is often the most impressive part of the presentation because it demonstrates problem-solving ability, not just tutorial-following.

Show one or two code snippets that illustrate an interesting implementation detail. Choose code that demonstrates something non-obvious — your re-ranking logic, your memory summarisation strategy, or your fallback handling. Keep the snippets short (10–15 lines) and explain them line by line.

**Results and Metrics (2 minutes).** Quantify your system's performance. "P95 latency is 2.4 seconds" is more credible than "it's pretty fast." Report cost per conversation, cache hit rate, and error rate. If you tested with real users, share their feedback. If you ran A/B tests, share the results. Numbers add credibility and show that you measured what you built, not just built it.

**Lessons Learned and Future Work (2 minutes).** Be honest about what went well and what you'd do differently. "Initially, RAG relevance was poor because my chunks were too large. After experimenting with chunk sizes, I found that 800-token chunks with 100-token overlap gave the best results" shows growth and analytical thinking. Describe 2–3 future enhancements you would prioritise: fine-tuning a smaller model on domain data, adding function calling for actions, or implementing multi-modal support.

**Q&A.** Open the floor for questions. Having prepared answers (covered in the next section) ensures you respond confidently.

**Presentation tips.** Start with the demo — hook the audience immediately with something tangible. Tell a story: the user's problem, your solution, the impact. Use visuals (architecture diagrams, screenshots, metrics charts) rather than bullet-point slides. Practise timing so you don't run over. Rehearse the demo at least three times.

## Preparing for Questions

Anticipating questions and preparing answers in advance makes the difference between a confident presentation and a stumbling one. The following questions come up in nearly every capstone presentation.

**"What makes your chatbot different from ChatGPT?"** This is the most important question because it gets at your project's value proposition. The answer should focus on domain specificity: "My chatbot is specialised for [domain] with RAG over [company/domain] documents. Unlike ChatGPT, it only references verified sources, eliminating hallucination for factual queries. It cites specific documents for transparency. It maintains conversation memory with user profiles. And it's customised with domain-specific prompts that guide the conversation toward actionable answers." The key is to articulate why a general-purpose chatbot is insufficient for your use case.

**"How do you handle hallucination?"** Multiple strategies work in concert. RAG grounds responses in retrieved documents. The system prompt explicitly instructs the model to only use provided context and to say "I don't have that information" rather than guessing. Post-processing can check for unsupported claims. And a user feedback loop identifies and flags cases where the model generates information not present in the source documents.

**"What's the cost per conversation?"** Have a specific number: "Approximately $0.002 per conversation on average, or $20 per 10,000 conversations. I optimised costs through prompt compression (30% reduction in token usage), semantic caching (40% hit rate on repeated queries), conversation summarisation (keeping history under 1,000 tokens), and routing simple queries to GPT-3.5-turbo instead of GPT-4." The specificity of the answer — actual percentages, actual dollar figures — demonstrates that you've measured and optimised, not just built.

**"How does it scale?"** Describe your architecture: "The backend is stateless — any server instance can handle any request because conversation state is stored in Redis. A load balancer distributes traffic across multiple instances. I've tested with 100 concurrent users successfully. Scaling to thousands of concurrent users requires adding more backend instances, which is a configuration change, not a code change." If you implemented rate limiting and backpressure, mention those as well.

**"What were your biggest challenges?"** Be honest. "Initially I struggled with token management — long conversations overflowed the context window until I implemented summarisation. RAG relevance was poor until I added re-ranking. I hit Docker networking issues during deployment that taught me about bridge networks and service discovery." Challenges demonstrate learning, and specific, technical answers show that you understood and resolved the issues rather than just working around them.

**"How would you improve it given more time?"** Have three concrete priorities: "First, fine-tune a smaller model on domain data to reduce cost and improve accuracy for domain-specific queries. Second, add function calling so the chatbot can take actions — book appointments, check order status, update preferences — rather than just answering questions. Third, implement multi-modal support to handle images and documents uploaded by users."

**"What's the security model?"** Describe the layers: "JWT authentication protects API access. Content moderation via the OpenAI Moderation API filters unsafe inputs and outputs. PII detection and redaction prevents sensitive data from being logged. Rate limiting prevents abuse. HTTPS encrypts data in transit. API keys are stored in a secret manager, never in code."

## Project Showcase: What Makes a Successful Capstone

Looking at successful chatbot projects reveals patterns in what makes them impressive — not as portfolio decoration, but as demonstrations of engineering skill.

A **legal research assistant** built RAG over case law and legal documents. Its distinguishing features were citation extraction from legal texts, jurisdiction-aware retrieval that prioritised cases from the relevant jurisdiction, and precedent linking that connected related cases. The measured impact was a 60% reduction in research time for the tasks it supported.

A **medical literature chatbot** built RAG over PubMed research papers. It implemented multi-hop reasoning across papers — synthesising information from multiple sources to answer questions that no single paper addressed. It added confidence scoring for medical claims and ranked references by journal impact factor. It helped doctors find relevant research in minutes instead of hours.

A **company onboarding assistant** built RAG over HR policies, benefits documentation, and IT setup guides. It implemented role-based information filtering so that different departments saw relevant information. It generated automated task lists for new hires and integrated with Slack for convenient access. It improved new hire experience scores by 40%.

An **e-commerce shopping assistant** built RAG over a product catalogue. It generated comparison tables for similar products, made budget-aware recommendations, and integrated with inventory systems to check availability in real time. It achieved a 25% increase in conversion rate.

What these projects have in common is not technical sophistication — some used basic RAG while others implemented advanced features. What they share is domain specificity (they solved particular problems, not general chat), measurable impact (they quantified the value they provided), and professional presentation (clear documentation, architecture diagrams, and polished demos). A well-executed simple project is more impressive than a poorly executed complex one.

## Your Project as a Portfolio Piece

Your capstone project has value beyond the course. Consider open-sourcing it with thorough documentation — a well-documented open-source project on your GitHub profile demonstrates both technical skill and communication ability. Write a blog post explaining your approach, the challenges you faced, and the decisions you made — this is valuable both for your own learning (writing forces clarity of thought) and as a discoverable record of your expertise. Present the project at a local meetup or tech community event. Use it as a talking point in job interviews — walking an interviewer through your architecture diagram and explaining your design decisions demonstrates exactly the kind of systematic thinking that engineering teams value. And if it solves a real problem for real users, deploy it and iterate based on feedback — nothing teaches like production traffic.
