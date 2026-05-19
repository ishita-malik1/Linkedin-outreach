# AI Agent for Personalized LinkedIn Outreach 
## Use case and problem statement 
Job searching is a numbers game, but it rarely feels that way when you are doing it. After spending time 
tailoring a resume, researching a company, and submitting an application, most candidates move on to 
the next role without ever following up directly with the people doing the hiring. This is not laziness. It is a 
resource problem. Finding the right person at a company, figuring out whether they are a recruiter or a 
hiring manager, reading enough about them to write something that does not sound copy-pasted, and 
then actually drafting the message is a 20-30 minute task per role. Multiply that across 10 or 15 active 
applications and it becomes the kind of work that gets deprioritized. 
The irony is that direct outreach is one of the highest-leverage things a job seeker can do. Recruiters and 
hiring managers consistently note that a personalized, well-timed message referencing something 
specific to the role meaningfully increases the chance of a response. The gap between knowing this and 
doing it at scale is where most candidates fall short. 
This project was built to close that gap. The goal was not to fully automate outreach, which raises 
concerns around LinkedIn's terms of service, but to automate the research and drafting work so that the 
human only needs to review and hit send. 
## What I built 
I built a Python-based AI agent that integrates with my existing job application tracker and produces 
ready-to-send LinkedIn outreach drafts for each active application. The tracker is a simple Excel sheet 
maintained throughout my job search, with columns for company name, job title, and application status. 
The agent reads the last five rows where status is not marked as rejected and treats each one as a task. 
For each role, the agent follows a multi-step research and drafting pipeline. It begins by searching the 
web for the specific job posting, extracting key responsibilities and requirements. It then searches for the 
hiring manager most likely connected to that role, prioritizing them over a generic recruiter since direct 
outreach to the decision-maker tends to perform better. If a hiring manager cannot be identified after 
multiple searches, the agent falls back to a recruiter. If neither is found, it flags the role and moves on 
without fabricating a contact. 
Once a contact is identified, the agent fetches their public LinkedIn page to confirm their name and 
current title. It then reads my resume, provided as a PDF, and identifies the one or two most relevant 
alignment points between my background and the specific role. These are concrete, not generic. For 
example, noting that a past project in real-time data infrastructure maps directly to a requirement in the 
job posting, rather than stating vaguely that I am a strong fit. 
Finally, the agent drafts two outreach messages. The first is a LinkedIn connection note capped at 300 
characters, warm and direct with a single specific hook. The second is a longer InMail of around 100 to 
150 words, written conversationally without buzzwords or filler phrases. Both are written as if they came 
from a real person who did their homework. 
The output for each role is a structured card showing the contact name, title, LinkedIn URL, confidence 
level in the contact's accuracy, the alignment point used, and both message drafts, along with any 
caveats such as whether the job posting could not be found. 
The stack is intentionally simple: Python, Google Gemini as the LLM, Serper.dev for web search, httpx 
and BeautifulSoup for fetching and parsing web pages, pypdf for resume extraction, and openpyxl for 
reading the Excel tracker. The project was built in Cursor using a natural language prompt describing the 
desired behavior, with Cursor generating the scaffolding and agent loop. 
## Why this system is agentic 
In a standard prompt-response setup, the user provides all context upfront and the model returns a 
single output. The model is reactive. An agentic system is different because the model is making 
decisions at runtime, deciding what information it needs, which tool to call, what to do with the result, 
and whether to proceed or try a different approach. This observe, reason, and act loop continues until the 
task is complete or the agent determines it cannot proceed. 
In this project the LLM acts as the orchestrator of the entire workflow. When given a company name and 
job title, it does not immediately draft a message. It reasons about what it needs to know first. It calls the 
search tool to find the job posting, evaluates the results, and decides whether to proceed or retry with a 
different query. It searches for the hiring manager, assesses whether the result is plausible, fetches the 
LinkedIn page to verify, reads the resume, reasons about relevance, and only then drafts the message. 
Each of these decisions is made by the model dynamically, not hardcoded into the program logic. 
The failure handling behavior illustrates this clearly. A static pipeline would crash or return blank output 
when a contact cannot be found. This agent reasons through the failure, tries a fallback search, and if 
that also fails, decides to flag the role and move on. That kind of adaptive decision-making under 
uncertainty is a core characteristic of agentic behavior. 
One deliberate design choice worth noting is that the agent does not send messages. It does not log into 
LinkedIn or take any irreversible action without human review. This reflects an important principle in 
agentic system design: autonomy should be scoped to steps where errors are recoverable. Research and 
drafting can be wrong without causing harm. The send step cannot, so it stays with the human. 
## End-to-end example 
Since this tool runs locally and is not deployed on the web, the following is a walkthrough of a real run to 
demonstrate that the workflow works end to end. 
### Evaluation 
I evaluated the system across three dimensions- 
The first was contact accuracy. For each role I manually verified whether the person surfaced was 
genuinely connected to that position, rating results as high, medium, or low confidence. Results were 
strongest for mid-sized and larger companies with active LinkedIn presence, and weakest for smaller 
companies or roles with generic titles where search results were ambiguous. 
The second dimension was message quality. I assessed each draft against a simple rubric: does it 
reference something specific to the role or person, does it avoid generic filler language, does it respect 
the 300 character limit, and does it read like something a real person would send. Most drafts passed. 
The most common failure was slightly formal language in the InMail when the job posting was vague and 
gave the agent little concrete detail to reference. 
The third dimension was failure handling. I tested edge cases including companies with no LinkedIn 
presence, overly generic job titles, and a deliberate typo in the company name. The agent handled the 
first two gracefully by flagging them and skipping the draft step.  
Overall the agent performed well for its intended use case, reliably saving 20 to 30 minutes of manual 
research per role and producing drafts that needed only minor editing before sending. 
