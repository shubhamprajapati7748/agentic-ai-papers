# ProgRAG: Hallucination-Resistant Progressive Retrieval and Reasoning over Knowledge Graphs

- Authors: Minbae Park, Hyemin Yang, Jeonghyun Kim, Kunsoo Park, Hyunjoon Kim
- Paper Link: https://arxiv.org/abs/2511.10240

## What is ProgRAG?

It’s a method for answering complex multi-hop questions over a knowledge graph (KG) by combining
- progressive question decomposition, 
- retrieval from the KG, 
- and LLM reasoning 
- designed to reduce hallucinations and improve grounded reasoning.

`Solve both Retrival error + Reasoning error`

## Why do we need it?

- Large language models (LLMs) are good at reasoning, but when you simply give them a large context of KG facts or ask them to retrieve everything themselves, they tend to hallucinate or lose precision. 

- Existing “retrieval-augmented” or KG-enhanced LLMs still struggle when the question requires multiple hops (relations) in the KG, and when the input context is long/complex. 

So the authors propose a structured pipeline: 
```
break a complex question down → retrieve relevant evidence per sub-question → prune/clean evidence → feed that to the LLM to reason.
```

## The 3 Key Steps of ProgRAG

### 1. Decompose the question → sequential sub-questions

A complex question (multi-hop) is decomposed into a sequence of simpler sub-questions (rather than everything in one go). 

For example: Q: “Which composer wrote the work performed at the venue where the athlete X won gold?”

- SubQ1: “Which venue did athlete X win gold at?”
- SubQ2: “Which work was performed at that venue?”
- SubQ3: “Who composed that work?”

The number of sub-questions = “exploration depth” of its reasoning path. They dynamically choose how deep to go to balance “enough reasoning” vs “over-exploration”.

This helps ensure the reasoning path is ***progressive***, clear, and less likely to go off tangent.

### 2. Retrieval + pruning for each sub-question

For each sub-question:
1. Use an external retriever to fetch relevant evidence (triples/nodes) from the KG.
2. Feed these to the LLM which does uncertainty-aware pruning — i.e., the LLM evaluates which retrieved pieces are likely useful and filters out noise.

This “retrieve then prune” loop means the LLM reasons over a narrowed search space, improving precision.

Also: when the LLM is uncertain (prediction confidence low), the system increases reliance on external knowledge (retriever) rather than purely relying on the LLM’s own memory. 

**Effect: less hallucination, more grounded evidence.**

### 3. Refine the context input for the LLM (prefix enumeration & repacking)

Once sub-questions are answered and partial reasoning paths built, the next step is to *structure the input context* for the LLM well. 

Two techniques:
- **Prefix enumeration**: enumerate possible reasoning prefixes (paths) to structure the way evidence + sub-answers are arranged.
- **Repacking**: reorganise (pack) the evidence + reasoning chain into a compact, well-ordered input to the LLM.

```
A messy or overly large context can confuse an LLM (it might ignore some evidence or focus on irrelevant parts). By carefully structuring, they improve accuracy and reduce hallucinations. 
```

### Why it works / benefits

- By decomposing, retrieval is done at each step → smaller, more relevant chunks → reduces combinatorial explosion of paths in the KG.
- By pruning with LLM uncertainty, they avoid “garbage evidence” being fed.
- By structuring input context, they improve the LLM’s focus and reduce hallucinated leaps.

Result into -> state-of-the-art performance on three KGQA benchmarks:
- On WebQSP: +3.3% accuracy vs best baseline. 
- On CWQ: +4.9%. 
- On CR‑LT: +10.9%. 

The approach thus helps reliability and reasoning quality for KG-based QA.

### Short Summary

ProgRAG is a method for multi-hop knowledge-graph question answering that avoids hallucination by:
1. Breaking a complex question into a sequence of simpler sub-questions (depth controlled).

2. For each sub-question, retrieving relevant evidence from the KG and letting the LLM prune it based on uncertainty (reducing noise).

3. Structuring the reasoning path and input context (via prefix enumeration/repacking) so the LLM sees a clean, focused context — which leads to more accurate answers.
The result: stronger grounding, fewer hallucinations, improved performance on benchmark datasets.