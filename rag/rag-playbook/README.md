# The RAG Playbook

- Author: Jason Liu
- Link : https://jxnl.github.io/blog/writing/2024/08/19/rag-flywheel/#1-start-with-synthetic-data
- Miskies App: https://www.miskies.app/miskie/rag-playbook-e8lbnc

## Steps for continually infer and improve RAG systems

1. Initial Implementation
2. Synthetic Data Generation
3. Fast Evaluations
4. Real-World Data Collection
5. Classification and Analysis
6. System Improvements
7. Production Monitoring
8. User Feedback Integration
9. Iteration


### 1. Start with Synthetic Data

Biggest Mistake -> Spending too much time on complex generation before understanding if their retrieval even works.  Synthetic data is your secret weapon here

- Generate synthetic questions for each chunk of text in your database.
- Use these to test your retrieval system and calculate precision and recall scores

This gives you a baseline to work from and helps identify low-hanging fruit for improvement.

Why is this so powerful?
* It helps you select the right embedding models and methods
* Enables lightning-fast evaluations (milliseconds vs. seconds per question)
* Allows rapid iteration and testing of ideas
* Can be done before you have any real user data
* Forces clarity on product goals and non-goals

> When you have concrete metrics like precision and recall, your stand-ups become far more productive. Instead of vague progress reports, you can say things like: "We improved recall by 5% by tweaking our chunking strategy." This focuses the team and gives leadership clear indicators of progress.

### 2. Focus of Leadning Metrics 

- Stop obsessing over lagging metrics like overall application quality.
- Instead, focus on leading metrics that predict improvements and are easier to act on.

For example:
- Number of retrieval experiments run per week
- Precision/recall improvements on synthetic data
- Time to run evaluation suite

### 3. Fast, Unit Test-Like Evaluations

Before you even think about complex generation, 
- nail your retrieval with fast, 
- unit test-style evaluations:
    - Take a search query
    - Find a list of relevant text chunks
    - Check if the desired chunk is in the results

This process should be blazing fast, allowing you to rapidly test changes in how you represent and index your text chunks. It's also great for verifying data recovery across different content types (tables, images, etc.)

> Crawl Before You Walk - Get your retrieval working first. It's easier to measure, usually the weak link, and sets a strong foundation for everything else.

### 4. Real World Data and clustering

Once you have some real user data, things get interesting. 
- You'll quickly realize that real-world questions are often stranger and more idiosyncratic than your synthetic ones. 
- They may not even have clear answers within your system.


This is where clustering becomes powerful:
- Use unsupervised learning to identify question topics and patterns
- Work with domain experts to refine and label these clusters
- Build few-shot classifiers to generate topic distributions for new questions

Now you can analyze:
- Types and frequency of questions per topic
- Cosine similarity scores within clusters
- Customer satisfaction and feedback per topic

> This segmentation is crucial. Just like Google eventually specialized into Maps, Images, and Shopping, you'll likely need to build targeted solutions for different question types.

### 5. Continuous Improvement Loop

Remember, RAG systems are never "done." Set up a continuous improvement cycle:
- Monitor production data in **real-time**, **classifying** questions by topic
- Identify changes in **question patterns** or new user needs
- Regularly *communicate with customers* to validate quantitative findings
- *Prioritize improvements* based on business impact and user satisfaction
- Run *targeted experiments* to address specific topic or capability gaps
- *Iterate and refine* your synthetic data generation based on new insights

> Detecting Concept Drift - One powerful technique is to include an "Other" category in your topic classification. Monitor the percentage of "Other" questions over time. If it starts growing unexpectedly, it's a strong signal that user behavior is shifting or you're onboarding customers with different needs.

## Conclusion

This systematic approach does more than just improve your RAG system. It fundamentally changes how you operate:
- Stand-ups become focused on concrete experiments and metrics
- You build intuition for what actually moves the needle
- Product decisions are driven by data, not guesswork
- You can detect and adapt to changing user needs much faster

> Remember, the goal isn't to have a perfect system on day one. 

It's to build a flywheel of continuous improvement that compounds over time. Start simple, measure relentlessly, and iterate based on real-world feedback. That's how you build RAG applications that truly deliver value.
