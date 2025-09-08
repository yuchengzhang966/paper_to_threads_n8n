# 🚀 Academic Paper to Twitter Threads n8n Workflow

Tired of seeing great research get lost in the digital void? This workflow is your personal AI assistant for turning dense academic PDFs into viral Twitter (X) threads!

It intelligently analyzes any paper from a URL, drafts an entire, engaging thread, and stages it in Airtable for your review. Once you're happy, a second trigger publishes the thread for you, tweet by tweet. It's designed to be resilient—if it gets interrupted, you can pick up right where you left off without any duplicate posts.

## ✨ Why Use This Workflow?

-   **Save Hours of Work**: Stop manually summarizing papers and crafting tweets. Let AI do the heavy lifting.
-   **Boost Your Reach**: Translate complex ideas into accessible content that a wider audience can understand and share.
-   **Maintain Full Control**: Generate content in batches, review and edit everything in Airtable, and publish only when you're ready.
-   **Publish with Confidence**: The built-in checks prevent duplicate posts, even if your workflow is interrupted.

## 🧠 Core Features

-   🤖 **AI-Powered Analysis**: Leverages Google Gemini to read and deeply understand a PDF, extracting key arguments, data, and highlights.
-   ✍️ **Expert-Level Copywriting**: A second AI step, acting as a social media pro, transforms the analysis into a polished, hook-driven Twitter thread.
-   🗄️ **Content Hub & Cache**: Uses Airtable as a central database to store, review, and manage all generated tweets before and after publishing.
-   🛡️ **Idempotent Publishing**: The workflow smartly checks if a tweet has already been posted, preventing duplicates and allowing you to safely re-run the publishing job.
-   🧑‍💻 **Human-in-the-Loop Design**: Content generation and publishing are separated, giving you full control to review, edit, and approve content before it goes live.

## ⚙️ How It Works

The workflow is split into two powerful phases:

### Phase 1: 📝 Content Generation & Storage

![Content Generation Flow](./images/phase-1-generation.png)
*The generation phase: A PDF URL is processed by AI, and the resulting tweets are stored in Airtable.*

1.  **Trigger (PDF URL)**: The workflow starts when a form is submitted with a URL to a PDF document.
2.  **Check for Duplicates**: It first queries Airtable to see if this document has been processed before. If so, it stops this branch of the workflow.
3.  **AI Analysis**: If it's a new document, the PDF URL is sent to a Google Gemini node. A detailed prompt instructs the AI to act as a research expert and extract the paper's title, main arguments, supporting points, and interesting highlights.
4.  **AI Copywriting**: The structured analysis from the first step is then passed to a second Gemini node, which acts as an expert social media copywriter. This node drafts the Twitter thread based on strict formatting rules (e.g., character limits, tone, hook-driven first tweet).
5.  **Data Structuring**: The AI's JSON output is parsed and then flattened into individual items, where each item represents a single tweet.
6.  **Store in Airtable**: Each tweet is saved as a new row in an Airtable table, containing its text, position in the thread, and other metadata. At this point, no tweets have been published.

### Phase 2: 🐦 Publishing to Twitter

![Publishing Flow](./images/phase-2-publishing.png)
*The publishing phase: Tweets are fetched from Airtable and posted sequentially to Twitter.*

1.  **Trigger (Document Title)**: This phase is initiated by submitting a form with the document's title.
2.  **Fetch Tweets**: The workflow retrieves all tweet records for that title from Airtable, sorted by their position.
3.  **Loop & Publish**: The workflow iterates through each tweet record one by one.
4.  **Check if Published**: For each tweet, it checks if an `x_id` (the ID of the post on X) already exists in its Airtable row. If it does, it skips to the next tweet.
5.  **Post Tweet**:
    -   If it's the first tweet in the thread (`tweet_pos` = 1), it's posted as a new status.
    -   If it's a subsequent tweet, it's posted as a reply to the previously posted tweet.
6.  **Update Airtable**: After a tweet is successfully posted, the workflow updates the corresponding Airtable row with the new `x_id`.
7.  **Wait**: A 10-minute wait is included between posts to respect API rate limits and ensure the thread posts correctly.

## 🛠️ What You'll Need

Before you can use this workflow, you will need:

-   ✅ An active **n8n** instance (Cloud or self-hosted).
-   ✅ **Google Gemini API Key**.
-   ✅ **Twitter (X) API v2 Credentials** with write permissions.
-   ✅ **Airtable Account** and a **Personal Access Token**.

## 🚀 Get Started in 3 Steps

### 1. Set Up Your Airtable Base

You need to create an Airtable base to store the tweet data.

1.  Create a new Base in Airtable (e.g., "AI Content").
2.  Create a table within that base (e.g., "Twitter Threads").
3.  Add the following fields. **The field names must match exactly** for the workflow to function correctly.

