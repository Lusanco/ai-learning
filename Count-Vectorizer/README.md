# Count Vectorizer

## Code w/Study Notes

```python
# Count Vectorizer

# imports
import numpy as np
import pandas as pd
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.naive_bayes import MultinomialNB # classifier of choice for this notebook
from sklearn.model_selection import train_test_split

import nltk
from nltk import word_tokenize
from nltk.stem import WordNetLemmatizer, PorterStemmer
from nltk.corpus import wordnet

# nltk downloads
nltk.download("wordnet")
nltk.download('punkt')
nltk.download('averaged_perceptron_tagger')

# Download data set (samples of BBC News) to classify documents
# https://www.kaggle.com/shivamkushwaha/bbc-full-text-document-classification
# !wget -nc https://lazyprogrammer.me/course_files/nlp/bbc_text_cls.csv

# Reading data set (previously downloaded .csv document)
df = pd.read_csv('bbc_text_cls.csv')

# See what the data looks like
df.head()
# output of df.head()
#                                               text	labels
# 0	Ad sales boost Time Warner profit\n\nQuarterly...	business
# 1	Dollar gains on Greenspan speech\n\nThe dollar...	business
# 2	Yukos unit buyer faces loan claim\n\nThe owner...	business
# 3	High fuel prices hit BA's profits\n\nBritish A...	business
# 4	Pernod takeover talk lifts Domecq\n\nShares in...	business

# Assign the two columns to separate variables
inputs = df['text']
labels = df['labels']

# Plot a histogram of our labels to see if we have imbalanced classes (if any class is over or under represented).
labels.hist(figsize=(10, 5))

# call train_test_split (before the use of CountVectorizer)
inputs_train, inputs_test, Ytrain, Ytest = train_test_split(
    inputs, labels, random_state=123)

# instantiate CountVectorizer object
vectorizer = CountVectorizer()

# call fit_transform on the training inputs
Xtrain = vectorizer.fit_transform(inputs_train)
# followed by calling transform on the test inputs
Xtest = vectorizer.transform(inputs_test)

# print Xtrain (sparce matrix)
print(Xtrain)
# Output of Xtrain
# <1668x26287 sparse matrix of type '<class 'numpy.int64'>'
# 	with 337411 stored elements in Compressed Sparse Row format>

# To see how sparce this matrix is, we're going to compute the percentage of values in extreme, which are non-zero.

# returns a Boolean matrix with true, where the value is non-zero and false otherwise.
(Xtrain != 0).sum() # numerator (count of non-zero values)
# Output is 337411

# what percentage of values are non-zero?

# denominator vv
# Xtrain.shape returns a tuple containing the number of rows and the number of columns
# np.prod returns elements inside Xtrain: rows x cols
# denominator ^^

# this expression returns the number of non-zero elements divided by the total number of elements
(Xtrain != 0).sum() / np.prod(Xtrain.shape)
# Output is 0.007695239935415004

# machine learning steps

# first step: creating an instance of our model
model = MultinomialNB()

# second step: fit the model on the train sets
model.fit(Xtrain, Ytrain)

# third step: check the performance of the model on both the train and the test sets
print("train score:", model.score(Xtrain, Ytrain))
print("test score:", model.score(Xtest, Ytest))
# train score: 0.9922062350119905
# test score: 0.9712746858168761

# **(score function returns the accuracy)**

# with stopwords
vectorizer = CountVectorizer(stop_words='english')
Xtrain = vectorizer.fit_transform(inputs_train)
Xtest = vectorizer.transform(inputs_test)
model = MultinomialNB()
model.fit(Xtrain, Ytrain)
print("train score:", model.score(Xtrain, Ytrain))
print("test score:", model.score(Xtest, Ytest))
# train score: 0.9928057553956835
# test score: 0.9766606822262118

# lemmatization tag function
def get_wordnet_pos(treebank_tag):
  if treebank_tag.startswith('J'):
    return wordnet.ADJ
  elif treebank_tag.startswith('V'):
    return wordnet.VERB
  elif treebank_tag.startswith('N'):
    return wordnet.NOUN
  elif treebank_tag.startswith('R'):
    return wordnet.ADV
  else:
    return wordnet.NOUN

# this class will do all the work of tokenizing and limiting each document
class LemmaTokenizer:
  def __init__(self):

    # instantiate WordNetLemmatizer object
    self.wnl = WordNetLemmatizer()

  def __call__(self, doc):

    # call the word_tokenize function from nltk
    # this will convert our documents into tokens
    tokens = word_tokenize(doc)

    # obtain the parts of speech tags by calling nltk.pos_tag
    # returns a list containing tuples consisting of word in the document with corresponding tag
    words_and_tags = nltk.pos_tag(tokens)

    # loop through each word/tag pair and call the lemmatize function
    # output is a list containing each lemmatized word in the input document
    return [self.wnl.lemmatize(word, pos=get_wordnet_pos(tag)) \
            for word, tag in words_and_tags]

# with lemmatization (the slowest process yet most accurate is with lemmatization)
vectorizer = CountVectorizer(tokenizer=LemmaTokenizer(), token_pattern=None)
Xtrain = vectorizer.fit_transform(inputs_train)
Xtest = vectorizer.transform(inputs_test)
model = MultinomialNB()
model.fit(Xtrain, Ytrain)
print("train score:", model.score(Xtrain, Ytrain))
print("test score:", model.score(Xtest, Ytest))
# train score: 0.9922062350119905
# test score: 0.9676840215439856

# stemming tokenizer class
class StemTokenizer:
  def __init__(self):

    # Instantiate PorterStemmer object
    self.porter = PorterStemmer()

  def __call__(self, doc):

    # call word_tokenize to tokenize document
    tokens = word_tokenize(doc)

    # call porter.stem for each token
    # output is a list of stemmed tokens
    return [self.porter.stem(t) for t in tokens]

# with stemming
vectorizer = CountVectorizer(tokenizer=StemTokenizer(), token_pattern=None)
Xtrain = vectorizer.fit_transform(inputs_train)
Xtest = vectorizer.transform(inputs_test)
model = MultinomialNB()
model.fit(Xtrain, Ytrain)
print("train score:", model.score(Xtrain, Ytrain))
print("test score:", model.score(Xtest, Ytest))
# train score: 0.9892086330935251
# test score: 0.9694793536804309

# string split function
def simple_tokenizer(s):
  return s.split()

# string split tokenizer
vectorizer = CountVectorizer(tokenizer=simple_tokenizer, token_pattern=None)
Xtrain = vectorizer.fit_transform(inputs_train)
Xtest = vectorizer.transform(inputs_test)
model = MultinomialNB()
model.fit(Xtrain, Ytrain)
print("train score:", model.score(Xtrain, Ytrain))
print("test score:", model.score(Xtest, Ytest))
# train score: 0.9952038369304557
# test score: 0.9712746858168761

# What is the vector dimensionality in each case?
# Compare them and consider why they are larger / smaller
```
