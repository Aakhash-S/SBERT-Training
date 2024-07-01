import torch
from sentence_transformers import SentenceTransformer

# Load the saved model
model = SentenceTransformer('output/training_stsbenchmark_distilbert-base-uncased-2024-06-28_23-33-56/final')

# Get input from the user for a corpus and a keyword
corpus = [
    "A man is playing a large flute.",
    "A man is playing a flute.",
    "A man is spreading shreded cheese on a pizza.",
    "Three men are playing chess.",
    "A man is playing the cello.",
    "Severe Gales As Storm Clodagh Hits Britain",
    "Dozens of Egyptians hostages taken by Libyan terrorists",
    "President heading to Bahrain",
    "China, India vow to further bilateral ties",
    "Putin spokesman: Doping charges appear unfounded"
]

keyword = "man playing instrument"

# Encode the corpus and keyword using the model
corpus_embeddings = model.encode(corpus)
keyword_embedding = model.encode(keyword)

# Calculate the cosine similarity between the keyword and each sentence in the corpus
cosine_similarities = torch.nn.CosineSimilarity(dim=1)(corpus_embeddings, keyword_embedding.unsqueeze(0))

# Get the indices of the sentences with the highest cosine similarity
top_indices = torch.topk(cosine_similarities, k=3, dim=0).indices

# Print the most relevant sentences
for i in top_indices:
    print(f"Sentence {i.item()}: {corpus[i.item()]}")
