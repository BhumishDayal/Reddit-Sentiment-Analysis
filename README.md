# 🧠 Reddit Sentiment Analysis Workflow using BrightData, LangChain, and Google Sheets

This `n8n` workflow is a **complete pipeline for scraping Reddit discussions**, classifying them by topic relevance, analyzing **user sentiments with LLMs**, and exporting the results to **Google Sheets** for reporting or visualization.

It is ideal for tracking public reactions to events like **Apple's WWDC**, analyzing trends in subreddits, or building sentiment dashboards across multiple posts and comments.

### 🔄 What the Workflow Does

1. **Scrapes Reddit Posts**: Uses the BrightData Dataset API to collect Reddit posts about specific keywords (e.g., `WWDC25`).
2. **Monitors Scraping Progress**: Checks the status of the scraping process and waits until the results are ready.
3. **Classifies Posts**: Uses a text classifier to categorize each post (e.g., WWDC-related or not).
4. **Performs AI-based Sentiment Analysis**: Breaks down each post's comments and uses Google Gemini via LangChain to detect nuanced sentiments (e.g., Excited, Skeptical, etc.).
5. **Logs Results to Google Sheets**: Posts and filtered comment sentiments are recorded into a structured Google Sheet for further use.

This workflow can run manually or be scheduled, and it combines API integration, conditional logic, LLM intelligence, and spreadsheet logging in one seamless system.

## ⚙️ Prerequisites

1. An [n8n](https://n8n.io/) environment
2. API keys and accounts for:
   - BrightData Dataset API
   - Google Sheets OAuth
   - Google Gemini API (via Langchain)

---

## 🧠 How the Workflow Works

---

### 1. **Manual Trigger**
- **Node**: `When clicking ‘Execute workflow’`
- **Purpose**: Manually initiates the workflow.
- **Setup**: No changes needed.

---

### 2. **Trigger BrightData Scraping**
- **Node**: `scrap reddit`
- **Type**: HTTP Request (POST)
- **Setup**:
  - URL: `https://api.brightdata.com/datasets/v3/trigger`
  - Headers:
    ```
    Authorization: Bearer YOUR_BRIGHTDATA_API_KEY
    ```
  - JSON Body:
    ```json
    [
      {
        "keyword": "WWDC25",
        "date": "Past month",
        "num_of_posts": 100,
        "sort_by": "New"
      }
    ]
    ```
  - Query Parameters:
    - `dataset_id`: `gd_lvz8ah06191smkebj4`
    - `include_errors`: `true`
    - `type`: `discover_new`
    - `discover_by`: `keyword`

---

### 3. **Check Scraping Status**
- **Node**: `Get status`
- **Type**: HTTP Request (GET)
- **URL**: `https://api.brightdata.com/datasets/v3/progress/{{ $json.snapshot_id }}`
- **Setup**: Use the same BrightData API key in the header.

---

### 4. **Switch on Scrape Status**
- **Node**: `Switch`
- **Conditions**:
  - If `status == ready` → proceed to data fetch
  - If `status == running` → wait 15 seconds

---

### 5. **Wait Node**
- **Node**: `Wait`
- **Setup**: Delay 15 seconds and retry checking status

---

### 6. **Get Scraped Data**
- **Node**: `Get data`
- **Type**: HTTP Request (GET)
- **URL**: `https://api.brightdata.com/datasets/v3/snapshot/s_mbtsa7xhm9v2pdx3l?format=json`
- **Header**: Same BrightData authorization token

---

### 7. **Loop Over Each Post**
- **Node**: `Loop Over Items`
- **Purpose**: Handles each post individually to avoid AI overload

---

### 8. **Text Classification (WWDC Relevance)**
- **Node**: `Text Classifier`
- **Setup**:
  - `Post title` and `Post description` as input
  - Categories:
    - `WWDC events`: Posts about Apple and WWDC
    - `Other`: Anything else

---

### 9. **Route Non-WWDC Posts**
- **Node**: `No category`
- **Purpose**: Bounces irrelevant posts back into loop

---

### 10. **Transform Post Fields**
- **Node**: `Edit Fields` (Set node)
- **Fields**:
  - `post_id`, `url`, `user_posted`, `title`, `description`, `comments`, etc.

---

### 11. **Split Comments for Analysis**
- **Node**: `Split Out`
- **Field**: `comments`

---

### 12. **Prepare Comment for AI**
- **Node**: `edit for Sentiment analysis`
- **Fields Set**:
  - `comment`, `user_commenting`, `url`, `description`, `title`

---

### 13. **Sentiment Analysis**
- **Node**: `Sentiment Analysis per comment` (Langchain)
- **Model**: `Google Gemini`
- **Prompt Structure**:
  - Title, Description, and Comment
  - System prompt guides it to choose from:
    - Excited, Disappointed, Impressed, Indifferent, Skeptical, Not clear

---

### 14. **Merge Results**
- **Node**: `Merge`
- **Purpose**: Combine sentiment and original post context

---

### 15. **Format Sentiment**
- **Node**: `format sentiment` (Set node)
- **Sets**: `title`, `url`, `post_id`, `community_name`, `sentimentAnalysis`, etc.

---

### 16. **Filter Valid Sentiments**
- **Node**: `Filter`
- **Condition**: Exclude comments with `sentimentAnalysis == "Not clear"`

---

### 17. **Append to Google Sheet**
- **Node**: `Append Sentiments`
- **Setup**:
  - Sheet: `APPLE #WWDC25 Sentiments`
  - Tab: `Reddit Comments`
  - Method: `appendOrUpdate`
  - Mapping: Uses `url` as key
  - Columns:
    - `comment`, `user_commenting`, `url`, `description`, `title`, `sentimentAnalysis`, `post_id`, `community_name`

---

## ✅ Final Notes

- Set up your **BrightData API key**, **Google Sheets OAuth**, and **Langchain Gemini** credentials before running.
- This workflow supports real-time Reddit monitoring + emotional classification using LLMs.
- Consider scheduling the initial trigger node for automatic scraping.

