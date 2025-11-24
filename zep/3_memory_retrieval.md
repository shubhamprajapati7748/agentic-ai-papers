# Memory Retrieval 

To make this easy, I’ll use a **simple story**, and then show:

1. **What the user asks**
2. **What the retrieval system searches for**
3. **How multiple search methods bring different candidates**
4. **How the reranker decides which ones matter most**
5. **What final memory is returned to the LLM**

Let’s dive in.

---

# 🧠 Scenario Setup (Knowledge Stored So Far)

Imagine the memory graph contains these facts:

### 💬 Conversations (episodes)

* “I bought a **MacBook Air** last month.”
* “I also got an **iPhone 15** last week.”
* “My **old Dell laptop** was too slow.”
* “I prefer **lightweight laptops** for travel.”
* “I might switch to a **gaming laptop** soon.”

### 🧩 Entities & Facts

* User → purchased → MacBook Air (valid: last month)
* User → purchased → iPhone 15 (valid: last week)
* User → previously_owned → Dell Laptop
* User → preference → lightweight laptops
* User → considering → gaming laptop

---

# 🎯 User Query

**“What laptop does the user currently use?”**

The goal of retrieval is to fetch only the **most relevant memories**, *not the entire conversation*.

---

# 🔍 SECTION 3 — RETRIEVAL WITH SEARCH + RERANKING (Illustrated)

Zep’s retrieval pipeline has **3 stages**:

### **A. SEARCH**

Find all possibly relevant memories using multiple methods.

### **B. RERANKING**

Sort & filter them by relevance and importance.

### **C. CONSTRUCTION**

Format final facts/entities into a compact context for the LLM.

Let’s break it down with our example.

---

# 🅰️ Step A: SEARCH (Multiple Search Methods)

Zep doesn’t depend on *one* search method.
It runs **3 parallel searches** and merges their results.

---

## 🔸 1. **Embedding Similarity Search**

Meaning-based search using vector embeddings.

Query:

> “What laptop does the user currently use?”

Closest memories:

* “User purchased **MacBook Air**”
* “User previously owned **Dell laptop**”
* “User prefers lightweight laptops”

---

## 🔸 2. **Full-Text Search (BM25)**

Keyword-based search on raw text.

Matching keywords:

* “laptop”
* “MacBook”
* “Dell laptop”
* “gaming laptop”

Candidates returned:

* “My old Dell laptop was too slow”
* “I bought a MacBook Air”
* “I might switch to a gaming laptop soon”
* “I prefer lightweight laptops”

---

## 🔸 3. **Graph Search (BFS/Neighborhood Search)**

Graph traversal around key entities:

* Laptop
* User
* Purchase events
* Last known devices

Graph search returns:

* Edges linked to “User”—“purchased”—“MacBook Air”
* Edges linked to “User”—“owned”—“Dell laptop”
* “preference → lightweight laptops”

---

# 🟦 Combined Candidate List

All candidates (unsorted for now):

1. Purchased MacBook Air
2. Old Dell laptop
3. Might switch to gaming laptop
4. Prefers lightweight laptops
5. Bought iPhone 15 (irrelevant)
6. Purchased last week (irrelevant)

---

# 🅱️ Step B: RERANKING (This is where magic happens)

Multiple reranking strategies are used:

### 1. **RRF (Reciprocal Rank Fusion)**

Combines ranks from all three searches to get balanced scoring.

### 2. **MMR (Maximal Marginal Relevance)**

Ensures diversity — avoids retrieving redundant facts.

### 3. **Graph distance weighting**

Facts directly connected to the user entity receive higher relevance.

### 4. **Entity mention frequency**

Frequently referenced entities are boosted.

### 5. **Cross-encoder scoring**

An LLM-based pairwise ranking for relevance to the question.

// Now let’s see what happens.

---

## 🟩 Reranking Example Outcome

After applying rerankers:

### **Top 1 (Most relevant)**

✔ **User purchased MacBook Air (last month, still valid)**
Reason: Strong signal for “currently used laptop”

### **Top 2**

✔ **User previously owned Dell laptop (expired fact)**
Useful as historical context, but ranked lower.

### **Top 3**

✔ **User prefers lightweight laptops**
Useful for extra reasoning (MacBook Air is lightweight)

### **Excluded**

❌ “I might switch to a gaming laptop” → low relevance
❌ “iPhone 15” → irrelevant
❌ Minor conversations → irrelevant

---

# 🅲 Step C: Context Construction

Now the system builds a clean, compact **context block** for the LLM.

---

## **📝 Final Constructed Memory Output**

*(This is exactly what Zep would return to the model)*

```
FACTS (Date range: Dec 2024 – Jan 2025)
- The user purchased a MacBook Air last month.
- The user previously owned a Dell laptop, which was slow.
- The user prefers lightweight laptops.

ENTITIES
USER:
- Recently bought a MacBook Air.
- Prefers lightweight devices for travel.

MacBook Air:
- User’s current laptop.
- Purchased last month.

Dell Laptop:
- Previously used by the user.
- Was too slow, leading to upgrade.
```

Now if the LLM receives this context, it can confidently answer:

> “The user currently uses a **MacBook Air**, which they purchased last month.”
