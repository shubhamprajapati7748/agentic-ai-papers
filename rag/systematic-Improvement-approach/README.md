# Systematically Improving Your RAG

- Link : https://jxnl.github.io/blog/writing/2024/05/22/systematically-improving-your-rag/
- Miskies App: https://www.miskies.app/miskie/systematically-improving--ec47o3

These foundational pieces set the stage for a comprehensive guide on systematically improving RAG applications, offering practical strategies for developers and organizations looking to optimize their systems.

By the end of this post, you'll have a clear understanding of my systematic approach to improving RAG applications for the companies I work with. We'll cover key areas such as:

- Create synthetic questions and answers to quickly evaluate your system's precision and recall
- Make sure to combine full-text search and vector search for optimal retrieval
- Implementing the right user feedback mechanisms to capture specifically what you're interested in studying
- Use clustering to find segments of queries that have issues, broken down into topics and capabilities
- Build specific systems to improve capabilities
- Continuously monitoring, evaluating as real-world data grows.


## Start with Synthetic Data 

Biggest mistake -> Spending too much time on the actual synthesis without actually understanding whether or not the data is being retrieved correctly. 

To avoid this:
- Create synthetic questions for each text chunk in your database
- Use these questions to test your retrieval system
- Calculate precision and recall scores to establish a baseline
- Identify areas for improvement based on the baseline scores

What we should be finding with synthetic data is that **synthetic data** should just be around *97% recall precision*. And synthetic data might just look like something very simple to begin with.

> for every text chunk, synthetically generate a set of questions that this text chunk answers

For those questions, can we retrieve those text chunks? And you might think the answer is always going to be yes
- But I found in practice that when I was doing tests against essays, full text search and embeddings basically performed the same, except full text search was about 10 times faster.

Whereas when I did the same experiment on pulling issues from a repository, it was the case that
- full text search got around 55% recall, 
- and then embedding search got around 65% recall

## Utilize Metadata

Ensuring relevant metadata (e.g., date ranges, file names, ownership) is extracted and searchable is crucial for improving search results.
- Extract relevant metadata from your documents
- Include metadata in your search indexes
- Use query understanding to extract metadata from user queries
- Expand search queries with relevant metadata to improve results


For example, if someone asks, **What is the latest x, y, and z?** Text search will never get that answer. Semantic search will never get that answer.

- You need to perform query understanding to extract date ranges. 
- There will be some prompt engineering that needs to happen. 
- That's the metadata, and being aware that there will be questions that people aren't answering because those filters can never be caught by full text search and semantic search.


And what this looks like in practice is if you ask the question, **what are recent developments in the field**
- the search query is now expanded out to more terms. 
- There's a date range where the language model has reasoned about what recent looks like for the research, and it's also decided that you should only be searching specific sources. 
- If you don't do this, then you may not get trusted sources. 
- You may be unable to figure out what recent means.

You'll need to do some query understanding to extract date ranges and include metadata in your search.


## Use Both Full-Text Search and Vector Search 

Utilize both full-text search and vector search (embeddings) for retrieving relevant documents. 

Ideally, you should use a single database system to avoid synchronization issues.
- Implement both full-text search and vector search
- Test the performance of each method on your specific use case
- Consider using a single database system to store both types of data
- Evaluate the trade-offs between speed and recall for your application

In my experience, full-text search can be faster, but vector search can provide better recall.

What ended up being very complicated was if you have a single knowledge base, maybe that complexity is fine, because you have more configuration of each one.

## Implement Clear User Feedback Mechanisms
 
Implementing clear user feedback systems (e.g., thumbs up/down) is essential for gathering data on your system's performance and identifying areas for improvement.

- Add user feedback mechanisms to your application
- Make sure the copy for these mechanisms clearly describes what you're measuring
- Ask specific questions like "Did we answer the question correctly?" instead of general ones like "How did we do?"
- Use the feedback data to identify areas for improvement and prioritize fixes

I find that it's important to **build out these feedback mechanisms as soon as possible**. And making sure that the copy of these feedback mechanisms explicitly describe what you're worried about.


Sometimes, we'll get a thumbs down even if the answer is correct, but they didn't like the tone. Or the answer was correct, but the latency was too high. Or it took too many hops.

- This means we couldn't actually produce an evaluation dataset just by figuring out what was a thumbs up and a thumbs down. 
- It was a lot of confounding variables. We had to change the copy to just "Did we answer the question correctly? Yes or no." 
- We need to recognize that improvements in tone and improvements in latency will come eventually. 
- But we needed the user feedback to build us that evaluation dataset.

## Cluster and Model Topics
 
Analyze user queries and feedback to 
- identify topic clusters, 
- capabilities, 
- and areas of user dissatisfaction. 
- This will help you prioritize improvements.

Why should we do this? Let me give you an example. 
- I once worked with a company that provided a technical documentation search system. 
- By clustering user queries, we identified two main issues:

    1. **Topic Clusters**: A significant portion of user queries were related to a specific product feature that had recently been updated. However, our system was not retrieving the most up-to-date documentation for this feature, leading to confusion and frustration among users.

    2. **Capability Gaps**: Another cluster of queries revealed that users were frequently asking for troubleshooting steps and error code explanations. While our system could retrieve relevant documentation, it struggled to provide direct, actionable answers to these types of questions.

Based on these insights, we prioritized updating the product feature documentation and implementing a feature to extract step-by-step instructions and error code explanations. These targeted improvements led to higher user satisfaction and reduced support requests.


Look for patterns like:
- Topic clusters: Are users asking about specific topics more than others? This could indicate a need for more content in those areas or better retrieval of existing content.

- Capabilities: Are there types of questions your system categorically cannot answer? This could indicate a need for new features or capabilities, such as direct answer extraction, multi-document summarization, or domain-specific reasoning.

## Continuously Monitor and Experiment

Continuously monitor your system's performance and run experiments to test improvements.

- Set up monitoring and logging to track system performance over time
- Regularly review the data to identify trends and issues
- Design and run experiments to test potential improvements
- Measure the impact of changes on precision, recall, and other relevant metrics
- Implement changes that show significant improvements

This could include tweaking search parameters, adding metadata, or trying different embedding models. Measure the impact on precision and recall to see if the changes are worthwhile.

Once you now have these questions in place, you have your synthetic data set and a bunch of user data with ratings. This is where the real work begins when it comes to systematically improving your RAG.

## Balance Latency and Performance 

Finally, make informed decisions about trade-offs between system latency and search performance based on your specific use case and user requirements.

- Understand the latency and performance requirements for your application
- Measure the impact of different configurations on latency and performance
- Make trade-offs based on what's most important for your users
- Consider different requirements for different use cases (e.g., medical diagnosis vs. general search)

Here, this is where having the synthetic questions that test against will effectively answer that question. Because what we'll do is we'll run the query with and without this parent document retriever, and we will have a recall with and without that feature and the latency improvement of that feature.

