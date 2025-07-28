# TP-NLP
The delivery consists of three important documents:

-'pruebas.ipynb': This notebook contains tests performed on different models and parameters based on the given problem statement to arrive at the best possible model, and it also serves as the training notebook.

-'opensearch_data_model.py': This file contains the data structure for the vector database.

-'daily_predictions.ipynb': This notebook takes the trained model and the created database to perform the daily task of grouping documents and searching for whatever is needed in the vector database.

# Tests
This stage must be repeated every time the model needs to be retrained to update the topics. The variable used to select the winning model is the silhouette_score, referred to later as the metric.

The first step is to import the documents. For the model input, the titles concatenated with the article body were used as a single value called 'title_text'.

First Model:
As a starting point, a test was conducted using BERTopic in Spanish as the base model, which yielded a metric of 0.198.

Second Model:
In this stage, I decided to add a multilingual SentenceEncoder for processing the 'title_text', a CountVectorizer with Spanish stopwords (including the possibility of bigrams for topic grouping), and a ClassTfidfTransformer for topic clustering. The metric improved to 0.206.

Third Model:
In addition to the components from the second model, I incorporated UMAP for dimensionality reduction. The hyperparameters were chosen based on the best metric obtained after several iterations. The metric improved to 0.212.

Fourth Model:
An HDBSCAN was added for cluster grouping, achieving a metric of 0.216, making this the winning model.

Summarizing Stage
This stage corresponds to the model that assigns a name to each topic. The selection was somewhat more subjective, as the goal was to summarize the topic in a few words. I tested a basic summarizer, but the best results came from a pre-trained transformer model: T5ForConditionalGeneration (specifically 'google-t5/t5-large').

Saving to the Vector Database
The next step involves saving the necessary data into the vector database for future production queries.

Unlike the approach taken in class, I opted to store entities and keywords as lists to facilitate matching new documents (along with their keywords and entities) to their assigned topics during production.

Additionally, I saved the winning topic model as a Joblib dump for use in future inferences.

# Predictions
Whenever new news articles are received, they are loaded into memory, the topic model saved in Joblib is loaded, and a topic inference is performed using the title concatenated with the text as input. This returns the most likely topic and its probability. After reviewing some predictions, I decided to set a minimum probability threshold of 0.2 for assigning a topic (other than the "junk" topic), as some articles with low probability did not truly belong to the stored topic.

Another difference from the approach discussed in class is that my vector database search is based on topics rather than embeddings. I found this to be a more logical and straightforward alternative since the saved model can directly infer which topic an article belongs to (and its probability) instead of relying on cosine similarity for the same task.

Once the topic is identified in the database, the keywords and entities present in both the article and the stored topic are retrieved.

The final step is saving the model outputs (ID, matched keywords, and entities) to a file.

   
