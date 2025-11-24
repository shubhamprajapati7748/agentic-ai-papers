# ZEP - Knowledge Graph Architecture for Agent Memory

- Paper Link: https://arxiv.org/pdf/2501.13956
- Miskies Slide: https://www.miskies.app/miskie/zep-research-paper-d32a9f

- Exisiting RAG limitation -> static document retrieval
- Enterprise applications -> Demands dynamic knowledge integration from diverse sources including ongoing conversations and business data

ZEP Address this with Graphiti
- A temporally-aware knowledge graph engine that dynamically synthesizes both unstructured conversational data and structured business data while maintaining historical relationship. 

-> Chat-based agents are limited to: 
- LLM's context windows
- Effective context utlizations 
- Knowledge grained during pre-training
- Additional Context is required to provide 
    - out of domain knowledge 
    - reduce hallucination 

> For agents to become pervasive in our daily lives -> Need to access Large corpus of continuously evolving data from user's interaction -> Along with Business + World data 

RAG not a good approach -> Because entire conversation histories, business datasets, and other domain-specific content cannot fit effectively inside LLM context windows

New Approach -> For agents using knowledge graphs to power LLM-agent memory


> ZEP - A memory Layer Service -> Powered by Graphiti
- It ingests and synthesizes both unstructured message data + Stuctured business data 
- Focus heavily on the accuracy, latency, and scalability of its memory retrieval mechanisms

> Graphiti - A dynamic, temporally-aware knowledge graph engine. 
- It updates the knowledge graph with new informations in a non-lossy manner, maintaining a timeline of facts and relationship, including their period of validity -> Enables KG to represent a complex, evolving world. 


## 2. Knowledge Graph Construction

In ZEP, memory is powered by a temporally-aware dynamic knowledge Graph 

G = (N , E, φ), where 
- N represents nodes, 
- E represents edges,
- φ : E → N × N represents a formal incidence function. 

This graph comprises three hierarchical tiers of subgraphs: 
- Episode Subgraph, 
- Semantic entity subgraph, 
- Community subgraph

Episode Subgraph Ge: 
- Episodic nodes (episodes), ni ∈ Ne, contain raw input data in the form of mes-
sages, text, or JSON. 
- Episodes serve as a non-lossy data store from which semantic entities and relations are extracted. 
- Episodic edges, ei ∈ Ee ⊆ φ∗(Ne × Ns), connect episodes to their referenced semantic entities.


Semantic Entity Subgraph Gs: 
- The semantic entity subgraph builds upon the episode subgraph. 
- Entity nodes (entities), ni ∈ Ns, represent entities extracted from episodes and resolved with existing graph entities.
- Entity edges (semantic edges), ei ∈ Es ⊆ φ∗(Ns × Ns), represent relationships between entities extracted from episodes.

Community Subgraph Gc: 
- The community subgraph forms the highest level of Zep’s knowledge graph.
- Community nodes (communities), ni ∈ Nc, represent clusters of strongly connected entities. 
- Communities contain high-level summarizations of these clusters and represent a more comprehensive, interconnected view of Gs’s structure. 
- Community edges, ei ∈ Ec ⊆ φ∗(Nc × Ns), connect communities to their entity members


The dual storage of both raw episodic data and derived semantic entity information mirrors psychological models of human memory. 
- These models distinguish between 
    - episodic memory, which represents distinct events, 
    - semantic memory, which captures associations between concepts and their meanings [8].

This approach enables LLM agents using Zep to develop more sophisticated and nuanced memory structures that better align with our understanding of
human memory systems.


Our use of community nodes to represent high-level structures and domain concepts builds upon work from GraphRAG enabling a more comprehensive global understanding of the domain. 

The resulting hierarchical organization from episodes to facts to entities to communities—extends existing hierarchical RAG strategies.

### 2.1 Episodes 

- ZEP Graph Construction begins with the ingestion of raw data units called "Episodes". Can be type of 
    - Messages 
    - Text 
    - Json 
- This reseach focus on Message type, it consists of short text along with
    - Associated actor who produces it.
    - Reference timestamp - when the message was sent 
- This temporal information enables Zep -> identify and extract relative or partial dates mentioned in the message content 
    - It provies dynamic nature of conversational data and memory

