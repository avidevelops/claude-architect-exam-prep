![Contributions Welcome](https://img.shields.io/badge/contributions-welcome-brightgreen)

# Claude Certified Architect – Foundations: Exam Prep Guide

## About This Guide
This guide is compiled from a detailed Q&A thread analyzing exam-style questions for the **Claude Certified Architect – Foundations** certification. It breaks down multi-agent architectures, context management, tool design, and batch processing principles, focusing heavily on architectural tradeoffs and best practices over raw prompt engineering.

## How the Exam Works
The Claude Architect Foundations exam tests your ability to design resilient, production-ready AI systems. It consists of scenario-based multiple-choice questions focusing on **Domain 1: Agentic Architecture**, **Domain 4: API & Orchestration**, and **Domain 5: Context Management**. The exam prioritizes structural, deterministic solutions (like schema design and tool boundaries) over probabilistic approaches (like prompt instructions). 

## Scenarios Covered
This guide covers core architectural scenarios you are likely to encounter, including:
- **Agentic Architectures**: Subagent spawning, parallelism, and workflow decomposition.
- **Context Management**: Mitigating "lost in the middle" effects, context bloat, and preserving provenance through summarization.
- **Tool & Schema Design**: Enforcing business rules, standardizing outputs, and designing machine-readable identifiers for tool chaining.
- **Batch Processing & State**: Resuming crashed sessions, optimizing SLA buffers, and handling partial batch failures.

---

## Questions & Answers

### Q1: Handling Semantic Errors in Extraction Totals
> *Scenario: Schema Design & Self-Correction*  


**Question:**  
An automated invoice extraction pipeline occasionally outputs structured JSON where the extracted line items do not add up to the total amount extracted from the invoice. What is the best architectural approach to handle this semantic error?

**Options:**  
- A) Add a `calculated_total` field alongside the `stated_total` field, compare them, and flag mismatches for human review.  
- B) Automatically adjust the line item values so they mathematically sum to the stated total.  
- C) Introduce a secondary LLM step to reconcile the math errors.  
- D) Add more few-shot examples of correct math to the prompt.  

**✅ Correct Answer: A — Extract `calculated_total` and flag discrepancies**

**Why this is correct:**  
JSON schemas prevent syntax errors but not semantic errors (like bad math). The most robust self-correction pattern is extracting both what the document explicitly states (`stated_total`) and what the model calculates from the line items (`calculated_total`). If they don't match, the record is flagged for human review. This improves reliability without fabricating data.

**Why the others are weaker:**  
- **B)** Adjusting line items invents values not supported by the document, corrupting data integrity.  
- **C)** Adds unnecessary architectural overhead when explicit deterministic validation is available.  
- **D)** Few-shot examples cannot guarantee deterministic consistency for arithmetic.  

**💡 Exam Takeaway:**  
Design self-correction flows by extracting both a stated value and a calculated value, routing mismatches to human review rather than silently "fixing" source document errors.

---

### Q2: Scaling Prompt Refinement vs. Batch Processing
> *Scenario: Batch Processing & Cost Optimization*  


**Question:**  
You are preparing to process 50,000 legacy documents using the Batch API. An initial test on 500 documents reveals that 18% of them require 2-3 prompt refinements to extract data correctly. What is the most cost-efficient strategy for scaling this workload?

**Options:**  
- A) Refine the prompt interactively on a representative sample to maximize first-pass success, then process all 50,000 documents via the Batch API.  
- B) Use the Batch API to process all 50,000 documents immediately, identify failures at scale, and resubmit them.  
- C) Process the 50,000 documents using the synchronous API to handle prompt refinement dynamically per document.  
- D) Begin submitting 5,000-document batches to incrementally learn failure modes in production.  

**✅ Correct Answer: A — Refine interactively first, batch process later**

**Why this is correct:**  
The guide explicitly recommends refining prompts on a representative sample *before* batch-processing large volumes. Iterative resubmissions at scale destroy the cost savings of the Batch API. By fixing the failure modes on a localized sample, you ensure the prompt is robust, maximizing first-pass success when you finally trigger the 50,000-document Batch run.

**Why the others are weaker:**  
- **B)** Paying for prompt learning on production-sized batches results in massive, expensive resubmission cycles.  
- **C)** Synchronous API throws away the 50% cost savings of the Batch API.  
- **D)** Still pays for iterative failure learning at a high scale.  

**💡 Exam Takeaway:**  
Refine prompts on a representative sample first to maximize first-pass success, then run the full volume through the Batch API to minimize expensive resubmissions.

---

### Q3: Batch API Scheduling for SLAs
> *Scenario: SLA Management & Async APIs*  


**Question:**  
Your system processes asynchronous user requests with a strict Service Level Agreement (SLA) requiring results within 30 hours of submission. You plan to use the Message Batches API, which can take up to 24 hours to complete. Which batch submission schedule best meets the SLA while maximizing cost efficiency?

**Options:**  
- A) Submit batches every 6 hours.  
- B) Submit one large batch at the end of each day.  
- C) Use the synchronous API instead to guarantee the SLA.  
- D) Submit batches every 4 hours.  

**✅ Correct Answer: D — Submit batches every 4 hours**

**Why this is correct:**  
Worst-case total turnaround time is the time a document waits for the next batch window *plus* the maximum batch processing time (24 hours). If you submit every 4 hours, a document arriving right after a cutoff waits 4 hours + 24 hours processing = 28 hours. This leaves a 2-hour safety buffer under the 30-hour SLA.

**Why the others are weaker:**  
- **A)** 6 hours wait + 24 hours processing = exactly 30 hours. This leaves zero buffer, risking SLA breaches.  
- **B)** End-of-day (24h) wait + 24h processing = 48 hours, failing the SLA entirely.  
- **C)** Unnecessarily sacrifices the Batch API's 50% cost discount.  

**💡 Exam Takeaway:**  
Calculate worst-case SLA by adding batch frequency interval to the 24-hour max processing time; always leave an operational buffer.

---

### Q4: Schema Design for Conflicting Source Truths
> *Scenario: Schema Engineering & Provenance*  


**Question:**  
An extraction pipeline processes technical manuals. A specific manual lists two conflicting battery capacities: one in the text and a different one in a detailed specs table. Historical data shows the specs table is correct 90% of the time. How should the extraction schema handle this?

**Options:**  
- A) Halt processing and flag the document for manual correction before extraction.  
- B) Use a single-value schema and prompt the model to pick the most likely correct value.  
- C) Hard-code a rule to always extract the value from the specs table.  
- D) Change the field to capture all conflicting values along with their source locations to preserve provenance for downstream reconciliation.  

**✅ Correct Answer: D — Capture all values with source locations**

