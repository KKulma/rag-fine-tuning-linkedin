People tend to treat chat-based LLM systems as a combination of a search engine and a knowledgeable and skilled assistant. They ask the LLM to answer questions or ask you to perform some task and often combine the two. 

However, the LLM isn't a knowledge lookup system, it's a **language transformer**. It doesn't actually know anything, it just has this complex enough map of our language to be able to auto complete most sentences in a way that is mostly correct, as long as the pattern of the information was readily available and prevalent in its training data.


### Embeddings

First, we take each recipe from a recipe book and place it in its own field in a database. Next, we find a two-dimensional embeddings model where each dimension represents some parameter. Then we give each recipe to the embeddings model and it places the words in the two-dimensional space. This produces a different map of points for each recipe with the key words featured. The embeddings model puts more similar words, like carrot and squash, closer together and dissimilar words, like squash and kohlrabi, further apart. 

Finally, lines called vectors are drawn between these points to represent the connections between the words in the text. This mathematical representation of the points and vectors now becomes the embeddings for each individual recipe. 

And we add these back into the database, so we have both the original text and its embeddings in one row. Now we can prompt an AI system with a query like, "How do I make a vegetable cake?" And the AI system will create embeddings from this question using the same embeddings model, and we get a new vector representing the query, in this case, the line between vegetable and cake. We can then use basic math to compare the query embedding against each of the recipe embeddings and we quickly see that some of the recipes are a close match while others are not. 

Finally, the AI system returns the original recipe text from the closest matched embeddings as context, and we are back in the RAG flow.


### Knowledge graphs

Our language is semantically, syntactically, and lexically ambiguous. For example, the same word can mean different things in different contexts. Let's say we have one embedding for use, tomato, paste, and another embedding for use, tooth, paste. These embeddings look nothing alike, but there are two common words in there, use and paste. Now let's say the user puts in a very vague prompt, "What paste do I use to make pasta?" Without further context, the embedding system will zero in on the words use and paste and find any embeddings that match. In return, we get tomato paste and toothpaste. I can think of exactly zero scenarios where both of these responses are correct, but the AI has neither knowledge nor understanding of the data or the query, so it returns both entities and the RAG-based completion becomes some nonsense about how to use toothpaste to add peppermint flavor to your pasta dish or something. 

The problem here is that **while an embedding system can help us identify close matches in language, it has no understanding of the semantic meaning of the connections.** Both toothpaste and tomato paste are pastes, but they have very different uses. 

To address this problem, we can lean on an ancient AI technology called knowledge graphs. Originally invented for search, knowledge graphs add direction and semantic meaning to the vectors connecting the dots in an embeddings map. So rather than having a vector that just says, "This word and this word are connected," the graph says maybe, "This word is part of this word or is used in this context." Remapping our examples using knowledge graphs, we add semantic context by connecting, for example, tomato and paste to cooking with an "is used in" graph and connecting tooth and paste to hygiene using a "is part of" graph. Now, when the system creates a knowledge graph of the query, "What paste do I use to make pasta?" it contains the added graph capturing the context "is used in cooking," and we get only tomato paste as a relevant match.

Now seeing this, you may rightfully conclude that we should just use knowledge graphs for RAG all the time, and in practical reality, that's not always possible. In real-world environments, AI systems using RAG draw from a variety of sources, including traditional databases, APIs, embeddings, and advanced knowledge graph vectors. The best option for a particular use case comes down to many variables, including what type of data it is, how much work is involved in transforming the data into, for example, vector embeddings, how often the data updates, et cetera. The typical recommendation is to start with the most basic option, a straight-up database with an API, and explore more advanced options if the results are not satisfactory.


### Fine-tuning

Fine-tuning is when you provide a huge list of training examples, a completed exchange, including a system message, a prompt, and a response, to a foundation model to train it to respond to specific types of queries in specific ways. When you have a fine-tuned model, you don't need to provide a complex system message or request a specific behavior every time. Instead, the system has been literally fine-tuned to create responses to fit the patterns in the training examples.

Fine-tuning is used to 
- conform chatbot responses to a specific tone, or
- include predefined information like standard contact support for more information phrases in every response, and 
- to perform specific actions like provide a document or database reference in every response.


The challenge with fine-tuning in LLM is you need to create a lot of training data for the fine-tuning to be successful, and then run all of those examples multiple times for the model to conform. And even then, the fine-tuned model doesn't always behave as you intended, so the process involves a lot of trial and testing and error before you get to usable state. That said, for a situation where consistency and responses matters at scale, a fine-tuned model is a good avenue to explore because the fine-tuning reduces the variance in response and gives you direct control over the LLM behavior in a way that foundation models simply do not.


### RAFT

RAFT - Retrieval Augmented Fine-Tuning

RAFT is an answer to the challenge of creating efficient LLM based AI systems that are grounded in specialized domain data like internal enterprise documents. The idea is to first use RAG to fine-tune the LLM on the domain=specific data, so it recognizes and matches the patterns in the data, and then use the resulting raft model in conjunction with RAG to produce better answers that are not only grounded in the domain data, but also conforms to its patterns. 


#### RAFT step-by-step

 1. create or generate a set of domain-specific questions matching the types of user queries the system will handle.
 
 2. use RAG to retrieve two types of domain documents, oracle docs containing the answer to the question and distractor docs containing irrelevant information. 
 
 3. combine the original question with the associated Oracle docs to create a chain of thought answer that breaks down not only the answer, but a detailed reasoning process. 
 
 4. run the fine-tuning process using three types of training examples, question, oracle data, and correct chain of thought answers. Question, distractor data, and correct chain of thought answers. And question, a mix of data, and correct chain of thought answers. 
 
 When a mix of these three training examples are used in the fine-tuning process, the model imprints a pattern to use relevant oracle data, ignore irrelevant distractor data, and discern what data is relevant for any question.