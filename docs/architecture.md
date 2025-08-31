sequenceDiagram
  participant G as Gmail
  participant n as n8n Workflow
  participant O as OpenAI
  participant S as Supabase (KB + Queue)
  participant Sl as Slack

  G->>n: Gmail Trigger (new unread, not spam, not labeled 'processed')
  n->>n: Mark as read + Label 'Pending'
  n->>O: Extract + Classify (primary_question, category, urgency)
  n->>n: Build RAG prompt
  n->>S: Vector search (knowledge_base)
  n->>O: Draft response (agent w/ tool)
  n->>n: Label 'Processed'
  n->>S: Insert approval_queue + approval_log
  n->>Sl: Post approval message (buttons)
  Sl->>n: Webhook (approve/reject)
  n->>S: Update status (+log)
  alt Approved
    n->>G: Send formatted HTML reply
    n->>n: Label 'Approved' (remove 'Processed')
  else Rejected
    n->>n: Label 'Rejected' (remove 'Processed')
  end
  n->>Sl: Update Slack message (final state)