**Why this is correct:**  
Forcing premature collapse into a single value is an anti-pattern. If the specs table is only right 90% of the time, hard-coding a preference guarantees a 10% error rate. Modifying the schema to accept an array of values with explicit source locations preserves full provenance, allowing downstream business logic or human reviewers to make an informed reconciliation.

**Why the others are weaker:**  
- **A)** Impractical for automated pipelines operating on legacy documents.  
- **B)** A single-value schema forces the LLM to destroy evidence of the conflict.  
- **C)** A heuristic rule is brittle and will output wrong data 10% of the time.  

**💡 Exam Takeaway:**  
Preserve conflicting source data in the structured output instead of forcing premature collapse into a single value.

---

### Q5: Output Formats for Tool Chaining
> *Scenario: Tool Interfaces & Identifiers*  


**Question:**  
An agent uses a `search_documents` tool to find files, and subsequently uses `share_document(document_id, email)` and `move_document(document_id, folder)` to act on them. How should the `search_documents` tool format its output to ensure reliable chaining?

**Options:**  
- A) Return clickable human-readable URLs.  
- B) Return structured data containing `document_id` and metadata for each result.  
- C) Return detailed prose summaries of the document contents.  
- D) Return a simple list of document titles.  

**✅ Correct Answer: B — Structured data with document IDs**

**Why this is correct:**  
Multi-step workflows require clear input/output contracts. Because the downstream tools (`share`, `move`) require a specific machine-usable identifier (`document_id`), the upstream `search` tool must return exactly that ID in a structured format alongside the human-readable metadata.

**Why the others are weaker:**  
- **A)** Clickable URLs are for humans. The agent would have to infer or parse IDs from the string.  
- **C)** Prose summaries do not provide the exact programmatic identifiers needed for the next API call.  
- **D)** Titles are ambiguous and cannot be passed directly into an ID-based API.  

**💡 Exam Takeaway:**  
Multi-step tool workflows require machine-usable identifiers; tools meant for chaining should always return structured data containing explicit IDs.

---

### Q6: The Primary Advantage of Structured Output
> *Scenario: Agentic Tool Integration*  


**Question:**  
When designing agentic workflows, what is the primary advantage of enforcing structured JSON output for tool responses and agent outputs?

**Options:**  
- A) It makes the LLM's reasoning process perfectly deterministic.  
- B) It ensures the semantic truth of the data fetched from backend APIs.  
- C) It significantly reduces token consumption compared to standard text.  
- D) It allows the agent and downstream systems to reliably access specific fields directly without parsing free-form text.  

**✅ Correct Answer: D — Reliable, direct field access without parsing**

**Why this is correct:**  
Structured outputs provide a stable schema contract. Instead of using regex or fuzzy text parsing to extract an account number or currency amount from a verbose prose response, downstream systems (and subsequent agent tools) can directly reference the exact JSON keys. This drastically reduces integration errors in chained operations.

**Why the others are weaker:**  
- **A)** JSON enforces the *shape*, but LLM reasoning remains fundamentally probabilistic.  
- **B)** Schema checks validate format, not semantic correctness (truth).  
- **C)** JSON often *increases* token overhead due to bracket/key repetition.  

**💡 Exam Takeaway:**  
Structured output gives agents reliable field access, preventing downstream errors associated with parsing free-form text.

---

### Q7: Dynamic Scoping to Solve Tool Selection Overload
> *Scenario: Context & Tool Management*  


**Question:**  
An agent has access to over 50 different API connector tools. During execution, it frequently selects the wrong connector, even when explicitly instructed to search first. What is the most effective architectural change to fix this tool selection failure?

**Options:**  
- A) Rewrite the tool descriptions for all 50 connectors to be more detailed.  
- B) Combine all 50 connectors into a single monolithic API call.  
- C) Implement better error handling so the agent can recover after selecting the wrong connector.  
- D) Provide a `search_connectors` tool that dynamically scopes the available tool set, exposing only the relevant matched connectors to the agent.  

**✅ Correct Answer: D — Dynamically scope the available tool set**

**Why this is correct:**  
Exposing 50+ tools simultaneously degrades the model's decision-making accuracy (decision complexity). The architectural fix is to enforce dynamic scoping: the agent first calls `search_connectors`, and the system responds by injecting *only* the 2-3 matched, highly relevant connector tools into the agent's context for the next turn. 

**Why the others are weaker:**  
- **A)** Better descriptions help, but 50+ active tools still cause systemic cognitive overload.  
- **B)** A monolithic API removes the agent's ability to inspect parameter requirements and choose correctly among subtle variants.  
- **C)** Error handling is a reactive band-aid; dynamic scoping prevents the error proactively.  

**💡 Exam Takeaway:**  
Reduce decision complexity by dynamically exposing only relevant tools to the agent, rather than dumping 50+ tools into the context window simultaneously.

---

### Q8: Constraining Tool Inputs with Enums
> *Scenario: Tool Schema Engineering*  


**Question:**  
An agent needs to query specific internal databases, but users often refer to them using ambiguous natural language (e.g., "research database" instead of "db_res_01"). How should the tool's input schema be designed to handle this reliably?

**Options:**  
- A) Use an `enum` parameter explicitly listing the allowed database system names.  
- B) Use a freeform string parameter and use backend fuzzy matching to find the right database.  
- C) Allow freeform strings but reject the tool call at runtime if the name is incorrect.  
- D) Default to searching all databases simultaneously if the user is ambiguous.  

**✅ Correct Answer: A — Use an enum parameter**

**Why this is correct:**  
Enums create a strict, constrained input contract. Providing an enum of exact backend values in the tool schema allows the model to leverage its semantic understanding to map the user's messy natural language ("research database") to the strict programmatic requirement ("db_res_01") *before* the tool executes.

**Why the others are weaker:**  
- **B)** Pushes ambiguity into the backend, making behavior unpredictable.  
- **C)** Rejection wastes an execution turn and increases latency when validation could be upfront.  
- **D)** Searching everything spikes cost, latency, and context bloat.  

**💡 Exam Takeaway:**  
Use `enum` fields in tool schemas to map ambiguous natural language to strict backend values, improving tool use reliability deterministically.

---

### Q9: Pagination Design in Agent Tools
> *Scenario: Tool Output Optimization*  


**Question:**  
A search tool automatically fetches and returns all matching records from a database. This frequently causes severe latency and context bloat, as most agent tasks only need the first few results. What is the best way to redesign this tool's output?

**Options:**  
- A) Silently limit the results to the top 5 most relevant hits.  
- B) Return the first page of results along with pagination metadata (e.g., total count, cursor) so the agent can fetch more if needed.  
- C) Create a separate `fetch_next_page` tool for the agent to use.  
- D) Add a `max_pages` parameter to let the agent decide how many pages to fetch internally.  

