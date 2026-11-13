# Topic 8 - Single-Agent vs. Multi-Agent Systems

## Project
**Workflow:** `Topic_8_Multi_Agent_Research_Editor`  
**Platform:** n8n  
**Test date:** November 13, 2026  
**Execution ID:** 640194  
**Status:** Successful (11.538 seconds)

## Objective
Demonstrate a sequential multi-agent pipeline where a specialized Researcher Agent fetches raw data from a mock database and hands the result to an Editor Agent that writes a customer-facing newsletter.

## Architecture
`When chat message received -> Researcher_Agent -> Editor_Agent`

- **Chat Trigger:** accepts a user's lookup request.
- **Researcher_Agent:** uses Google Gemini and the `database_search_tool` HTTP Request tool; returns source facts without commentary.
- **database_search_tool:** performs a GET request against `https://jsonplaceholder.typicode.com/posts/{record_id}`. The tool uses an n8n `$fromAI('record_id', 'Numeric database record ID', 'number')` expression to supply the numeric record ID.
- **Editor_Agent:** uses a separate Google Gemini model; receives the original request and researcher output, and writes a professional newsletter. No tools are connected to this agent.

## Configuration
1. Connect the Chat Trigger to Researcher_Agent, then connect Researcher_Agent to Editor_Agent.
2. Researcher prompt: `{{ $json.chatInput }}`. Instruct it to call the database tool and return only retrieved facts.
3. HTTP Request Tool: GET, no authentication, URL `https://jsonplaceholder.typicode.com/posts/{{ $fromAI('record_id', 'Numeric database record ID', 'number') }}`.
4. Editor prompt: set Source to **Define below**, Expression mode, and include the original chat request plus `{{ $json.output }}` from the Researcher Agent. Match the Chat Trigger node name exactly in any cross-node expression.
5. Editor system instructions: use only retrieved facts, avoid invention, and produce a concise, polite newsletter.

## Test case and results
**Input:** `Look up database record 3 and write a newsletter update.`

**Observed:**
- Researcher_Agent invoked `database_search_tool` with `record_id = 3`.
- Tool requested `https://jsonplaceholder.typicode.com/posts/3` and returned a record with `userId: 1`, `id: 3`, `title`, and `body`.
- Researcher_Agent returned the raw record content to Editor_Agent.
- Editor_Agent generated a newsletter headed **Update Regarding Database Record #3**.
- Full workflow execution **succeeded in 11.538 seconds**, execution ID **640194**.

## Execution-log evidence
| Log entry | Model-call duration | Tokens |
|---|---:|---:|
| Researcher Gemini call shown | 2,805 ms | 204 |
| Editor Gemini call shown | 6,742 ms | 397 |

These are the individual model-call values visible in the supplied screenshots, not a comprehensive total of all internal model calls. The complete workflow duration is 11.538 seconds.

## Evidence index
1. Complete successful execution canvas
2. Researcher_Agent raw output
3. database_search_tool GET response for record 3
4. Editor_Agent newsletter output
5. Researcher Gemini execution log
6. Editor Gemini execution log

## Notes
- JSONPlaceholder is a public mock API, not a production database.
- Record 3 contains placeholder Latin text; the generated newsletter is a demonstration of agent orchestration, not a factual business update.
- Keep API credentials out of screenshots and exported submissions.