This bi-temporal approach represents a novel advancement in LLM-based knowledge graph construction and underlies much of Zep’s unique capabilities compared
to previous graph-based RAG proposals.

- The episodic edges, Ee, connect episodes to their extracted entity nodes.
-  Episodes and their derived semantic edges maintain bidirectional indices that track the relationships between edges and their source episodes.

This design reinforces the non-lossy nature of Graphiti’s episodic subgraph by enabling both forward and backward traversal:
- semantic artifacts can be traced to their sources for citation or quotation, - while episodes can quickly retrieve their relevant entities and facts

### 2.2 Semantic Entities and Facts

#### 2.2.1 Entities 

Entity extraction represents the initial phase of episode processing.
- During ingestion, the system processes both the current **message content** and the last n messages to provide context for named entity recognition.
- Given our focus on message processing, the **speaker** is automatically extracted as an **entity**
- Following initial entity extraction, we employ a reflection technique inspired by reflexion[12] to minimize hallucinations and enhance extraction coverage.
- The system also extracts an entity summary from the episode to facilitate subsequent entity resolution and retrieval operations


After extraction, the system embeds each entity name into a **1024-dimensional** vector space. 
- This embedding enables the retrieval of similar nodes through cosine *similarity search* across existing graph entity nodes.
- The system also performs a *separate full-text search* on existing entity names and summaries to identify additional candidate nodes.
- These candidate nodes, together with the episode context, are then processed through an LLM using our entity resolution prompt. 
- When the system identifies a duplicate entity, it generates an updated name and summary.


Following entity extraction and resolution, 
- the system query the data into the knowledge graph using *predefined Cypher queries*. 
- We chose this approach over LLM-generated database queries to ensure consistent schema formats and reduce the potential for hallucinations.

#### 2.2.2 Facts

Each fact containing its key predicate.
- the same fact can be extracted multiple times between different
entities,
- enabling Graphiti to model complex multi-entity facts through an implementation of hyper-edges.

Following *extraction*, the system *generates embeddings* for facts in preparation for graph integration.
- The system performs edge *deduplication* through a process similar to entity resolution
- The *hybrid search* for relevant edges is constrained to edges existing between the same entity pairs as the proposed new edge
-  This constraint not only prevents *erroneous combinations* of similar edges between different entities but also significantly reduces the *computational complexity* of the deduplication process by limiting the search space to a subset of edges relevant to the specific entity pair.


#### 2.2.3 Temporat Extraction and Edge Invalidation 

A key *differentiating* feature of Graphiti compared to other knowledge graph engines is its capacity to manage *dynamic information updates through temporal extraction and edge invalidation processes*

- The system extracts temporal information about facts from the episode context using. This enables *accurate extraction and datetime representation* of both absolute timestamps

- The introduction of new edges can invalidate existing edges in the database
-  The system employs an LLM to compare new edges against semantically related existing edges to identify potential contradictions. 

This comprehensive approach enables the dynamic addition of data to Graphiti as conversations evolve, while maintaining both current relationship states and historical records of relationship evolution over time.


###  2.3 Communities
After establishing the episodic and semantic subgraphs, the system constructs the community subgraph through community detection. 

- This choice was influenced by label propagation’s straightforward dynamic extension, which enables the system to maintain accurate community representations for longer periods as new data enters the graph, delaying the need for complete community refreshes.

- The dynamic extension implements the logic of a single recursive step in label propagation. 

When the system adds a new entity node ni ∈ Ns to the graph, it surveys the communities of neighboring nodes. 

The system assigns the new node to the community held by the plurality of its neighbors, then updates the community summary and graph accordingly. 

While this dynamic updating enables efficient community extension as data flows into the system, the resulting communities gradually diverge from those that would be generated by a complete label propagation run. 

Therefore, periodic community refreshes remain necessary. 

However, this dynamic updating strategy provides a practical heuristic that significantly reduces latency and LLM inference costs.

Following [4], our community nodes contain summaries derived through an iterative map-reduce-style summarization of member nodes. 

However, our retrieval methods differ substantially from GraphRAG’s map-reduce approach [4]. To support our retrieval methodology, we generate community names containing key terms and relevant subjects from the community summaries. These names are embedded and stored to enable cosine similarity searches.

