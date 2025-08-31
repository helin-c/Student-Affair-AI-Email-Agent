


# 🎓 Student Affairs AI Email Agent  

An **AI-powered intake & reply workflow** built with **n8n, OpenAI, Supabase, Slack, and Gmail**.  
This workflow automates the processing of student emails, classifies and drafts responses using RAG (retrieval-augmented generation), and enables a **human-in-the-loop approval** via Slack before sending polished replies back to students.  

---

## ✨ Features  

- 📥 **Gmail Integration**  
  - Monitors unread inbox messages (ignores spam & already processed).  
  - Auto-labels messages (`Pending`, `Processed`, `Approved`, `Rejected`, `Error`).  

- 🧠 **AI Classification & Extraction**  
  - Extracts sender, subject, and body.  
  - Classifies emails into categories (`admissions`, `financial_aid`, `academics`, etc.).  
  - Rates urgency on a 1–5 scale.  
  - Identifies missing information (passport, student ID, course code…).  

- 📚 **Knowledge Base RAG**  
  - Connects to a **Supabase pgvector store** for policies, FAQs, and procedures.  
  - Uses OpenAI embeddings + GPT for retrieval-augmented replies.  

- 💬 **Slack Approval Flow**  
  - Posts draft responses to a Slack channel with buttons: **Approve & Send** or **Reject**.  
  - Updates the Slack message in real-time once a decision is made.  

- 📧 **Polished Email Replies**  
  - Sends back HTML-styled professional responses (with letterhead, signature, contact info).  
  - Adds confidentiality footer & reference number.  

- 📝 **Audit Logging**  
  - Stores all requests, AI outputs, reviewer decisions, and timestamps in Supabase tables.  

---

## 🏗️ Architecture  

```mermaid
sequenceDiagram
  participant G as Gmail
  participant n as n8n Workflow
  participant O as OpenAI
  participant S as Supabase (KB + Queue)
  participant Sl as Slack

  G->>n: Gmail Trigger (new unread)
  n->>n: Label "Pending" + Mark as Read
  n->>O: Extract & Classify
  n->>S: Vector Search (knowledge_base)
  n->>O: Generate Draft Reply (RAG Agent)
  n->>n: Label "Processed"
  n->>S: Insert into approval_queue + approval_log
  n->>Sl: Send Slack message (Approve/Reject buttons)
  Sl->>n: Webhook (decision)
  alt Approved
    n->>G: Send formatted HTML reply
    n->>n: Label "Approved" (remove "Processed")
  else Rejected
    n->>n: Label "Rejected" (remove "Processed")
  end
  n->>Sl: Update Slack message with decision
````

---

## 🗂️ Repository Structure

```
student-affairs-email-agent/
├─ workflows/
│  └─ student_affairs_email_agent.public.json   # sanitized n8n export
├─ docs/
│  ├─ architecture.md
│  ├─ zoom.html             # interactive zoomable workflow viewer
│  └─ assets/
│      └─ workflow.png      # static screenshot of the n8n canvas
├─ sql/
│  ├─ knowledge_base.sql
│  ├─ approval_queue.sql
│  └─ approval_log.sql
├─ .env.example
├─ README.md
├─ LICENSE
└─ SECURITY.md
```

---

## ⚙️ Setup

### 1. Prerequisites

* [n8n](https://n8n.io/) running locally or hosted
* Supabase project with [pgvector](https://supabase.com/docs/guides/database/extensions/pgvector) enabled
* OpenAI API key
* Gmail API credentials (OAuth2)
* Slack App with channel + interactive buttons enabled

### 2. Database Setup

Run the SQL files in `/sql` inside your Supabase project:

```bash
psql -h db.supabase.co -U postgres -d postgres -f sql/knowledge_base.sql
psql -h db.supabase.co -U postgres -d postgres -f sql/approval_queue.sql
psql -h db.supabase.co -U postgres -d postgres -f sql/approval_log.sql
```

### 3. Environment Variables

Copy `.env.example` → `.env` and fill in:

```env
OPENAI_API_KEY=sk-xxxx
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your_anon_key
SLACK_CHANNEL_ID=C123456789
```

### 4. Import Workflow

* In n8n → Import → upload `workflows/student_affairs_email_agent.public.json`.
* Attach your credentials in the n8n UI.

### 5. Test Flow

1. Send an email to the monitored Gmail account.
2. Watch Slack → approve/reject.
3. Approved = professional email sent back, labeled `Approved`.
4. Rejected = labeled `Rejected`, logged in Supabase.

---

## 📸 Demo

* **Workflow Viewer:** [Zoom & pan view](./docs/zoom.html)
* **Slack Approval Example:**
  ![Slack Approval](docs/assets/slack-approval.png)
* **Approved Email Reply:**
  ![Email Reply](docs/assets/email-reply.png)

---

## 🔐 Security

⚠️ **Important:**

* Do **not** commit service keys (Supabase service\_role, OpenAI, Slack bot tokens, Gmail secrets).
* This repo uses placeholders (`{{OPENAI_API_KEY}}`) in the workflow JSON.
* See [SECURITY.md](SECURITY.md) for vulnerability disclosure.

---

## 📜 License

[MIT License](./LICENSE) © 2025 Helin Cinko

---

## 🙌 Acknowledgments

* [n8n](https://n8n.io/) for the automation framework
* [Supabase](https://supabase.com/) for database + vector search
* [OpenAI](https://openai.com/) for embeddings + GPT responses
* [Slack API](https://api.slack.com/) for approvals

---

## 🚀 Share

Built as a demonstration of **AI + workflow automation in higher education**.
See my [LinkedIn post](#) for the story behind this project.

```
