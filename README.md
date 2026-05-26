# ai-lead-qualification-n8n
# AI Lead Qualification System — n8n + OpenRouter Workflow

![Status](https://img.shields.io/badge/Status-Complete-brightgreen)
![Stack](https://img.shields.io/badge/Stack-n8n%20%7C%20OpenRouter%20%7C%20Google%20Sheets%20%7C%20Gmail-blue)
![AI](https://img.shields.io/badge/AI-GPT--3.5--turbo%20via%20OpenRouter-purple)
![Niche](https://img.shields.io/badge/Niche-EdTech%20%7C%20Coaching%20Centers-orange)

An AI-powered lead qualification workflow that analyzes student inquiry text using GPT-3.5, assigns dynamic lead scores, routes high-priority leads for immediate follow-up, and updates CRM records automatically — built for NEET/JEE coaching institutes.

---

## Live Demo

🔗 https://aileadqual-8yrmwn9q.manus.space

---

## What This Does

When executed, this workflow:

1. Reads all student inquiry records from Google Sheets
2. Skips leads already marked as `"Qualified"` (rerun safety)
3. Sends each inquiry to OpenRouter AI for qualification analysis
4. Parses the structured JSON response (Lead_Score, Priority, AI_Reason)
5. Routes high-priority leads → instant Gmail alert to owner
6. Updates all CRM records with AI-generated scores and status

Every lead gets a unique AI assessment based on their actual inquiry text. No hardcoded scoring.

---

## Problem It Solves

Coaching center owners can't manually evaluate every inquiry for intent. High-priority students — those asking about specific batch dates, fees, or scholarships — need a response within minutes. Low-intent inquiries can wait.

Without qualification, owners spend equal time on all leads and lose their best prospects to competitors who respond faster.

This system makes the prioritization decision automatically using AI.

---

## Tech Stack

| Layer | Tool |
|---|---|
| Automation Platform | n8n |
| AI Model | GPT-3.5-turbo via OpenRouter API |
| CRM / Data Layer | Google Sheets |
| Communication | Gmail |
| Response Parsing | Custom JavaScript (Code node) |
| Routing Logic | IF node, conditional branching |

---

## Workflow Architecture

```
Manual Trigger
    ↓
Get all rows — Google Sheets (AI_Lead_Qualification)
    ↓
IF: Status != "Qualified" (skip already processed)
    ↓ TRUE
HTTP POST → OpenRouter API (GPT-3.5-turbo)
    ↓
Edit Fields (extract AI_Response, preserve CRM fields)
    ↓
Code Node — JavaScript (parse JSON, strip markdown fences)
    ↓
IF: Priority = "High"
    ↓ TRUE                         ↓ FALSE
Gmail alert → owner            Update CRM: "Reviewed - Low Priority"
    ↓
Update CRM: Status = "Qualified"
```

---

## Google Sheets Schema

| Column | Type | Description |
|---|---|---|
| Name | String | Student full name |
| Email | String | Student email (match key) |
| Inquiry | String | Raw inquiry text from student |
| Lead_Score | Number | AI-assigned score (1-10) |
| Priority | String | High / Medium / Low |
| Status | String | New / Qualified / Reviewed - Low Priority |

---

## AI Prompt Structure

```
System: You are an AI lead qualification system for a coaching institute.

User: Analyze this lead inquiry.

Name: [student name]
Inquiry: [student inquiry text]

Return ONLY valid JSON in this format:
{
  "Lead_Score": number (1-10),
  "Priority": "High" or "Medium" or "Low",
  "AI_Reason": "short explanation"
}
```

Strict JSON output instruction prevents freeform responses that break parsing.

---

## JavaScript Parsing Logic

```javascript
return items.map(item => {
  let response = item.json.AI_Response;

  // Strip markdown code fences (common LLM behavior)
  response = response.replace(/```json/g, "").replace(/```/g, "").trim();

  const parsed = JSON.parse(response);

  return {
    json: {
      ...item.json,
      Lead_Score: parsed.Lead_Score,
      Priority: parsed.Priority,
      AI_Reason: parsed.AI_Reason
    }
  };
});
```

---

## Setup Instructions

**Prerequisites:**
- n8n instance (local or cloud)
- OpenRouter account with API key
- Google Sheets with the schema above
- Gmail account
- Google Sheets OAuth2 and Gmail OAuth2 credentials in n8n

**Steps:**

1. Clone or download this repository
2. Import `Demo-3-AI-Lead-Qualification-System.json` into n8n
3. Connect Google Sheets OAuth2 credential
4. Connect Gmail OAuth2 credential
5. Add your OpenRouter API key to the HTTP Request node Authorization header
6. Update the Google Sheets document ID to your own sheet
7. Add test inquiry data (varied inquiries to see different AI scores)
8. Click Execute Workflow and verify AI scoring and Gmail alert

**⚠️ Important:** Never commit your OpenRouter API key to a public repository. Use n8n credential manager or environment variables for production deployments.

---

## Debugging Notes

**401 Unauthorized from OpenRouter**
Check for trailing spaces in the Authorization header name. `"Authorization "` and `"Authorization"` are treated as different headers in n8n.

**JSON.parse() failing**
AI sometimes wraps JSON in markdown code fences. The Code node strips these before parsing. If parsing still fails, log `item.json.AI_Response` to inspect raw output.

**Priority IF condition not routing correctly**
Verify AI is returning capitalized values ("High" not "high"). Prompt explicitly specifies capitalization — if changed, update IF node condition to match.

**CRM fields missing downstream**
After HTTP Request node, original CRM fields are replaced by API response object. Edit Fields node re-maps Name, Email, and Inquiry using node references (`$('Get row(s) in sheet').item.json.Name`).

---

## Upgrade Path

For production deployment, this workflow can be extended with:

- Webhook trigger for real-time processing as inquiries arrive
- Upgrade to GPT-4 or Claude for higher qualification accuracy
- WhatsApp alert integration for instant mobile notifications
- Multi-model routing (fast model for scoring, premium model for high-priority only)
- Airtable or Supabase migration for larger data volumes

---

## Related Projects

| Project | Description |
|---|---|
| Demo 1 | LeadFlow AI — Frontend SaaS product demo |
| Demo 2 | Lead Follow-Up Automation (n8n + Google Sheets + Gmail) |

---

## About

Built by Karthik — AI automation builder focused on vertical AI systems for Indian coaching institutes and EdTech businesses.

---

## Contact

📩 karthikgajula93@gmail.com