**✅ Correct Answer: B — First page + explicit pagination metadata**

**Why this is correct:**  
Returning hundreds of records wastes context and time. The optimal design is to return just the first page (e.g., 10 records) alongside metadata like `total_matches` and a `next_cursor`. This gives the agent the situational awareness to decide if it has enough information, or if it needs to pass the cursor back into the search tool to get page 2.

**Why the others are weaker:**  
- **A)** Silently truncating results hides potentially vital information from the agent without warning.  
- **C)** Clutters the toolset; pagination should be a parameter of the existing search tool.  
- **D)** Still encourages hidden internal fetching and multi-page latency.  

**💡 Exam Takeaway:**  
Return the first page with total match count and a cursor; give the agent explicit pagination state instead of incurring massive multi-page latency by default.

---

### Q10: Assessing Trust in MCP Annotations
> *Scenario: Security & Model Context Protocol*  


**Question:**  
A third-party Model Context Protocol (MCP) server provides tools annotated with `readOnlyHint=true`. You are designing the user confirmation flow for your agent application. How should you treat these tool annotations?

**Options:**  
- A) Trust the annotations automatically because the MCP server runs locally.  
- B) Treat the annotations as untrusted metadata, and base your confirmation bypass policy on your trust of the vendor/server, not the self-reported labels.  
- C) Infer the server's trustworthiness by testing the tools in a sandbox first.  
- D) Bypass user confirmations safely, as the `readOnlyHint` guarantees no destructive actions can occur.  

**✅ Correct Answer: B — Treat annotations as untrusted metadata**

**Why this is correct:**  
Tool annotations like `readOnlyHint` or `destructiveHint` are entirely self-reported by the MCP server. If the server itself is unvetted or third-party, its metadata cannot be trusted as a security boundary. Confirmation bypass policies should be built on explicit trust of the vendor providing the server, not just taking the server's word for it via annotations.

**Why the others are weaker:**  
- **A)** Local execution does not mean the code/server is trustworthy.  
- **C)** Sandbox behavior does not guarantee malicious capabilities aren't hidden.  
- **D)** `readOnlyHint` is just a label, not a cryptographic or system-level guarantee.  

**💡 Exam Takeaway:**  
Treat MCP annotations as untrusted self-reported metadata unless the server itself is explicitly trusted by your system.

---

### Q11: Removing Unnecessary Tool Dependencies
> *Scenario: Tool Interface Design*  


**Question:**  
An agent frequently executes a two-step sequence: it calls `get_property_details(property_id)` to find an address, then passes that address to `get_neighborhood_info(address)`. This chaining increases latency and failure rates. How should the tool design be improved?

**Options:**  
- A) Merge both tools into a single `get_all_property_data` tool.  
- B) Modify `get_neighborhood_info` to accept `property_id` directly and resolve the address internally.  
- C) Improve the prompt to ensure the agent extracts the address more reliably.  
- D) Create a middle-tier helper tool to manage the data handoff.  

**✅ Correct Answer: B — Internalize predictable resolution steps**

**Why this is correct:**  
Forcing the agent to orchestrate a purely mechanical ID-to-string lookup wastes tokens, adds a failure point, and increases latency. If `get_neighborhood_info` frequently logically follows property identification, updating its interface to accept the `property_id` and handle the address lookup internally abstracts the complexity away from the LLM.

**Why the others are weaker:**  
- **A)** Over-consolidates; neighborhood data and property data are still distinct capabilities.  
- **C)** Prompt improvements don't fix the architectural latency of sequential tool calls.  
- **D)** Adds more surface area and preserves the chaining flaw.  

**💡 Exam Takeaway:**  
Design tools to internalize predictable dependencies (like ID lookups) to reduce avoidable chaining, latency, and failure coupling.

---

### Q12: Normalizing Heterogeneous Tool Outputs
> *Scenario: External Integrations & Tool Responses*  


**Question:**  
An agent tracks shipments using tools that call multiple different shipping carrier APIs. Each carrier returns timestamps, statuses, and delay codes in completely different JSON formats. How should you design the tool output provided to the agent?

**Options:**  
- A) Pass the raw JSON to the agent and provide extensive prompt instructions on how to parse each carrier's format.  
- B) Normalize the carrier responses into a single common schema (e.g., `status`, `estimated_delivery`) before returning it to the agent.  
- C) Create separate tracking tools for each carrier to keep the raw schemas distinct.  
- D) Return both the normalized schema and the full raw response to maximize context.  

**✅ Correct Answer: B — Normalize responses into a common schema**

**Why this is correct:**  
Agents reason best over consistent, predictable data structures. Dumping heterogeneous, carrier-specific schemas into the context forces the LLM to expend attention on parsing formats rather than solving the user's core problem (e.g., determining delivery status). Normalizing the output in your middleware abstracts away vendor quirks.

**Why the others are weaker:**  
- **A)** Pushes parsing logic into the prompt, making reasoning brittle.  
- **C)** Bloats the tool set unnecessarily.  
- **D)** Adding raw output just adds noise and consumes context window space when the agent only needs the core fields.  

**💡 Exam Takeaway:**  
Normalize heterogeneous API responses into a consistent common schema *before* returning them to the agent to reduce cognitive load and improve reasoning.

---

### Q13: Machine Identifiers vs. Ambiguous Human Inputs
> *Scenario: Tool Input Brittleness*  


**Question:**  
An agent updates sports scores using an `update_game_score(date, team_name)` tool. The tool frequently fails due to ambiguous team nicknames, rematches on the same day, and date format variations. What is the most reliable tool design to fix this?

**Options:**  
- A) Require strict ISO-8601 date formats and official full team names in the tool schema.  
- B) Improve the tool description to provide examples of correct formatting.  
- C) Add regex validation to the tool parameters to catch formatting errors early.  
- D) Introduce a `search_games` tool that returns a `game_id`, and update the scoring tool to accept only the `game_id`.  

**✅ Correct Answer: D — Use a lookup step returning `game_id`**

**Why this is correct:**  
Natural language attributes (like dates and team names) are inherently ambiguous and brittle when used as database keys. The architectural fix is to separate discovery from action. Use a lookup tool (`search_games`) that returns a machine-usable identifier (`game_id`), and force the mutative tool (`update_score`) to rely exclusively on that strict identifier.

**Why the others are weaker:**  
- **A)** Does not solve rematches, and LLMs still struggle with strict manual text formulation.  
- **B)** Documentation doesn't eliminate input ambiguity.  
- **C)** Validation catches errors but doesn't help the agent resolve the correct underlying game.  

**💡 Exam Takeaway:**  
Replace tools relying on ambiguous human-entered text fields with a two-step lookup/action pattern utilizing strict machine-usable identifiers (`IDs`).

---