| Field Name  | Field Type         | Description                                                  |
| :---------- | :----------------- | :----------------------------------------------------------- |
| `tweet_key` | Single line text   | A unique key for each tweet (auto-generated).                |
| `doc_title` | Single line text   | The title of the source document.                            |
| `thread_pos`| Number             | The position of the thread (for multi-thread documents).     |
| `tweet_pos` | Number             | The position of the tweet within its thread (1, 2, 3...).    |
| `text`      | Long text          | The content of the tweet.                                    |
| `media_ids` | Single line text   | Comma-separated list of media IDs (future use).              |
| `x_id`      | Single line text   | The ID of the tweet after it has been posted to X (Twitter). |

!Airtable Base Setup
*The required fields and their types in your Airtable table.*

### 2. Import & Configure in n8n

1.  **Import Workflow**: In your n8n canvas, select **Import from File** and upload the `academic paper to twitter threads.json` file.
2.  **Configure Credentials**:
    -   Locate the **Google Gemini** nodes ("Read the document", "Generate the X posts") and select your configured Gemini API credential from the dropdown.
    -   Locate the **Airtable** nodes (e.g., "Search records1", "Create a record1"). Select your Airtable credential and update the **Base** and **Table** properties to match the base and table you created in the previous step.
    -   Locate the **Twitter** nodes ("Create the first time posts", "Create Tweet2") and select your Twitter API credential.
3.  **Activate the Workflow**: Save and activate the workflow.
4.  **You're ready to go!**

## 🎬 How to Use It

### Step 1: Generate Tweet Content ✍️

1.  With the workflow active, find the **"On form submission2"** trigger node.
2.  Open its **Test URL**.
3.  In the form, paste the URL of the academic paper PDF into the `pdfUrl` field and submit.

!Generate Content Form
*The form used to kick off the content generation process.*

4.  The workflow will run the generation phase. You can check your Airtable base to see the newly created rows for each tweet.

### Step 2: Review & Publish Your Thread 🚀

1.  Once you have reviewed the content in Airtable and are ready to post, find the **"On form submission1"** trigger node.
2.  Open its **Test URL**.
3.  In the form, enter the exact title of the document as it appears in the `doc_title` column in Airtable and submit.

!Publish Thread Form
*The form used to publish a pre-generated thread.*

4.  The workflow will begin publishing the tweets one by one, updating each row with the corresponding `x_id` after it's posted.

## 🧑‍💻 A Deep Dive into the Code Nodes

For those who love to tinker, here's a look under the hood at the custom JavaScript that powers this workflow's data handling.

### 1. The JSON Ninja (`Code2`, `turn into JSON data1`)

*   **Purpose**: To safely extract a valid JSON object from the raw text response of the Google Gemini AI.
*   **The Challenge**: LLMs are brilliant but sometimes messy. They might wrap their perfect JSON response in chatty text or markdown code blocks (e.g., ` ```json ... ``` `). A normal parser would choke on this.
*   **The Solution**: This script is a JSON-finding ninja.
    1.  It strips any markdown code fences from the AI's text output.
    2.  It first attempts to parse the cleaned text directly.
    3.  If that fails, it scans the text to find the first complete, balanced JSON structure (`{...}` or `[...]`) and attempts to parse only that segment.
*   **Usage in Workflow**: This code appears twice. It's used after *each* call to the Gemini AI (once after the initial analysis and once after the tweet generation) to ensure the subsequent steps receive clean, usable JSON data.

### 2. The Digital Rolling Pin (`Flatten for airtable1`)

*   **Purpose**: To convert the nested JSON structure of threads and tweets into a flat list of individual tweet objects, ready for database insertion.
*   **The Challenge**: The AI gives us a neat, nested package of threads and tweets. But to get it into our Airtable database (one row per tweet), we need to flatten it out.
*   **The Solution**: This script is the digital rolling pin that does just that.
    1.  The script iterates through the `threads` array and then the nested `tweets` array from the AI's output.
    2.  For each tweet, it creates a simple, single-level object with the exact field names required by the Airtable table (`doc_title`, `tweet_pos`, `text`, etc.).
    3.  It returns an array of these simple objects. The n8n workflow then processes each object in this array as an individual item, allowing the "Airtable Create" node to create one row per tweet.

```javascript
// Example of the "Flatten for airtable1" node's logic
const docTitle = $input.first().json.threads[0].title || "Untitled";
const out = [];

for (const thread of ($input.first().json.threads || [])) {
  for (const tw of (thread.tweets || [])) {
    out.push({
      json: {
        doc_title: docTitle,
        thread_pos: thread.position,
        tweet_pos: tw.position,
        text: tw.text,
        // ... and other fields
      }
    });
  }
}
return out;
```
