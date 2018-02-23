---
title: "word-embeddings"
date: 2018-02-23 13:01
---

[TOC]

# Word Embeddings#
词嵌入向量具有很强的泛化和迁移学习能力，即使使用很小的训练集学习出来的词向量，也很容易应用与其他词。
## word represention
## embedding matrix

# Learning Word Embeddings
## word2vec
## negative sampling
## GloVe word vectors

# Application
## sentiment classification
输入一个句子，输出该句子的情感分类。
### 1 - 简单模型
<center>
<img src="/static/images/ML/DL/rnn/Emojifier-V1.png" style="width:900px;height:300px;">
<caption><center> **Figure**: Baseline model (Emojifier-V1).</center></caption>
</center>
不考虑单词顺序，将词向量简单平均后扔进普通神经网络做多分类。该方法简单粗暴，无法考虑句子中词语的反义即顺序关系。比如，they are lacking good taste, good service and good postion. 这句话很容易在这种网络中被认为是好的类。因为出现了很多 good. 由于未考虑时序关系，lacking good 导致。
$$ z^{(i)} = W . avg^{(i)} + b$$
$$ a^{(i)} = softmax(z^{(i)})$$
$$ \mathcal{L}^{(i)} = - \sum_{k = 0}^{n_y - 1} Yoh^{(i)}_k * log(a^{(i)}_k)$$

### 2 - 基于RNN的序列模型
考虑单词顺序对句子的影响。many-to-one 的 rnn 模型。rnn 会考虑前后单词之间的关系，因此 rnn 是更好的选择。
由于不同的输入sentence长度不一样，如果不对输入句子做特殊处理，就无法并行计算所有句子。这里采用的方法就是 padding.
<center>
<img src="/static/images/ML/DL/rnn/emojifier-v2.png" style="width:900px;height:300px;">
<caption><center> **Figure**: Emojifier-V2. A 2-layer LSTM sequence classifier.</center></caption>
</center>



```python
# GRADED FUNCTION: sentences_to_indices

def sentences_to_indices(X, word_to_index, max_len):
    """
    Converts an array of sentences (strings) into an array of indices corresponding to words in the sentences.
    The output shape should be such that it can be given to `Embedding()` (described in Figure 4). 
    
    Arguments:
    X -- array of sentences (strings), of shape (m, 1)
    word_to_index -- a dictionary containing the each word mapped to its index
    max_len -- maximum number of words in a sentence. You can assume every sentence in X is no longer than this. 
    
    Returns:
    X_indices -- array of indices corresponding to words in the sentences from X, of shape (m, max_len)
    """
    
    m = X.shape[0]                                   # number of training examples
    
    ### START CODE HERE ###
    # Initialize X_indices as a numpy matrix of zeros and the correct shape (≈ 1 line)
    X_indices = np.zeros((m, max_len))
    
    for i in range(m):                               # loop over training examples
        
        # Convert the ith training sentence in lower case and split is into words. You should get a list of words.
        sentence_words = [word.lower() for word in X[i].strip().split(' ')]
        
        # Initialize j to 0
        j = 0
        
        # Loop over the words of sentence_words
        for w in sentence_words:
            # Set the (i,j)th entry of X_indices to the index of the correct word.
            X_indices[i, j] = word_to_index[w] if w != '' else 0
            # Increment j to j + 1
            j = j + 1
            
    ### END CODE HERE ###
    
    return X_indices
```

使用keras 实现 word embedding layer
<center>
<img src="/static/images/ML/DL/rnn/embedding.png" style="width:900px;height:300px;">
<caption><center> **Figure**: embedding.</center></caption>
</center>