### Q14: Enforcing Business Rules and Thresholds
> *Scenario: System Boundaries & Security*  


**Question:**  
An agent processes employee reimbursements. Company policy dictates that payouts over $500 must go to a pending approval queue and cannot be disbursed automatically. You need to ensure this threshold absolutely cannot be bypassed by the agent. Where should this rule be enforced?

**Options:**  
- A) Add a `requires_approval` boolean parameter to the tool schema that the agent must set.  
- B) Enforce the $500 threshold inside the `process_reimbursement` tool logic itself.  
- C) Write a strict system prompt instructing the agent to never disburse amounts over $500.  
- D) Provide two separate tools (`disburse` and `request_approval`) and trust the agent to select correctly.  

**✅ Correct Answer: B — Enforce internally inside the tool logic**

**Why this is correct:**  
If a rule *cannot* be bypassed, it must be enforced deterministically in the backend, not probabalistically in the LLM. Implementing the threshold inside the tool code guarantees that even if the agent hallucinates or is subjected to prompt injection, any payload over $500 will gracefully degrade to a "pending approval" state without breaking policy.

**Why the others are weaker:**  
- **A)** Leaves the security gate in the hands of the LLM; the agent could hallucinate `requires_approval=false`.  
- **C)** Prompt constraints are probabilistic and easily bypassed.  
- **D)** Agent tool selection can fail; if it chooses `disburse` for an $800 charge, the policy breaks.  

**💡 Exam Takeaway:**  
Strict business rules and approval thresholds must be enforced deterministically inside the backend tool logic, never via prompt instructions or agent tool selection.

---

### Q15: The Importance of Parameter Descriptions
> *Scenario: Tool Schema Documentation*  


**Question:**  
During execution, an agent repeatedly struggles to format inputs correctly for `user_id` and `fields_to_update` when calling an update tool. What is the most effective way to help the model understand exactly what values and formats to provide?

**Options:**  
- A) Write clear, detailed parameter descriptions in the tool schema.  
- B) Make the JSON schema extremely strict with complex regex constraints.  
- C) Rename the tool to include formatting hints in the tool name itself.  
- D) Add error-handling logic that explains the formatting rules only when the tool fails.  

**✅ Correct Answer: A — Clear parameter descriptions in the schema**

**Why this is correct:**  
Tool descriptions and parameter descriptions are the *primary mechanism* models use to understand expected inputs, formats, examples, and boundaries. Writing explicit instructions on a property (e.g., `"description": "UUID of the user to update (required)"`) directly guides the model's token generation prior to tool execution.

**Why the others are weaker:**  
- **B)** Strict validation rejects bad input but doesn't tell the model *how* to generate good input in the first place.  
- **C)** A naming workaround is vastly inferior to proper, expansive descriptions.  
- **D)** Reactive error messages waste execution turns compared to proactive schema descriptions.  

**💡 Exam Takeaway:**  
Clear, detailed parameter descriptions in the JSON schema are the primary and most important mechanism for guiding a model on input formatting.

---

### Q16: Handling Tool Errors: Transient vs. Syntax
> *Scenario: Error Recovery in Workflows*  


**Question:**  
A tool experiences two types of errors: transient network timeouts, and permanent user syntax errors. How should the tool handle these errors to optimize the agent workflow?

**Options:**  
- A) Pass all errors to the agent and prompt it to retry timeouts but stop on syntax errors.  
- B) Automatically retry transient network timeouts inside the tool, but immediately return syntax errors with clear validation details to the agent.  
- C) Uniformly auto-retry all errors 3 times before returning a failure message to the agent.  
- D) Return all errors immediately to the agent as generic 'Tool Execution Failed' messages.  

**✅ Correct Answer: B — Internal retries for transient, immediate return for syntax**

**Why this is correct:**  
Agents shouldn't waste context tokens and LLM execution time resolving predictable, system-level transient errors (like timeouts). Those should be handled via standard engineering retry loops inside the tool. However, syntax/validation errors require the agent's reasoning capability to fix the input. Returning those immediately with explicit validation details allows the agent to self-correct efficiently.

**Why the others are weaker:**  
- **A)** Pushing network retry loops to the LLM wastes expensive execution turns.  
- **C)** Retrying syntax errors is a waste of compute; a malformed UUID will fail 3 times identically.  
- **D)** Generic error messages give the LLM zero context to self-correct validation failures.  

**💡 Exam Takeaway:**  
Abstract transient error retries (timeouts) into backend tool logic, but return validation/syntax errors immediately to the agent with explicit details so it can self-correct.

---

### Q17: Differentiating Similar Tools via Descriptions
> *Scenario: Tool Boundaries & Selection*  


**Question:**  
An agent has access to an `archive_file` tool and a `delete_file` tool. It frequently calls `delete_file` when it should have archived a backup file. What is the most direct way to fix this tool selection error?

**Options:**  
- A) Expand the tool descriptions to clearly define the purpose, boundaries, and specific scenarios where archiving is preferred over deleting.  
- B) Add a confirmation prompt directly inside the `delete_file` tool logic.  
- C) Remove the `delete_file` tool entirely and handle deletions via a separate batch process.  
- D) Provide few-shot examples in the system prompt showing correct usage.  

**✅ Correct Answer: A — Clarify boundaries in tool descriptions**

**Why this is correct:**  
Tool descriptions are the highest-leverage mechanism for solving tool selection errors. When two tools have overlapping capabilities, their descriptions must explicitly define negative constraints and boundaries (e.g., `delete_file: "Permanently deletes a file. Do NOT use this for backup or compliance files—use archive_file instead."`). 

**Why the others are weaker:**  
- **B)** A confirmation prompt protects against damage but doesn't fix the fact that the agent is making the wrong choice upstream.  
- **C)** A massive architectural change that might not be possible given system requirements.  
- **D)** Few-shot prompting is helpful, but updating the native tool descriptions is the more robust, structural fix.  

**💡 Exam Takeaway:**  
When models confuse similar tools, rewrite the tool descriptions to explicitly define negative boundaries and specific scenarios for use.

---

### Q18: Resuming Sessions with Modified Files
> *Scenario: Iterative Workflows & State Updates*  


**Question:**  
You are using an agent to analyze a 12-file codebase. After the agent completes its initial review, a developer modifies 3 of the files. You want the agent to update its findings efficiently. What is the best approach?

**Options:**  
- A) Resume the session, explicitly inform the agent which 3 files changed, and instruct it to re-analyze only those files in the context of its prior findings.  
- B) Start a completely fresh session and have the agent re-analyze all 12 files from scratch.  
- C) Resume the session but don't explicitly mention the changes, trusting the agent to notice the file diffs organically.  
- D) Create a new session containing only the 3 modified files, discarding the context of the other 9 files.  

