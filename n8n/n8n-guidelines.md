# N8N Workflow Prompt Framework

When requesting n8n workflow builds, structure your prompt with these key elements:

## 1. **Objective Statement**

Start with a clear, single-sentence goal:

- "Build a workflow that automatically scrapes job postings and sends personalized applications"
- "Create a lead generation system that enriches contact data and adds it to a CRM"

## 2. **Input Sources**

Specify exactly where data originates:

- Webhook triggers (what data structure?)
- Scheduled intervals (how often?)
- Manual triggers
- Database queries
- API endpoints
- File uploads

## 3. **Data Processing Steps**

List transformations in sequence:

- What needs to be extracted/parsed?
- What enrichment is needed (AI, APIs, lookups)?
- What validation/filtering rules apply?
- What decisions/branching logic is required?

## 4. **External Integrations**

Name specific services and what actions to take:

- APIs (with authentication method if known)
- Databases (type and connection details)
- Email services
- Cloud storage
- CRM/marketing tools

## 5. **Output Requirements**

Define success criteria:

- Where does data end up?
- What format is needed?
- What notifications should fire?
- What error handling is required?

## 6. **Constraints & Preferences**

Include any limitations:

- Rate limits to respect
- Cost considerations (API call minimization)
- Timing requirements
- Data privacy needs
- Existing infrastructure to work with

## Example Strong Prompt:

```
Build an n8n workflow that monitors my target companies for job postings:

INPUT:
- Manual trigger to start
- Google Sheet with company names and career page URLs

PROCESSING:
1. Loop through each company
2. Scrape their careers page for new postings
3. Use Claude API to analyze if posting matches my skills
   (Next.js, React, TypeScript)
4. Extract: job title, description, location, salary if listed
5. Score relevance 1-10 based on tech stack match

INTEGRATIONS:
- HTTP Request for scraping (with Cheerio parsing)
- Anthropic API for analysis
- PostgreSQL to store results and track already-seen postings
- Gmail to send daily digest

OUTPUT:
- Insert new relevant jobs (score ≥7) into PostgreSQL
- Daily email with top 5 matches
- Error notifications if scraping fails

CONSTRAINTS:
- Run daily at 8am
- Rate limit: max 1 request per second per domain
- Store raw HTML for debugging failed parses
```

## Pro Tips:

**Be specific about data structure** - Share example JSON/objects if possible

**Mention error scenarios** - "If scraping fails, log and continue to next company"

**State your n8n knowledge level** - "I'm new to n8n" vs "I'm comfortable with expressions"

**Include existing assets** - "I already have the Claude MCP server configured"

**Request explanations** - "Please explain the Code node logic for complex parsing"

This framework ensures you get workflows that actually work with minimal back-and-forth revisions.
