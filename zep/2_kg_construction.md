
# ✅ 1. Episode Subgraph (Gₑ) – “Raw Memory Layer”

Think of this layer as the **diary** of everything the system sees.

### What is stored here?

* Every incoming **message**, **event**, or **JSON object**
* Exactly *as it was received*
* With **timestamps**, **who said it**, and **metadata**

### Example

User and Agent have this conversation:

```
User (2025-01-05 10:30): I bought a MacBook Air yesterday.
Assistant: Oh nice! When did you decide to upgrade?
User: Last week. My old laptop was too slow.
```

The Episode Subgraph stores these as separate nodes:

| Episode Node ID | Content                             | Timestamp   |
| --------------- | ----------------------------------- | ----------- |
| E1              | “I bought a MacBook Air yesterday.” | Jan 5 10:30 |
| E2              | “When did you decide to upgrade?”   | Jan 5 10:30 |
| E3              | “Last week…”                        | Jan 5 10:31 |

No processing. No summarization.
Just **raw memory**.

This ensures:
✔ Nothing is lost
✔ Future extraction can be more accurate
✔ You can re-extract facts if LLMs improve

---

# ✅ 2. Semantic Entity Subgraph (Gₛ) – “Understanding the World Layer”

This layer tries to **understand what each episode means**.

Zep uses LLMs and embeddings to pull out:

### A. **Entities**

These are “things in the world”:

* People
* Places
* Devices
* Organizations
* Concepts

### B. **Facts (Edges)**

Relationships between entities:

* “User → purchased → MacBook Air”
* “MacBook Air → purchase_date → Jan 4”
* “User → had_old_device → Laptop”

---

## 🔍 Example (continued)

From E1:

> "I bought a MacBook Air yesterday."

Zep extracts:

### Entities:

* **User**
* **MacBook Air**
* **Purchase Event**

### Fact:

```
(User) --purchased_on(2025-01-04)--> (MacBook Air)
```

From E3:

> “Last week. My old laptop was too slow.”

Zep extracts:

### Entities:

* Old Laptop
* Purchase Decision Time

### Facts:

```
(User) --decided_upgrade_on(2024-12-29)--> (MacBook Air)
(User) --previous_device--> (Old Laptop)
```

---

## 🧠 Important: Entity Resolution

Different messages may refer to the same entity in different ways.

Example:

* “MacBook Air”
* “My new MacBook”
* “The Apple laptop”

Zep needs to detect they’re the **same object**.

How it works:

1. Embed names
2. Compare similarity
3. Sometimes ask LLM: “Are these the same entity in this context?”

So all references collapse into a single entity node:

```
(MacBook Air)
```

This avoids duplication and creates long-term consistency.

---

## 🧩 Important: Fact Deduplication

If a user repeatedly says the same fact:

* “I live in Mumbai.”
* “My home is in Mumbai.”

Zep avoids adding duplicate edges; instead, it updates confidence or timestamps.

---

## 🕒 Temporal Extraction (critical feature)

Zep reads **when facts are valid**, not just what they are.

Example from the conversation:

* “I bought a MacBook Air yesterday.” → valid on **Jan 4**
* “Last week I decided to upgrade.” → valid on **Dec 29**

These become **temporal edges** stored with validity windows:

```
(User) --purchased(MacBook Air)--> valid: Jan 4 2025
(User) --decided_upgrade--> valid: Dec 29 2024
```

This is the “temporal knowledge graph” innovation.

---

# ✅ 3. Communities Subgraph (G₍c₎) – “Higher-Level Understanding Layer”

Once the graph has a bunch of entities and facts, Zep tries to find **clusters** of related concepts.

This helps the memory system understand *themes* in a user’s life or business environment.

### How communities are built:

* Use **label propagation algorithm**
* Cluster entities that:

  * Frequently co-occur
  * Are closely connected
  * Belong to same topic

### Example Community

From earlier conversation:

Entities involved:

* MacBook Air
* Old Laptop
* Upgrade Decision
* Purchase Event
* User
* Laptop Performance

They cluster into a **community** called:

```
“Laptop Upgrade Journey”
```

The system also creates:

* **Community summary**
* **Representative embedding**
* **Relationships to member entities**

### Why communities matter?

They help in:

* Fast retrieval
* Higher-level reasoning
* Structured summarization
* Topic-based memory queries (e.g., "What does the user prefer in laptops?")

---

# 🎯 Putting It All Together (Fully Explained)

### When user messages come in, Zep does:

---

### **Step 1: Store raw message → Episode node**

Good for:

* Auditing
* Reference
* Re-extracting later

---

### **Step 2: Extract entities & facts → Entities Subgraph**

Good for:

* Reasoning
* Retrieval
* State tracking

---

### **Step 3: Group related entities → Communities**

Good for:

* Topic-level memory
* Fast clustering
* Organized context retrieval

---

# 🚀 A Super Simple Mini Example

Let’s take just one message:

> "My sister Riya moved to Pune last month."

### Episode added:

```
E1: “My sister Riya moved to Pune last month.”
```

### Entities extracted:

* User
* Riya
* Pune
* Move Event

### Facts extracted:

```
(Riya) --is_sister_of--> (User)
(Riya) --moved_to--> (Pune) [valid: last month → e.g., 2024-12-01]
```

### Community detection:

Entities cluster under:

```
“Family & Residence Community”
```

With relationships:

* Riya → User
* Riya → Pune
* Move Date

Now the agent can answer:

* “Where does Riya live now?”
* “When did she move?”
* “Who in the user’s family recently changed residence?”

All because of the layered memory graph.