**✅ Correct Answer: A — Resume and inform agent of specific diffs**

**Why this is correct:**  
If the majority of a session's context is still valid, resuming is efficient. However, you must structurally inform the agent about the delta. By explicitly stating which files changed and directing it to re-analyze only those targets, the agent utilizes its existing contextual understanding of the codebase without wasting time re-verifying unmodified files.

**Why the others are weaker:**  
- **B)** Unnecessarily expensive and slow, throwing away perfectly valid context.  
- **C)** The agent may hallucinate that its prior tool findings are still true without looking, leading to stale reasoning.  
- **D)** Discarding the other 9 files removes crucial cross-file context needed for code analysis.  

**💡 Exam Takeaway:**  
When resuming sessions after minor code/data changes, explicitly inform the resumed session about the specific changes for targeted re-analysis rather than forcing full re-exploration.

---

### Q19: Exposing MCP Content Catalogs
> *Scenario: Discovery via Model Context Protocol*  


**Question:**  
An agent interacting with multiple Model Context Protocol (MCP) servers wastes significant context and time performing sequential lookup calls just to discover what data (issue tickets, docs, schemas) is available. How can you improve data discovery?

**Options:**  
- A) Consolidate all the MCP servers into a single, massive endpoint.  
- B) Add a new `discover_data` tool to every MCP server.  
- C) Implement keyword-based routing in the coordinator to send queries to the right server automatically.  
- D) Expose each MCP server's content catalog as an MCP Resource, allowing the agent to read what data exists before making targeted tool calls.  

**✅ Correct Answer: D — Expose catalogs as MCP Resources**

**Why this is correct:**  
MCP distinguishes between *Tools* (for taking actions or specific queries) and *Resources* (for exposing available context/data). Exposing hierarchies, issue summaries, or database schemas directly as Resources allows the agent to holistically "see" what data exists in a lightweight manner before deciding exactly which precise tool calls to make, eliminating exploratory spam.

**Why the others are weaker:**  
- **A)** Destroys microservice boundaries and doesn't solve the exploratory querying issue.  
- **B)** Re-invents the wheel; Resources are the native MCP feature for this exact problem.  
- **C)** Keyword routing is brittle and fails when questions span multiple systems.  

**💡 Exam Takeaway:**  
Use MCP Resources to expose content catalogs (docs, schemas, summaries) so agents can discover available data before wasting turns on exploratory tool calls.

---

### Q20: Prompt Chaining for Predictable Multi-Aspect Workflows
> *Scenario: Workflow Decomposition*  


**Question:**  
An agent is responsible for reviewing pull requests. Every PR must be reviewed for three specific aspects: code style, security vulnerabilities, and documentation accuracy. Which architectural pattern is best suited for this workflow?

**Options:**  
- A) Dynamic subagent decomposition, letting a coordinator agent decide which aspects to check case-by-case.  
- B) A routing agent that categorizes the PR and sends it to either a style, security, or docs specialist.  
- C) A single, massive prompt instructing one agent to analyze all three aspects simultaneously.  
- D) Prompt chaining: reviewing style, security, and documentation in separate sequential passes, then merging the outputs into a final synthesis.  

**✅ Correct Answer: D — Prompt chaining sequential passes**

**Why this is correct:**  
When a workflow is highly predictable, mandatory, and follows the exact same decomposition every time, *prompt chaining* is the superior pattern. Because style, security, and docs are entirely different cognitive lenses, chaining them sequentially in isolated passes prevents attention dilution (the model forgetting to check security while obsessing over style). 

**Why the others are weaker:**  
- **A)** Dynamic decomposition is for unpredictable workflows where the required tasks change per user request.  
- **B)** Routing implies only *one* of the specialists gets the job. The prompt says *every* PR gets all three.  
- **C)** Massive all-in-one prompts lead to severe attention dilution and missed instructions.  

**💡 Exam Takeaway:**  
Use prompt chaining for predictable, static multi-aspect workflows; isolate distinct cognitive tasks into separate sequential passes before merging them into a final output.

---

### Q21: Optimizing Follow-Up Queries in Multi-Agent Orchestration
> *Scenario: Agentic Architecture & Context Management*

**Question:**  
In a production environment, follow-up summarization queries to a multi-agent system take over 40 seconds. Investigation shows the coordinator agent spawns a synthesis subagent for each follow-up request, passing 80000 tokens of accumulated findings. The coordinator already holds these findings in its context from orchestrating the initial research. What is the most effective way to improve response time for these follow-up summaries?

**Options:**  
- A) Compress findings before passing them to the synthesis subagent.  
- B) Increase the synthesis subagent's context window.  
- C) Handle the summarization directly using the coordinator's existing context.  
- D) Cache synthesis subagent responses.  
- E) Use `fork_session` to speed up subagent spawning.  

**✅ Correct Answer: C — Handle directly at the coordinator level**

**Why this is correct:**  
Since the coordinator already has the research findings in its context from the orchestration phase, follow-up summarization queries should be handled by the coordinator itself. Subagents start fresh and do not inherit the coordinator's conversation history. Spawning a subagent and passing 80k tokens is an anti-pattern when the coordinator can do the work with its existing context.

**Why the others are weaker:**  
- **A)** Still spawns the subagent unnecessarily; reduces tokens but doesn't fix the architectural flaw.  
- **B)** Does not reduce latency; the bottleneck is the spawning and token transfer process, not the context limits.  
- **D)** Caching is a band-aid and does not fix the incorrect delegation pattern.  
- **E)** `fork_session` is intended for divergent exploration branches, not latency optimization.  

**💡 Exam Takeaway:**  
If the coordinator already has the information in its context, handle it at the coordinator level—never spawn a subagent just to re-process data the coordinator already owns.

---

### Q22: Passing Findings to a Synthesis Subagent
> *Scenario: Agentic Architecture & Subagent Isolation*

**Question:**  
After web search and document analysis subagents finish their tasks, the coordinator needs to spawn a synthesis subagent to combine the findings. What is the correct approach for providing the synthesis subagent with the information it needs?

**Options:**  
- A) Let the synthesis subagent call the search and analysis tools itself.  
- B) Let the synthesis subagent read directly from the coordinator's session history.  
- C) Pass a prose summary of the findings to the synthesis subagent.  
- D) Pass complete findings embedded directly in the synthesis subagent's prompt using a structured format separating content from metadata.  

**✅ Correct Answer: D — Pass complete findings in a structured format**

**Why this is correct:**  
Subagents start with zero knowledge and do *not* automatically inherit the coordinator's conversation history. The coordinator must explicitly inject all prior agent outputs directly into the synthesis subagent's prompt. Using a structured format (e.g., `{claim, evidence, source_url, date}`) ensures content, source metadata, and temporality are preserved so the synthesis agent can properly attribute findings.

