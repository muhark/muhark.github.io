---
layout: post
title: "Implementing Word2Vec in PyTorch"
date: 2021-10-21 00:00:00 +0000
categories: [python, ml, nlp]
---

_Note_:

Apologies for the protracted absence, I definitely over-estimated my ability to maintain this blog while my doctorate got very busy. The good news is that I've been spending the time working on Docker and PyTorch, so I have lots of new things to write about!

## Preface: On Embeddings

Eight years since [Mikolov et al](https://arxiv.org/abs/1301.3781) first proposed a novel architecture for computing vector representation of words in 2013, I think we can safely say that the "word embedding" approach to operationalising text data is entering the political science text-as-data methods mainstream.

The utility of embedding methods is linked directly to the original challenges motivating text-as-data methods. By representing natural language numerically, embedding methods offer the possibility to leverage a broad range of quantitative tools on hitherto unusable sources of data.

The number of political science papers using word embeddings has exploded in just the past three years. It's not my intention to write a full-on literature review in this post, but these three articles (in my humble opinion) are an excellent place to start in understanding the potential applications of word embedding models for computational social science.

- [Garg et al (2018), PNAS](https://www.pnas.org/content/pnas/115/16/E3635.full.pdf) use word embeddings to quantify shifting gender and ethnic stereotypes in the US over the past 100 years.
- [Rodman (2019), Political Analysis](https://static1.squarespace.com/static/5ca7d04ea09a7e68ba44e707/t/5cda219af4e1fc94236bc0cf/1557799325771/Diachronic_Word_Vectors___Political_Analysis_Final_Version.pdf) uses word embeddings to look at the changing meaning and context of salient terms such as "equality" in US news corpora.
- [Rodriguez and Spirling (2021), Journal of Politics](https://www.journals.uchicago.edu/doi/10.1086/715162) evaluate the utility of word embedings for various social science applications.

At a high level, word embeddings represent the individual words (vocabulary) of a collection of texts (corpus) as vectors in a _k_-dimensional space (where _k_ is determined by the researcher–more on this later). These vectors encode information about the relationship between words and their context, and are used for downstream language modelling tasks.

If you're crippled by the same induction-skeptical methods anxiety that I am, then you too may be wondering:

- How does it work?
- Why does it work?
- How do we know whether it's worked?

In order to answer the first two questions for myself, I recently tried implementing my own version of Mikolov et al's `Word2Vec` algorithm in PyTorch. (Note that the state-of-the-art has moved past `Word2Vec` in Natural Language Processing, and I suspect that computational social science will follow suit soon. Nevertheless, implementing papers in code is always a good exercise.)

Also please do let me know if there are mistakes in my implementation; there are a few points I am a bit unsure on, which I'll list at the end of the post.

I'll try to write in a way that is accessible to researchers without familiarity with Python, PyTorch or neural networks. As usual, feel free to reach out to me on Twitter, etc. if you want more clarity on anything in this post.

## Step 1: DataLoader

As with any NLP task (or any data analysis task for that matter), there are two steps:

- Preparing Data (the Loader)
- Processing Data (the Model)

I'll be using the publicly available `tweets_hate_speech_detection` dataset from Huggingface. (TW: some of these tweets are pretty nasty, so you may prefer to choose a different dataset.)

```python
import torch
import datasets

dataset = datasets.load_dataset('tweets_hate_speech_detection')
```

We need a function to split up the raw tweets into lists of tokens. I'll keep the pre-processing very simple for this demonstration, and apply:

1. Lowercase everything
2. Remove all symbols other than `a-z@#`.
3. Split on spaces.
4. Remove stopwords/empty tokens
5. Apply snowball stemmer to remainder

```python
# For simplicity let's remove alphanumeric but keep @, #
import re
from nltk.corpus import stopwords
from nltk.stem.snowball import SnowballStemmer

ss = SnowballStemmer('english')
sw = stopwords.words('english')

def split_tokens(row):                             # STEP
    row['all_tokens'] = [ss.stem(i) for i in       # 5
                     re.split(r" +",               # 3
                     re.sub(r"[^a-z@# ]", "",      # 2
                            row['tweet'].lower())) # 1
                     if (i not in sw) and len(i)]  # 4
    return row

# Determine vocabulary so we can create mapping
dataset = dataset.map(split_tokens)
```

From these we can construct some useful variables for the future. But first let's remove tokens that occure fewer than 10 times to reduce the size of our vocabulary.

- `counts`: Total word counts
- `vocab`: Unique tokens in corpus
- `n_v`: Size of vocab
- `id2tok`/`tok2id`: Move back and forth between tokens and numeric ids

```python
from collections import Counter

counts = Counter([i for s in dataset['train']['all_tokens'] for i in s])
counts = {k:v for k, v in counts.items() if v>10} # Filtering
vocab = list(counts.keys())
n_v = len(vocab)
id2tok = dict(enumerate(vocab))
tok2id = {token: id for id, token in id2tok.items()}

# Now correct tokens
def remove_rare_tokens(row):
    row['tokens'] = [t for t in row['all_tokens'] if t in vocab]
    return row

dataset = dataset.map(remove_rare_tokens)
```


Now finally we need to prepare the "sliding window" used in the Word2Vec algorithm. It's easiest to explain this preparation by example. Suppose the following sentence, _every good boy does fine_, with a window size of 2.

1. `(every, good)`
2. `(every, boy)`
3. `(good, every)`
4. `(good, boy)`
5. `(good, does)`
6. `(boy, every)`

... and, so on. The sentence is converted into pairs of `target`, `context` where context is a list of the tokens within the window.

Let's implement this:

```python
def windowizer(row, wsize=3):
    """
    Windowizer function for Word2Vec. Converts sentence to sliding-window
    pairs.
    """
    doc = row['tokens']
    wsize = 3
    out = []
    for i, wd in enumerate(doc):
        target = tok2id[wd]
        window = [i+j for j in
                  range(-wsize, wsize+1, 1)
                  if (i+j>=0) &
                     (i+j<len(doc)) &
                     (j!=0)]

        out+=[(target, tok2id[doc[w]]) for w in window]
    row['moving_window'] = out
    return row

dataset = dataset.map(windowizer)
```

Now we build this into a PyTorch Dataset class so that we can pass it to a DataLoader class. While this may seem like a lot of redundant work (why can't we just pass the whole dataframe to the model and let it figure it out from there?), taking these steps early on pays off when you need to scale your workflow to work with very large datasets.

From my limited experience, the main advantage of using a DataLoader is the ability to efficiently manage VRAM usage and transfer between disk/RAM/VRAM. It also allows for multiprocessing on the loading/preprocessing side, which can provide enormous speed-ups.

The `Dataset` class requires the following three methods. I'll write with the assumption that you are vaguely aware of the existence of Python classes.

- `__init__`: This gets executed when the class is instantiated. Typically, here is where you define attributes, such as an underlying data object or a pre-processing step that you don't want to execute on-the-fly.
- `__len__`: This should return the length of the dataset. I assume it's important for knowing how much memory to allocate.
- `__getitem__`: Given an index, return the element of the dataset corresponding to that index.

In this tutorial, I've applied each of the pre-processing steps to the dataset using the `dataset.map` function. We could also put this into the `__init__` method of the following.

The processing that I am doing here is:

- Building a single tensor to hold all (word, context-word) pairs, which we'll randomly sample from.
- Returning the (word, context-word) pair.

Also note that this is not the most RAM-efficient way of implementing this.

```python
from torch.utils.data import Dataset, DataLoader

class Word2VecDataset(Dataset):
    """
    Takes a HuggingFace dataset as an input, to be used for a Word2Vec dataloader.
    """
    def __init__(self, dataset, vocab_size, wsize=3):
        self.dataset = dataset
        self.vocab_size = vocab_size
        self.data = [i for s in dataset['moving_window'] for i in s]

    def __len__(self):
        return len(self.data)

    def __getitem__(self, idx):
        return self.data[idx]


```

And now we wrap the dataset with a DataLoader. Note that at this point I'm defining two "global" variables (in all caps): `BATCH_SIZE` and `N_LOADER_PROCS`.

`BATCH_SIZE` is the number of observations returned with each call. Much of the speed-ups from GPU processing come from massive batched matrix computations. When choosing batch size, remember that it's generally at trade-off between VRAM usage and speed, except for when the dataloader itself is the bottleneck. To speed up the dataloader, we can pass an argument to `num_workers` to enable parallelisation on the data preparation and loading.


```python
BATCH_SIZE = 2**14
N_LOADER_PROCS = 10

dataloader = {}
for key in dataset.keys():
    dataloader = {key: DataLoader(Word2VecDataset(
                                    dataset[key], vocab_size=n_v),
                                  batch_size=BATCH_SIZE,
                                  shuffle=True,
                                  num_workers=N_LOADER_PROCS)}

```

## Step 2: Building the Network

Now that we've defined our loader, we can define our neural network. The way we build neural networks in PyTorch might seem strange at first, but it quickly becomes very natural. We use the same Python class structure to instantiate the "building blocks" in the `__init__`, and then define the "forward pass" (i.e. the path from the input data to the output) in the `forward` method.

A quick refresher on the `Word2Vec` architecture as defined by Mikolov et al:

- Three layers: input, hidden and output.
- Input and output are the size of the vocabulary. Hidden is smaller.
- Fully connected with linear activations.

There are two variants of this architecture:

- `CBOW` (continuous bag-of-words): context word is input, center word is output.
- `Skip-gram`: center word is input, context word is output.

Since these are mirror images of each other (the difference is the order of input/output), we can define both of them with the same architecture (I think).

### Aside: manually implementing one-hot encoding

Before we begin, however, a quick aside about how to generating one-hot encodings. Here's how we would do it "manually":

```python
from torch import nn

size = 10
input = 3

def one_hot_encode(input, size):
    vec = torch.zeros(size).float()
    vec[input] = 1.0
    return vec

ohe = one_hot_encode(input, size)
linear_layer = nn.Linear(size, 1, bias=False)

# Set edge weights from 0 to 9 for easy reference
with torch.no_grad():
    linear_layer.weight = nn.Parameter(
        torch.arange(10, dtype=torch.float).reshape(linear_layer.weight.shape))

print(linear_layer.weight)
print(linear_layer(ohe))
```

    Parameter containing:
    tensor([[0., 1., 2., 3., 4., 5., 6., 7., 8., 9.]], requires_grad=True)
    tensor([3.], grad_fn=<SqueezeBackward3>)

What's going on here?

1. First, we create a tensor of zeros equal in size to the vocabulary, and then assign `1` to the value corresponding to our feature.
2. We instantiate a linear layer with no bias, which is essentially a 10x1 tensor of edge weights.
3. I overwrite the randomly initialised weights with the values 0-9. We wrap this in `torch.no_grad()` to disable gradient tracking; in short, operations on PyTorch tensors with gradient tracking enabled are stored in order to differentiate the loss w.r.t. every parameter in the model. Because here I am manually setting the parameters, I don't actually want this action to be stored and considered when making a future backprop calculation.
3. When we pass our one-hot encoded vector, we retrieve the weight corresponding to the input id.

PyTorch implements this more efficiently using their `nn.Embedding` object, which takes the input index as an input and returns edge weight corresponding to that index.

Here's the equivalent code.

```python
embedding_layer = nn.Embedding(size, 1)

with torch.no_grad():
    embedding_layer.weight = nn.Parameter(
        torch.arange(10, dtype=torch.float
        ).reshape(embedding_layer.weight.shape))

print(embedding_layer.weight)
print(embedding_layer(torch.tensor(input)))

```

    Parameter containing:
    tensor([[0.],
            [1.],
            [2.],
            [3.],
            [4.],
            [5.],
            [6.],
            [7.],
            [8.],
            [9.]], requires_grad=True)
    tensor([3.], grad_fn=<EmbeddingBackward>)

With that out of the way, here's how I implement the Word2Vec model.

```python
class Word2Vec(nn.Module):
    def __init__(self, vocab_size, embedding_size):
        super().__init__()
        self.embed = nn.Embedding(vocab_size, embedding_size)
        self.expand = nn.Linear(embedding_size, vocab_size, bias=False)

    def forward(self, input):
        # Encode input to lower-dimensional representation
        hidden = self.embed(input)
        # Expand hidden layer to predictions
        logits = self.expand(hidden)
        return logits
```

Going through it line-by-line:

- `class [...]`: we define our neural network as a child class of `nn.Module`, meaning we inherit all the methods of the parent class `nn.Module`. Note also that we are not building the network here, but a blueprint to instantiate the network.
- `super().__init__()`: instantiates all of the init methods of the parent class.
- `self.embed = [...]`: an embedding layer to convert the input (the index of the center/context token) into the the one-hot encoding, and then retrieve the weights corresponding to these indices in the lower-dimensional hidden layer.
- `self.expand = [...]`: a linear layer to predict the probability of a center/context word given the hidden layer. We disable bias (the intercept) because we rescale our predictions anyways.
- `forward()`: defining the forward pass.
- `hidden = [...]`: making the pass from the input layer to the smaller hidden layer.
- `logits = [...]`: re-expanding the hidden layer to make predictions. These raw predictions need to be re-scaled using softmax, but we skip this step here as PyTorch implements the relevant steps under Cross Entropy loss.

## Step 3: Training

Unlike the ML or statistical models that you may be used to, when we train a neural network there's not a clear point at which the model has "finished" training (i.e. converged).

Training in the context of neural networks means repeatedly making predictions using the observations in the dataset and then adjusting the parameters to correct for the error in the predictions. Because we don't want the network to perfectly learn the most recent prediction while forgetting all other predictions, we usually give it a "learning rate", which is some penalty on the loss adjustment to prevent fitting only to the most recent observation.

The longer we train the network, the more perfectly it will learn the training data, but often this comes with the risk of overfitting and failing to generalise to unseen data. However, given that with Word2Vec our goal is not to infer unseen data, but to describe "seen" data, I'm not sure what the implications of over-fitting are (maybe between "global" meanings and corpus-specific meanings?)

We use a for-loop to do our training. Here's the set-up:

```python
# Instantiate the model
EMBED_SIZE = 100 # Quite small, just for the tutorial
model = Word2Vec(n_v, EMBED_SIZE)

# Relevant if you have a GPU:
device = torch.device('cuda') if torch.cuda.is_available() else torch.device('cpu')
model.to(device)

# Define training parameters
LR = 3e-4
EPOCHS = 10
loss_fn = nn.CrossEntropyLoss()
optimizer = torch.optim.AdamW(model.parameters(), lr=LR)
```

- `cbow = [...]`: we instantiate the model.
- `device = [...]`: torch tensors can be moved to either CPU/RAM or GPU/VRAM. If you have a CUDA-enabled GPU (i.e. Nvidia), you can run your computations considerably faster. Google Colab has these available to use for free. We move models and tensors using their `.to` method.
- `LR`: learning rate. A very long and detailed topic, but in short lower learning rate reduces overfitting but increases training time.
- `EPOCHS`: number of times to pass the full training data through the model.
- `loss_fn`: in short, the appropriate loss function for making categorical predictions (but do look up why this is the case).
- `optimizer`: the algorithm on how to update the parameters as a function of loss. A very simple optimizer would be Stochastic Gradient Loss, which travels down the gradient towards an optimum. `AdamW` works quite well, but I haven't looked into how it works.

To begin, let's run a ten loops in the training loop.

```python
from tqdm import tqdm  # For progress bars

progress_bar = tqdm(range(EPOCHS * len(dataloader['train'])))
running_loss = []
for epoch in range(EPOCHS):
    epoch_loss = 0
    for center, context in dataloader['train']:
        center, context = center.to(device), context.to(device)
        optimizer.zero_grad()
        logits = model(input=context)
        loss = loss_fn(logits, center)
        epoch_loss += loss.item()
        loss.backward()
        optimizer.step()
        progress_bar.update(1)
    epoch_loss /= len(dataloader['train'])
    running_loss.append(epoch_loss)
```

Let's plot the running loss:

```python
import matplotlib.pyplot as plt
plt.plot(running_loss)
```

    <output truncated>

As we continue the training, the marginal increase in accuracy will decrease. Remember, though, that accuracy is not the main goal with embedding models. Let's check out our embeddings.

Remember that the embeddings are the edge weights between the hidden layer and the output. Let's access and inspect the ones corresponding to "freedom", "mom", "school" and "#power".


```python
wordvecs = model.expand.weight.cpu().detach().numpy()
tokens = ['good', 'father', 'school', 'hate']
```

Now let's get the closest vectors (by various metrics):

```python
from scipy.spatial import distance
import numpy as np

def get_distance_matrix(wordvecs, metric):
    dist_matrix = distance.squareform(distance.pdist(wordvecs, metric))
    return dist_matrix

def get_k_similar_words(word, dist_matrix, k=10):
    idx = tok2id[word]
    dists = dist_matrix[idx]
    ind = np.argpartition(dists, k)[:k+1]
    ind = ind[np.argsort(dists[ind])][1:]
    out = [(i, id2tok[i], dists[i]) for i in ind]
    return out

dmat = get_distance_matrix(wordvecs, 'cosine')
for word in tokens:
    print(word, [t[1] for t in get_k_similar_words(word, dmat)], "\n")
```

good ['tomorrow', 'even', 'great', 'one', 'got', 'see', 'work', 'today', 'love', '@user']

father ['day', 'dad', '#fathersday', 'happi', '#dad', 'us', 'fathersday', 'thank', 'wish', '#smile']

school ['favorit', 'first', 'amp', 'islam', 'trump', 'today', 'man', 'last', 'call', 'still']

hate ['@user', 'amp', 'reason', 'yet', 'make', 'would', 'someon', 'final', 'way', 'say']


Now let's train the model for another 90 epochs and see how these change:


```python
EPOCHS = 90
progress_bar = tqdm(range(EPOCHS * len(dataloader['train'])))
for epoch in range(EPOCHS):
    epoch_loss = 0
    for center, context in dataloader['train']:
        center, context = center.to(device), context.to(device)
        optimizer.zero_grad()
        logits = model(input=context)
        loss = loss_fn(logits, center)
        epoch_loss += loss.item()
        loss.backward()
        optimizer.step()
        progress_bar.update(1)
    epoch_loss /= len(dataloader['train'])
    running_loss.append(epoch_loss)
```

Extracting the new word vectors:

```python
wordvecs_100_epochs = model.expand.weight.cpu().detach().numpy()
dmat_100_epochs = get_distance_matrix(wordvecs_100_epochs, 'cosine')
for word in tokens:
    print(word, [t[1] for t in get_k_similar_words(word, dmat_100_epochs)], "\n")
```

    good ['great', 'happi', '@user', 'day', 'today', 'amp', 'love', 'make', 'even', 'im']

    father ['dad', 'day', '#fathersday', 'happi', 'great', 'love', 'god', 'one', 'hope', 'enjoy']

    school ['first', 'week', 'year', 'month', 'back', 'see', 'next', 'last', 'tomorrow', 'one']

    hate ['peopl', 'would', 'kill', 'mani', 'say', 'much', 'man', 'dont', 'someon', 'sad']

These vectors (unfortunately) seem like a better ordering of most similar words within the context of co-occurrence of the corpus. (Deeper analysis in follow-up post, I promise!)

## Wrap-up

I'll leave it off for here since it's been 3 months since I wrote a post, but there's some pretty neat stuff you can do with tensorboard to track, explore and visualise embeddings. I also want to write a bit more about what's going on, and what to make of `Word2Vec` embeddings now that we have seen how to generate them.


As a last note, to save your embeddings and model, you can use `torch.save(model.state_dict())` to save the model for later usage.

```python
torch.save(model.state_dict(), 'word2vec-twitter_hate-100epochs.checkpoint')
```

Then to load it you instantiate a model and load in the pre-trained weights:

```python
model2 = Word2Vec(n_v, EMBED_SIZE)
model2.load(torch.load('word2vec-twitter_hate-100epochs.checkpoint'))
```

That's all for now!