```python
# GRADED FUNCTION: pretrained_embedding_layer

def pretrained_embedding_layer(word_to_vec_map, word_to_index):
    """
    Creates a Keras Embedding() layer and loads in pre-trained GloVe 50-dimensional vectors.
    
    Arguments:
    word_to_vec_map -- dictionary mapping words to their GloVe vector representation.
    word_to_index -- dictionary mapping from words to their indices in the vocabulary (400,001 words)

    Returns:
    embedding_layer -- pretrained layer Keras instance
    """
    
    vocab_len = len(word_to_index) + 1                  # adding 1 to fit Keras embedding (requirement)
    emb_dim = word_to_vec_map["cucumber"].shape[0]      # define dimensionality of your GloVe word vectors (= 50)
    
    ### START CODE HERE ###
    # Initialize the embedding matrix as a numpy array of zeros of shape (vocab_len, dimensions of word vectors = emb_dim)
    emb_matrix = np.zeros((vocab_len, emb_dim))
    
    # Set each row "index" of the embedding matrix to be the word vector representation of the "index"th word of the vocabulary
    for word, index in word_to_index.items():
        emb_matrix[index, :] = word_to_vec_map[word]

    # Define Keras embedding layer with the correct output/input sizes, make it trainable. Use Embedding(...). Make sure to set trainable=False. 
    embedding_layer = Embedding(vocab_len, emb_dim, trainable=False)
    ### END CODE HERE ###

    # Build the embedding layer, it is required before setting the weights of the embedding layer. Do not modify the "None".
    embedding_layer.build((None,))
    
    # Set the weights of the embedding layer to the embedding matrix. Your layer is now pretrained.
    embedding_layer.set_weights([emb_matrix])
    
    return embedding_layer


# GRADED FUNCTION: Emojify_V2

def Emojify_V2(input_shape, word_to_vec_map, word_to_index):
    """
    Function creating the Emojify-v2 model's graph.
    
    Arguments:
    input_shape -- shape of the input, usually (max_len,)
    word_to_vec_map -- dictionary mapping every word in a vocabulary into its 50-dimensional vector representation
    word_to_index -- dictionary mapping from words to their indices in the vocabulary (400,001 words)

    Returns:
    model -- a model instance in Keras
    """
    
    ### START CODE HERE ###
    # Define sentence_indices as the input of the graph, it should be of shape input_shape and dtype 'int32' (as it contains indices).
    sentence_indices = Input(shape=input_shape, dtype='int32')
    
    # Create the embedding layer pretrained with GloVe Vectors (≈1 line)
    embedding_layer = pretrained_embedding_layer(word_to_vec_map, word_to_index)
    
    # Propagate sentence_indices through your embedding layer, you get back the embeddings
    embeddings = embedding_layer(sentence_indices) 
    
    # Propagate the embeddings through an LSTM layer with 128-dimensional hidden state
    # Be careful, the returned output should be a batch of sequences.
    X = LSTM(128, return_sequences = True)(embeddings)
    # Add dropout with a probability of 0.5
    X = Dropout(0.5)(X)
    # Propagate X trough another LSTM layer with 128-dimensional hidden state
    # Be careful, the returned output should be a single hidden state, not a batch of sequences.
    X = LSTM(128)(X)
    # Add dropout with a probability of 0.5
    X = Dropout(0.5)(X)
    # Propagate X through a Dense layer with softmax activation to get back a batch of 5-dimensional vectors.
    X = Dense(5)(X)
    # Add a softmax activation
    X = Activation('softmax')(X)
    
    # Create Model instance which converts sentence_indices into X.
    model = Model(inputs=sentence_indices, outputs=X)
    
    ### END CODE HERE ###
    
    return model

'''
maxLen = 20
model = Emojify_V2((maxLen,), word_to_vec_map, word_to_index)
model.summary()
model.compile(loss='categorical_crossentropy', optimizer='adam', metrics=['accuracy'])
X_train_indices = sentences_to_indices(X_train, word_to_index, maxLen)
Y_train_oh = convert_to_one_hot(Y_train, C = 5)
model.fit(X_train_indices, Y_train_oh, epochs = 50, batch_size = 32, shuffle=True)
X_test_indices = sentences_to_indices(X_test, word_to_index, max_len = maxLen)
Y_test_oh = convert_to_one_hot(Y_test, C = 5)
loss, acc = model.evaluate(X_test_indices, Y_test_oh)
print()
print("Test accuracy = ", acc)
'''
```




    '\nmaxLen = 20\nmodel = Emojify_V2((maxLen,), word_to_vec_map, word_to_index)\nmodel.summary()\nmodel.compile(loss=\'categorical_crossentropy\', optimizer=\'adam\', metrics=[\'accuracy\'])\nX_train_indices = sentences_to_indices(X_train, word_to_index, maxLen)\nY_train_oh = convert_to_one_hot(Y_train, C = 5)\nmodel.fit(X_train_indices, Y_train_oh, epochs = 50, batch_size = 32, shuffle=True)\nX_test_indices = sentences_to_indices(X_test, word_to_index, max_len = maxLen)\nY_test_oh = convert_to_one_hot(Y_test, C = 5)\nloss, acc = model.evaluate(X_test_indices, Y_test_oh)\nprint()\nprint("Test accuracy = ", acc)\n'



## debiasing word embeddings