**Why the others are weaker:**  
- **A)** Violates tool scoping. Synthesis agents should synthesize, not perform searches.  
- **B)** Architectural impossibility; subagents cannot inherit coordinator context.  
- **C)** A prose summary loses source attribution, meaning the synthesis agent cannot cite sources it doesn't have.  

**💡 Exam Takeaway:**  
Explicitly inject all prior findings directly into the subagent's prompt using a structured `{claim, evidence, source, date}` format; never rely on context inheritance or pass prose-only summaries.

---

### Q23: Procedural vs. Goal-Oriented Subagent Prompts
> *Scenario: Agent Adaptability & Prompt Engineering*

**Question:**  
A coordinator provides exact search queries, source priorities, and date filters step-by-step to a web search subagent. However, the subagent often reports "insufficient results" instead of trying alternatives, drops in quality on emerging topics, and rarely surfaces unconventional sources. What is the most effective way to improve the subagent's adaptability?

**Options:**  
- A) Add a fallback instruction to report failure if fewer than 5 results are found.  
- B) Replace procedural instructions with goal-oriented prompts detailing research goals and quality criteria.  
- C) Expand the exact query lists to cover emerging topics.  
- D) Provide generic, single-word queries to broaden the search base.  

**✅ Correct Answer: B — Switch to goal + quality criteria prompts**

**Why this is correct:**  
Over-specifying coordinator prompts with procedural, step-by-step instructions turns the subagent into a rigid executor. If exact steps fail, it hits a dead end. Providing a research intent, minimum quality thresholds (e.g., "minimum 5 distinct claims"), and criteria for credible sources grants the subagent the authority and context to dynamically adapt its approach and form its own queries.

**Why the others are weaker:**  
- **A)** Reinforces the rigidity that is causing the agent to quit early.  
- **C)** Still procedural. You cannot hard-code queries for unknown emerging patterns.  
- **D)** Degrades the specificity of the research entirely.  

**💡 Exam Takeaway:**  
Replace step-by-step procedural instructions with research goals and quality criteria to give subagents the authority to adapt their approaches when pre-specified paths fail.

---

### Q24: Parallel Processing in Subagent Architecture
> *Scenario: Latency Optimization & Tool Execution*

**Question:**  
A document analysis subagent processes citations in complex legal cases sequentially. A landmark case citing 12 precedents currently takes over 3 minutes to process. What is the most effective way to reduce this latency?

**Options:**  
- A) Increase the subagent's context window.  
- B) Have the coordinator emit multiple Task tool calls simultaneously in a single response.  
- C) Use `fork_session` to speed up processing.  
- D) Use the Message Batches API.  

**✅ Correct Answer: B — Parallel subagent spawning via multiple tool calls**

**Why this is correct:**  
Sequential processing of independent, parallelizable work is an anti-pattern. Instead of looping sequentially, the coordinator should emit all 12 Task tool calls in a single response turn. This spawns 12 parallel analysis subagents simultaneously, dropping the total latency from the *sum* of all tasks (3+ minutes) to the duration of the *longest single task* (~15-20 seconds).

**Why the others are weaker:**  
- **A)** Does not address the sequential loop architecture.  
- **C)** `fork_session` is for divergent exploration, not pipeline parallelism.  
- **D)** Batch API has up to a 24-hour SLA, making latency drastically worse.  

**💡 Exam Takeaway:**  
Emit multiple Task tool calls in a single coordinator response turn to achieve parallel execution and reduce pipeline latency.

---

### Q25: Handling Conflicting Findings and Uncertainty
> *Scenario: Context Management & Synthesis Precision*

**Question:**  
Final reports consistently mishandle uncertainty. For example, a web search agent returns a $50B estimate (unspecified methodology), while a document analysis agent returns a $35B estimate (±$7B, 95% CI). The coordinator either picks one arbitrarily or produces a vague, hedged statement. What approach best avoids this?

**Options:**  
- A) Instruct the coordinator to always prefer peer-reviewed sources.  
- B) Ask the synthesis subagent to pick the most recent figure.  
- C) Have the coordinator average the two conflicting values.  
- D) Require subagents to return structured data with methodology/confidence metadata and a `conflict_detected` flag to preserve both values.  

**✅ Correct Answer: D — Structured claim mappings with metadata and flags**

**Why this is correct:**  
Unstructured prose findings lose provenance. If subagents return plain text, the coordinator has no structured basis to distinguish high-quality methodologies from guesses. Subagents must return structured objects `{claim, source_type, methodology, confidence, date}`. When claims conflict, a programmatic `conflict_detected` flag forces the synthesis agent to handle it explicitly by presenting both values with their source context in the final report, rather than silently collapsing them.

**Why the others are weaker:**  
- **A)** A prompt-based rule is probabilistic and fails if only non-peer-reviewed data is available.  
- **B)** Recency does not equal accuracy.  
- **C)** Averaging fabricates a number unsupported by either source, destroying attribution.  

**💡 Exam Takeaway:**  
Require subagents to return structured metadata objects and use a `conflict_detected` boolean so synthesis agents preserve both values with attribution rather than silently collapsing them.

---

### Q26: Context Trimming and the "Lost in the Middle" Effect
> *Scenario: Context Budgets & Downstream Handoffs*

**Question:**  
A web search agent gathers 120k tokens of raw content, a document analysis agent extracts 15k tokens of insights, and a synthesis agent produces a 3k-token narrative draft. The coordinator must now pass context to a report generation agent for final output with proper citations. Which context-passing strategy provides the best balance of completeness and efficiency?

**Options:**  
- A) Pass the 120k tokens of raw content and all intermediate outputs.  
- B) Pass only the 3k-token synthesis narrative.  
- C) Pass the synthesis narrative and the full document analysis reasoning chain.  
- D) Pass the synthesis narrative along with a lean, structured citation index and conflict flags.  

**✅ Correct Answer: D — Pass synthesis narrative + structured citation index**

**Why this is correct:**  
Passing 138k+ tokens of raw content to the report agent wastes budget, balloons latency, and triggers the "lost in the middle" effect (where the LLM ignores context in the middle of a massive prompt). Passing too little causes hallucinations. The optimal balance is passing the dense narrative backbone (3k) plus a compact, structured citation index (5–8k tokens) representing only the essential metadata `{citation_id, claim, source, key_quote}`. 

**Why the others are weaker:**  
- **A)** Causes "lost in the middle" and massive token bloat.  
- **B)** The report agent will have no citation data and will fabricate sources.  
- **C)** Reasoning chains are internal to the previous agent and just add noise downstream.  

**💡 Exam Takeaway:**  
Pass the synthesis narrative plus a structured citation index; strip raw content and reasoning chains entirely to avoid "lost in the middle" degradation.

