# Stemming and Lemmatization

##  Code w/Study Notes

```python
# Stemming and Lemmatization

# imports
import nltk
from nltk.stem import PorterStemmer
from nltk.stem import WordNetLemmatizer
from nltk.corpus import wordnet

# nlkt downloads
nltk.download("wordnet")
nltk.download('averaged_perceptron_tagger')

# inits for stemmer and lemmatizer
porter = PorterStemmer()
lemmatizer = WordNetLemmatizer()

# The stemmer for walking, walked and walks results as expected with 'walk'
porter.stem("walking")
porter.stem("walked")
porter.stem("walks")

# The stemmer does not have rules for ran. The results are supposed to be 'run' and not 'ran'
porter.stem("ran")

# For running and bosses, stemmer correctly outputs 'run' and 'boss' respectively
porter.stem("running")
porter.stem("bosses")

# By the time you have gotten to this part of the code, you can start to understand a little bit on how the stemmer works. Results for replacement are 'replac', not a real word.
porter.stem("replacement")

# Stemming an entire sentence vv
sentence = "Lemmatization is more sophisticated than stemming".split()
# Stemmer stems one word at a time, hence the need for tokenizing (use of .split()) and stemming through a for loop to stem each word individually.
for token in sentence:
  print(porter.stem(token), end=" ")
# == Result: 'lemmat is more sophist than stem' == #
# == Note: Not all of these are real words...   == #
# Stemming an entire sentence ^^

# Results for unnecesary and berry are 'unnecesari' and 'berri' respectively
porter.stem("unnecessary")
porter.stem("berry")

# Use of Lemmatizer: Notice on how without the telling the lemmatizer what kind of word it is, it defaults to NOUN. Resulting in a wrong output.
# When the lemmatizer has been specified what type of word it is, it returns the expected output from a 'database'. In this case the 'wordnet' import.
lemmatizer.lemmatize("walking") # result: walking
lemmatizer.lemmatize("walking", pos=wordnet.VERB) # result: walk
lemmatizer.lemmatize("going") # result: going
lemmatizer.lemmatize("going", pos=wordnet.VERB) # result: go
lemmatizer.lemmatize("ran", pos=wordnet.VERB) #result: run

# Results comparison between a stemmer and lemmatizer
porter.stem("mice") # results: mice
lemmatizer.lemmatize("mice") # results: mouse

porter.stem("was") # results: wa
lemmatizer.lemmatize("was", pos=wordnet.VERB) # results: be

porter.stem("is") # results: is
lemmatizer.lemmatize("is", pos=wordnet.VERB) # results: be

porter.stem("better") # results: better
lemmatizer.lemmatize("better", pos=wordnet.ADJ) # results: good

# Lemmatize with automatic speech tagging: averaged_perceptron_tagger
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

sentence = "Donald Trump has a devoted following".split()

# Run speech tagger, returns a tuple with the word and corresponding tag.
words_and_tags = nltk.pos_tag(sentence)
# [('Donald', 'NNP'),
# ('Trump', 'NNP'),
# ('has', 'VBZ'),
# ('a', 'DT'),
# ('devoted', 'VBN'),
# ('following', 'NN')]

# Run lemmatizer on each token, we convert the text first using the function above
for word, tag in words_and_tags:
  lemma = lemmatizer.lemmatize(word, pos=get_wordnet_pos(tag))
  print(lemma, end=" ")
# Results 'Donald Trump have a devote following '

# Another example for lemmatization vv
sentence = "The cat was following the bird as it flew by".split()
words_and_tags = nltk.pos_tag(sentence)

for word, tag in words_and_tags:
  lemma = lemmatizer.lemmatize(word, pos=get_wordnet_pos(tag))
  print(lemma, end=" ")
  # == Original: 'The cat was following the bird as it flew by' == #
  # == Result: 'The cat be follow the bird a it fly by '...     == #
# Another example for lemmatization ^^
```
