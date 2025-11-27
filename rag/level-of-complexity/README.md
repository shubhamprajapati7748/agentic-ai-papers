# Levels of Complexity: RAG Applications

- Article Link : https://jxnl.github.io/blog/writing/2024/02/28/levels-of-complexity-rag-applications/

## Level 1 - The Basics 

This level is designed to introduce you to the core concepts and techniques essential for working with RAG models. 

By the end of this section, you'll have a solid understanding of 
- how to traverse file systems for text generation
- chunk and batch text for processing,
- and interact with embedding APIs.

Let's dive in and explore the exciting capabilities of RAG applications together!

- Recursively traverse the file system to generate text.
- Utilize a generator for text chunking.
- Employ a generator to batch requests and asynchronously send them to an embedding API.
- Store data in LanceDB.
- Implement a CLI for querying, embedding questions, yielding text chunks, and generating responses.

## Level 2: More Structured Processing

Here, we focus on enhancing the efficiency and effectiveness of our RAG applications through 
- better asynchronous programming, 
- improved chunking strategies, 
- and robust retry mechanisms in processing pipelines.

Processing
- Better Asyncio
- Better Chunking
- Better Retries

Search
- Better Ranking (Cohere)
- Query Expansion / Rewriting
- Parallel Queries

Answering
- Citating specific text chunks
- Streaming Response Model for better structure.

## Level 3: Observability

This stage emphasizes the importance of implementing 
- comprehensive logging mechanisms to monitor 
- and measure the multifaceted performance of your application.

Establishing robust observability allows you to swiftly pinpoint and address any bottlenecks or issues, ensuring optimal functionality.

### Log the Citation 

By logging the citations:
- we can quickly understand if the model is citing the correct information, - what text chunks are popular, 
- review and understand if the model is citing the correct information. 
- and also potentially build a model in the future that can understand what text chunks are more important.

### Log mean cosine scores and reranker scores

By attaching this little metadata, we will be able to very cheaply identify queries that may be performing poorly.

### Log user level metadata for the search

By including other group information information, we can quickly identify if a certain group is having a bad experience

Examples could be: 
- Organization ID
- User ID
- User Role
- Signup Date
- Device Type
- Geo Location
- Language

This could help you understand a lot of different things about how your application is being used.
- Maybe people on a different device are asking shorter queries and they are performing poorly. 
- maybe when a new organization signed up, the types of questions they were asking were being served poorly by the language model


>  Review these things during stand-up once a week, look at some examples, and figure out what we could do to improve our system. 

When we see poor scores, we can look at the query and the rewritten query and try to understand what exactly is going on.

### Have Users

You've already set yourself for success by 
- understanding queries, 
- rewriting them, 
- and monitoring how users are actually using your system. 

## Level 4: Evaluations

Evaluation -  Crucial for understanding the performance and effectiveness of our systems.

Primarily, we are dealing with two distinct systems: 
- search system and 
- question answering (QA) system. 

It's common to see a lot of focus on *evaluating the QA system* given its direct interaction with the end-user's queries. 

Also the search system acts as the backbone, fetching relevant information upon which the QA system builds its answers.

### Evaluating the Search System 

The aim here is to enhance our focus on key metrics such as precision and recall at various levels (K).

For instances where the dataset might be limited, turning to synthetic data is a practical approach
- It involves selecting random text chunks or documents and then prompting a language model to generate questions that these texts could answer.
- And crucial for verifying the search system's ability to accurately identify and retrieve the text chunks responsible for generating these questions.

```python
def test():

    text_chunk = sample_text_chunk()

    questions = ask_ai(f"generate questions that could be ansered by {text_chunk.text}")

    for question in questions:

        search_results = search(question)



    return {

        "recall@5": (1 if text_chunk in search_results[:5] else 0),

        ...

    }

average_recall = sum(test() for _ in range(n)) / n
```

### Evaluating the Answering System

This is a lot trickier, but often times people will use a framework like, I guess, to evaluate the questions. 

Here I recommend spending some more time building out a data set that actually has answers.

```python
def test():
    text_chunk = sample_text_chunks(n=...)
    question, answer = ask_ai(f"generate questions and answers for {text_chunk.text}")
    ai_answer = rag_app(question)
    return ask_ai(f"for the question {question} is the answer {ai_answer} correct given that {answer} is the correct answer?")
```

### Evaluating the Answering System: Feedback

It's also good to build in **feedback mechanisms** in order to get better scores. I recommend building a *thumbs up, thumbs down* rating system rather than a five star rating system. 


#### The purpose of synethic data

The purpose of synthetic data is to help you quickly get some metrics out.
- It will help you build out this evaluation pipeline in hopes that as you get more users and more real questions, 
- you'll be able to understand where we're performing well and where we're performing poorly using the suite of tests that we have. 
- Precision, recall, mean ranking scores, etc.

## Level 5: Understanding Short comings

At this point you should be able to have a data set that is extremely diverse using both 
- synthetic data
- production data. 

We should also have a suite of scores that we can use to evaluate the quality of our answers.

org_id	query	rewritten	answer	recall@k	precision@k	mean ranking score	reranker score	user feedback	citations	sources	...
org123	...	...	...	...	...	...	...	...	...	...	...


Now we can do a bunch of different things to understand how we're doing by doing exploratory data analysis. 
- We can look at the mean ranking score and reranker score and see if there are any patterns.
- We can look at the citations and see if there are any patterns. 
- We can look at the user feedback and see if there are any patterns. 
- We can look at the sources and see if there are any patterns. 
- We can look at the queries and see if there are any patterns.


### Clustering Queries

We can use clustering to understand if there are any patterns in the queries. 
- We can use clustering to understand if there are any patterns in the citations. 
- We can use clustering to understand if there are any patterns in the sources. 
- We can use clustering to understand if there are any patterns in the user feedback.


I find that there's usually two different kinds of clutches that we detect.
- Topics
- Capabilities

**Topics** are captured by the *nature of the text chunks* and the queries. **Capabilities** are captured by the n*ature of the sources* or additional metadata that we have.

Capabilites could be more like:
- Questions that ask about document metadata "who modified the file last"
- Questions that require summarzation of a document "what are the main points of this document"
- Questions that required timeline information "what happened in the last 3 months"
- Questions that compare and contrast "what are the differences between these two documents"