---

### Q27: Preserving Provenance Through Multi-Agent Summarization
> *Scenario: Metadata Preservation & Attribution*

**Question:**  
A synthesis agent receives findings from upstream agents and passes a consolidated prose summary to a report generation agent. During testing, the report generator makes factual claims but cannot accurately attribute them because source metadata was lost during the summarization step. What is the most effective approach to ensure proper source attribution?

**Options:**  
- A) Ask the synthesis agent to re-include the full source text in its summary.  
- B) Assign a `citation_id` at the earliest agent, and require the synthesis agent to produce an inline-tagged narrative alongside a preserved structured citation index.  
- C) Instruct the report agent to infer the original sources from the claim content.  
- D) Instruct the synthesis agent to "preserve sources" in its prose output.  

**✅ Correct Answer: B — Use Citation IDs as persistent anchors**

**Why this is correct:**  
Prose summaries inherently destroy metadata (URLs, page numbers, dates) as they collapse text into readable sentences. The structural fix is to separate content from metadata. Assign a `citation_id` at the source discovery level. The synthesis agent then writes a narrative using inline brackets (e.g., `[src_001]`), while the full metadata lives safely in a separate structured citation index array passed alongside the narrative.

**Why the others are weaker:**  
- **A)** Re-inflates the context window, causing "lost in the middle" failures.  
- **C)** The report agent doesn't have the original sources; it will hallucinate plausible citations.  
- **D)** Prompt-based instructions are probabilistic; under token pressure, models will still drop metadata in prose.  

**💡 Exam Takeaway:**  
Assign `citation_id` tags at the earliest stage, require the synthesis agent to output an inline-tagged narrative, and pass a separate structured citation index to the final agent.

---

### Q28: Resuming a Crashed Pipeline and State Management
> *Scenario: Stateful Orchestration & Failure Recovery*

**Question:**  
A multi-agent research pipeline crashes after processing 12 out of 18 documents. Each agent has partially completed work. You need to resume the pipeline without losing the fidelity of prior findings or repeating completed work. What is the best state management approach?

**Options:**  
- A) Run the `--resume` command directly on the crashed session.  
- B) Use `fork_session` from the crash point to branch the execution.  
- C) Write completed findings to a structured checkpoint file, start a fresh session, and inject the checkpoint as structured context.  
- D) Resume the session without noting partial document states.  

**✅ Correct Answer: C — Structured summary injection via a fresh session**

**Why this is correct:**  
When a pipeline crashes mid-execution, tool results in that session become stale and unreliable. Blindly resuming risks the agent re-processing data or corrupting context. The correct pattern is to extract completed work into a durable structured JSON checkpoint file, start a *new* session, and inject that checkpoint at the top of the prompt. You must explicitly tell the agent which documents are complete, which are partial (by section), and which are pending.

**Why the others are weaker:**  
- **A)** Stale tool results from the crashed session are unreliable.  
- **B)** `fork_session` is for parallel divergent exploration, not pipeline failure recovery.  
- **D)** The agent will likely re-do completed work or skip the partially processed document entirely.  

**💡 Exam Takeaway:**  
Extract completed work to a structured checkpoint file, start a fresh session, and explicitly inject the checkpoint—never blindly `--resume` a crashed session with stale tool results.

---

### Q29: Temporal Differences vs. Data Contradictions
> *Scenario: Data Synthesis & Schema Design*

**Question:**  
During research on "renewable energy adoption," a web search agent returns statistics from 2024, while a document analysis agent returns statistics from 2021. The synthesis agent incorrectly flags these as contradictory rather than recognizing a growth trend over time. What single change best enables the synthesis agent to correctly interpret this difference?

**✅ Correct Answer: Require `publication_date` in structured outputs and update synthesis instructions.**

**Why this is correct:**  
Without a structural representation of time, differing values look like conflicts to the model. By enforcing `publication_date` in the subagent output schema and instructing the synthesis agent to view differing values across distinct dates as progression, the system correctly identifies temporal growth. `conflict_detected: true` should only trigger for differing values within the *same* time period.

**💡 Exam Takeaway:**  
Always include temporal metadata (`publication_date`) in extraction schemas to prevent synthesis agents from misinterpreting chronological progression as data contradiction.

---

### Q30: Human Review Thresholds & Stratified Accuracy
> *Scenario: Output Validation & Automation Gating*

**Question:**  
An extraction system has operated with 100% human review for 3 months. Analysis shows that extractions with a model confidence of >=90% have a 97% accuracy overall. To reduce reviewer workload, you plan to automate high-confidence extractions. Before deploying, what validation step is most critical?

**Options:**  
- A) Segment the accuracy metrics by document type and field.  
- B) Deploy the automation globally at the >=90% confidence threshold.  
- C) Run a random sample review across all documents before deploying.  
- D) Increase the confidence threshold to >=95% just to be safe.  

**✅ Correct Answer: A — Segment accuracy by document type AND field**

**Why this is correct:**  
An aggregate metric of "97% overall" is dangerous because it can mask catastrophic failure rates in minority subsets (e.g., standard invoices might be 99.2% accurate, while handwritten forms are 61.0%). You must run a stratified analysis to break down accuracy by document type and field before automating, ensuring you only remove humans from safe segments.

**Why the others are weaker:**  
- **B)** Automating based on aggregate metrics will silently ship high error rates for specific edge cases.  
- **C)** Random sampling under-represents rare or difficult document types.  
- **D)** Self-reported model confidence is an unreliable proxy for actual accuracy.  

**💡 Exam Takeaway:**  
Always segment accuracy by document type and field before automating; overall accuracy metrics mask per-category failure rates.

---

### Q31: Batch Processing Chunking and Cost Efficiency
> *Scenario: API Usage & Error Handling*

**Question:**  
After a daily batch of 10,000 documents completes processing, 300 documents (3%) fail with `context_length_exceeded` errors. The result file identifies each failure by its `custom_id`. What is the most cost-effective approach to process these failures?

**Options:**  
- A) Resubmit all 10,000 documents with a smaller chunk size.  
- B) Switch the 300 failed documents to the synchronous API.  
- C) Extract only the 300 failed documents using their `custom_id`, chunk them, and resubmit them as a new batch.  
- D) Increase the context window limit globally.  

**✅ Correct Answer: C — Resubmit only failed documents with chunking**

**Why this is correct:**  
Resubmitting successful documents wastes money (97% of the cost). You must use the `custom_id` mapping provided by the Batch API result file to isolate the 300 failures. Because the error is `context_length_exceeded`, the documents are too large for a single request. You must chunk those specific documents into smaller segments and submit them in a new batch request.

**Why the others are weaker:**  
- **A)** Reprocessing 9,700 already-successful documents is a massive waste of API costs.  
- **B)** Synchronous API costs 2x more than Batch and offers no latency advantage for documents that require chunking anyway.  
- **D)** Context limits are fixed model constraints, not toggleable configuration settings.  

**💡 Exam Takeaway:**  
Use `custom_id` to extract only failed documents, modify them (e.g., chunking for context limits), and resubmit only the failures to maintain Batch API cost efficiencies.

---

### Q32: Tool Definitions Consuming the Context Window
> *Scenario: Context Degradation & Schema Bloat*

**Question:**  
An extraction system uses a 12-field JSON schema and detailed tool descriptions totaling ~2,500 tokens. For documents under 150k tokens, accuracy is 98%. However, for documents between 175k-185k tokens, accuracy drops to 71%, with information from the final third of the document consistently missing. The model's context window is 200k tokens. What is the most likely cause of this degradation?

**Options:**  
- A) The tool definition consumes input tokens, and combined with system prompts, pushes the total input close to the context limit, degrading attention for content at the end.  
- B) Schemas exceeding 8-10 fields inherently increase decision complexity.  
- C) Very long documents exceed the model's effective attention span causing middle-document degradation.  
- D) The model distributes attention proportionally, causing the end of the document to receive insufficient focus.  

**✅ Correct Answer: A — Tool definition + system prompt consumes usable context**

**Why this is correct:**  
Context limits are absolute. If you have a 200k limit, but 2,500 tokens are taken by the tool schema and 1,500 by the system prompt, your *effective* document space is ~196k. As document tokens approach 180k+, the total payload nears the edge of the context boundary. Models suffer degraded attention (not just hard cut-offs) near the absolute limit, which manifests specifically as the "final third consistently missing."

**Why the others are weaker:**  
- **B)** Schema complexity causes semantic errors (wrong values), not positional omission. Accuracy is 98% on short docs with the same schema.  
- **C)** "Lost in the middle" affects the middle, whereas the symptom here specifically calls out the *final third* missing.  
- **D)** Proportional attention is a fabricated concept.  

**💡 Exam Takeaway:**  
Large tool schemas and system prompts consume your context budget; long documents near the absolute limit will suffer degraded attention at the end of the prompt. Trim schemas or chunk documents.

---

### Q33: Enforcing Tool Execution Order with `tool_choice`
> *Scenario: Tool Orchestration & Dependencies*

**Question:**  
A pipeline uses an `extract_metadata` tool that returns a DOI. It also has `lookup_citations` and `verify_doi` enrichment tools that *require* a DOI to function. When users ask to "extract the metadata and tell me how cited it is", the model sometimes calls the enrichment tools first, which fail because they lack the DOI. What is the most effective way to ensure structured metadata extraction happens first?

**Options:**  
- A) Add a prompt instruction to "always call extract_metadata first."  
- B) Set `tool_choice` to strictly force the `extract_metadata` tool on the first turn, then use `tool_choice: "auto"` for subsequent turns.  
- C) Combine all three tools into a single massive tool.  
- D) Use `tool_choice: "auto"` but restrict the descriptions of the enrichment tools.  

**✅ Correct Answer: B — Force prerequisite tool selection via `tool_choice`**

**Why this is correct:**  
When tools have strict dependencies (a DOI must exist before citations can be looked up), you must enforce a deterministic prerequisite gate. Setting `tool_choice: {"type": "tool", "name": "extract_metadata"}` explicitly forces the model to execute that tool on Turn 1. Once the DOI is returned to the context, you switch to `"auto"` on Turn 2 so the model can freely use the enrichment tools.

**Why the others are weaker:**  
- **A)** Prompt instructions have a non-zero failure rate. Probabilistic prompting is inferior to deterministic API constraints.  
- **C)** Decreases composability and massively bloats tool complexity.  
- **D)** Under `"auto"`, the model reads the full user request and may still attempt the second step in parallel.  

**💡 Exam Takeaway:**  
When deterministic execution order is required for tool dependencies, use forced `tool_choice` on the first turn, then release to `"auto"` on subsequent turns.

---

## Key Concepts Cheat Sheet

* **Tool vs. Prompt Enforcement:** Always prefer enforcing rules, execution orders (`tool_choice`), and business logic thresholds inside the schema or backend tool itself, rather than relying on prompt instructions.
* **Context Budgeting:** Prevent "lost in the middle" effects by aggressively trimming raw content and intermediate reasoning chains before downstream handoffs.
* **Structured Data Over Prose:** Prose collapses metadata. Always use structured JSON outputs to preserve attribution (via explicit IDs, dates, URLs) through multi-agent pipelines.
* **Batch API Optimization:** To save costs, chunk and resubmit *only failed documents* (using `custom_id`), submit batches based on strict SLA math + buffer, and refine your prompts on small samples *before* wide-scale batch execution.
* **Tool Interfaces:** Reduce agent guesswork. Use enums for bounded categories, require backend systems to handle ID-to-string lookups, and always provide explicit pagination controls instead of dumping massive datasets.
* **Session Recovery:** Never `--resume` blindly after a crash. Extract a structured checkpoint of completed work, inject it into a *new* session, and tell the agent exactly where to start.
* **Schema Descriptions:** Tool and parameter descriptions in the schema are the *primary mechanism* an LLM uses to know when to use a tool and how to format its inputs. 

## Tips from a Real Exam Taker

> *"I chose the most intuitive answers based on my current experience building AI Agents and AI Harnessing systems also experimenting running claude code with Local llms etc.*
> 
> *It was a tricky paper. Where for most of the MCQs, several options could technically solve the problem in the question, but the real challenge was picking the solution that follows the best practices. And this is where software engineering experience comes handy.*
> 
> *The exam tests your AI proficiency, AI ethics, system design & critical thinking and user experience mindset."*

Based on these reflections and the patterns in the exam, keep these tactical strategies in mind when sitting for the test:

1. **Always look for the structural fix.** The wrong answers heavily feature "improve the prompt" or "tell the agent not to do it." The correct answers almost always feature structural API changes, `tool_choice`, `enum` parameters, and structured JSON flags.
2. **Beware of over-spawning.** If a coordinator already holds the context, spawning a subagent is wrong. Subagents are for fresh exploration, not re-processing existing data.
3. **Machine Identifiers rule.** If an agent needs to perform an action on a specific item (a document, a game, a user), the correct architecture always uses an explicit ID (`game_id`, `citation_id`), never an ambiguous text string.
4. **Assume self-reported metadata is untrusted.** Especially for MCP servers, `readOnly` flags are just suggestions. Base security bypasses on system-level trust, not tool annotations.

Need these questions in a flash card or quiz view? Try [https://ericspencer.us/Claude-architect-quiz/](https://ericspencer.us/Claude-architect-quiz/)
