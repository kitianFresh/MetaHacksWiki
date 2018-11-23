---
title: "word-embeddings"
date: 2018-02-23 13:01
---

[TOC]

# Word Embeddings#
## Intuition
在介绍词向量之前，我们先说如何在计算机中表示一个词。

不论是哪一国语言，如果我们考虑请语言学家，根据语法和规则来学习某门语言，由于每种语言有他的语法，耗费的人力成本和时间成本非常高，并且基于语法和规则的翻译，系统也是很难编写的。Google 就是一个例子，他们每开除一名语言学家，机器翻译准确率就会提升。尽管不同语言有不同的语法，但是他们都有更抽象abstract和更加通用general的结构能够被学习到，那是什么呢，你会发现，大多数语言都是由字符构成单词（词素，词素再构成单词un+fortunate+ly）（汉语，日语，韩语可能是基本笔画构成字），词构成句子，句子构成段落，段落构成文章。然后从语义上来讲，不同语言相同语义的某些词，他必定出现在不同语言相同语义的上下文，什么意思呢？比如，"机器学习是实现人工智能的最好的方法之一"，"machine learning is one of best ways of achieving artificial intelligience"。不管是哪一种语言，机器学习(machine learning)可能往往出现在人工智能(artificial intelligience)的上下文。这些都是可以学习的隐含pattern。你会发现，一旦你找到了一种可以学习这些pattern的模型，那么你就找到了更加通用的解决方案，不同的语言，使用语料库进行训练即可得到模型，不需要和传统的基于规则和语法的编程一样，每次换一门语言，需要重新学习规则，重新编写系统。

那什么是词向量呢？我们知道单词是所有语言的基础，所以表征出单词的含义，我们就成功了一半。我们需要判断两个词语是否同义或者同样的词性（语义和语法），有什么好的方法呢？暂停30s想一下，我们如何学习英语的同义词呢？我记得上高中的时候，我经常通过把一组同义词放在一起学习，背诵和比较，机器能这样做么？可以，但是得人工收集和分类很多不同的词语，然后对这些词语做标记，同一个意思的词语都是同样的标记，比如他们都是动词，都是喜欢的意思，让后存储在计算机中，直接查询？这太复杂了。。。。所以有大神尽然神奇的想到把一个词语使用向量来表示，what，fuck，这可以吗？学习计算机的同学可能知道，这样存储对计算机来讲太爽了，很容易设计数据结构和运算。但是，这是什么意义？能够有效果么？如何表示成向量？接下来我们就来看看到底是怎么做到的，为啥有效果，而且还秒杀传统的方法。

再次回到我开头说的，从语义上来讲，每一种语言的相同语义的某些词，他必定出现在不同语言相同语义的上下文，这些都是可以学习的隐含pattern。如果两个词是同义词，那是不是这两个词会很大的可能性出现在相同的上下文。然后我突然顿悟到，这不就是我考研时候使用的方法么，单词不认识，通过上下文推出他可能是某个词的同义词，只是没学过而已。我们会通过脑海里原始的一些pattern（我们见过的相似的句子以及上下文）推理出来，这个词应该就是那个词的意思！顿时感觉，大神的思路也是非常平凡的，难就难在，人家就给你做出来了。

然后，为什么要用向量来表示一个词语呢？有意义么？答案当然是有的！我们知道，向量是一组不同的元素构成的，我们最开始学习的向量，其实是物理中的矢量，就是带方向的，通过(x,y,z) 来分别指明三个方向，向量就是更高维度的矢量了，他的每一个维度都是有具体含义的。一个词是不是有不同的属性和特点，比如他是消极词的概率，他使用的频率，他是副词的概率，他是动词的概率等等。因此，向量可以比较完善的表示出一个词的意义和结构。

接下来，就是通过神经网络玄学来试试，能不能自动学习出一个向量，虽然我不知道这个向量每个维度到底是啥意思。

词嵌入向量具有很强的泛化和迁移学习能力，即使使用很小的训练集学习出来的词向量，也很容易应用与其他词。
## word represention
## embedding matrix
### 为什么Word2Vec 不使用正则项？
因为Word2Vec的目的并不是为了让这个模型去适应语料库意外的语料，也不是用来再去预测其他的未见过测用例，他只是为了训练出当前语料库中所有单词的词向量。只需要拟合当前数据即可，不需要泛化。
### 为什么Word2Vec 隐藏层没有使用激活函数？
目的不是为了泛化，也不需要激活，激活是为了

### Word2Vec 两个矩阵能够使用同一个吗？也就是说 W.T = W'
目前网上没人思考过这个问题，如果我们得到了第一个矩阵是词嵌入矩阵，他的每一行代表的是对应单词的词向量，然后采用计算隐层向量（他其实就是某个词向量）和其他所有词向量的距离，用这个距离来度量输入词和其他词之间的关系，以此再进行softmax概率分布来评估误差，但是你不能还是用第一个矩阵来学习，因为第一个矩阵是用来学习词嵌入的，第二个矩阵是用来学习词嵌入和他的上下文单词关系的，这是两个不同的学习对象，因此我们需要再引入一个新矩阵来学习。

目前我认为如果不能的话，应该是这个原因。但是如果可以的话，我觉得也说得通，这得做实验了。

### 为什么取第一个矩阵做词向量而不是第二个矩阵？
目前网上没人思考过这个问题，目前的解释认为从one-hot 输入到隐层其实是编码进入词向量空间，第一层映射是真实的拿到词向量，也就是说第一个矩阵才是学习我们需要的词向量的矩阵，从隐层到输出是解码到one-hot，但是第二层映射解码出来的目的不是变会原来输入单词的one-hot,而是尽可能匹配出上下文单词的one-hot。也即是说第二层映射过程中发生的是距离计算和度量，第二个矩阵学习到的是某个词向量和他上下文距离远近的关系。

# Learning Word Embeddings
## word2vec
## hierarchical softmax & negative sampling for speeding training

## GloVe word vectors
## 参考资料
 - 论文
  - [A-Neural-Probabilistic-Language-Model-1](http://wiki.hacksmeta.com/static/pdf/A-Neural-Probabilistic-Language-Model-1.pdf)
  - [Efficient-Estimation-of-Word-Representations](http://wiki.hacksmeta.com/static/pdf/Efficient-Estimation-of-Word-Representations-2.pdf)
  - [5021-distributed-representations-of-words-and-phrases-and-their-compositionality](http://wiki.hacksmeta.com/static/pdf/5021-distributed-representations-of-words-and-phrases-and-their-compositionality-3.pdf)
  - [GloVe-Global-Vectors-for-Word-Representation](http://wiki.hacksmeta.com/static/pdf/GloVe-Global-Vectors-for-Word-Representation-4.pdf)
  - [word2vec-Parameter-Learning-Explained](http://wiki.hacksmeta.com/static/pdf/word2vec-Parameter-Learning-Explained-5.pdf)
 - 博客
  - [秒懂词向量Word2vec的本质](https://mp.weixin.qq.com/s/aeoFx6sIX6WNch51XRF5sg)
  - [理解 Word2Vec 之 Skip-Gram 模型](https://zhuanlan.zhihu.com/p/27234078)
  - [深度学习word2vec笔记之基础篇算法篇应用篇--写的非常到位](http://blog.csdn.net/han____shuai/article/details/50882135)
  - [词嵌入的直觉理解：从计数向量到 Word2Vec](http://blog.idejie.com/2017/08/13/word-embeding/)
 - 实现
  - [Learn Word2Vec by implementing it in tensorflow](https://towardsdatascience.com/learn-word2vec-by-implementing-it-in-tensorflow-45641adaf2ac)
  - [自己动手写word2vec](http://blog.csdn.net/u014595019/article/details/51884529)
  - [Word2Vec (Part 1): NLP With Deep Learning with Tensorflow (Skip-gram)](http://www.thushv.com/natural_language_processing/word2vec-part-1-nlp-with-deep-learning-with-tensorflow-skip-gram/)
  - [Word2Vec word embedding tutorial in Python and TensorFlow](http://adventuresinmachinelearning.com/word2vec-tutorial-tensorflow/)
 - Word2Vec Resources
  - [Word2Vec Resources](http://mccormickml.com/2016/04/27/word2vec-resources/)
  - [理解 Word2Vec 之 Skip-Gram 模型](https://zhuanlan.zhihu.com/p/27234078)

 - Projects
  - [聊天机器人的开发思路](http://zake7749.github.io/2016/12/17/how-to-develop-chatbot/)
  - [新闻上的文本分类：机器学习大乱斗](https://zhuanlan.zhihu.com/p/26729228)
  - [jizhi](https://jizhi.im/index)
  - [biendata](https://biendata.com/)
  - [分分钟带你杀入Kaggle Top 1%](https://zhuanlan.zhihu.com/p/27424282)
- 文本分类和标签
  - [Hierarchical Attention Networks for Document Classification](https://www.cs.cmu.edu/~diyiy/docs/naacl16.pdf)
  - [Large Scale Multi-label Text Classification with Semantic Word Vectors](https://cs224d.stanford.edu/reports/BergerMark.pdf)
  - [Recurrent Neural Network for Text Classification with Multi-Task Learning](https://www.ijcai.org/Proceedings/16/Papers/408.pdf)
  - [A Sensitivity Analysis of (and Practitioners’ Guide to) Convolutional Neural Networks for Sentence Classification](https://arxiv.org/pdf/1510.03820.pdf)
  - [Character-level Convolutional Networks for Text Classification](https://arxiv.org/abs/1509.01626)

  - [Implementing a CNN for Text Classification in TensorFlow](http://www.wildml.com/2015/12/implementing-a-cnn-for-text-classification-in-tensorflow/)

- 比赛
 - [2017BDCI-360搜索：AlphaGo之后“人机大战”Round 2 ——机器写作与人类写作的巅峰对决](https://github.com/a550461053/BDCI2017-360)
 - [kaggle-toxic_comment](https://github.com/peterhurford/kaggle-toxic_comment)
 - [34th, Lots of FE and Poor Understanding of NNs CODE INCLUDED](https://www.kaggle.com/c/jigsaw-toxic-comment-classification-challenge/discussion/52645)
 - [Top 1% Solution to Toxic Comment Classification Challenge](https://medium.com/@zake7749/top-1-solution-to-toxic-comment-classification-challenge-ea28dbe75054)
 - [Lessons from Toxic : Blending is the new sexy](https://www.kaggle.com/jagangupta/lessons-from-toxic-blending-is-the-new-sexy)

  https://github.com/Qinbf/Tensorflow/blob/master/Tensorflow%E5%9F%BA%E7%A1%80%E4%BD%BF%E7%94%A8%E4%B8%8E%E6%96%87%E6%9C%AC%E5%88%86%E7%B1%BB%E5%BA%94%E7%94%A8/%E7%A8%8B%E5%BA%8F/data_handle.ipynb

就是如何判断一篇文章是否抄袭了另一篇文章，哪里抄袭了。如果是人工去判断的话，非常简单。我们一般会先扫描一遍，发现有很多重复的段落和句子，基本就可以判定他们很相似了。但是如果这两篇文章表达的是同样的内容，但是很多词被替换了，语句顺序也被打乱了，这个时候，光靠扫描句子词已经无法判断了，我们得看语义。这是我们人工的做法，具体到计算机，它要如何识别这种相似呢？我们也从人工上最简单的方法开始，文档基本上就是词构成的序列，如果我们比较这两个序列的差异（计算机里面有个算法叫最小编辑距离），就可以大致得到他们是否在字面上相似。但是当面临第二个问题的时候，这种做法就失效了。

第二个问题，最简单的办法是TF-IDF，把每一个文档看成是一个TF-IDF构成的向量，计算向量之间的距离来判断。TF-IDF靠的是词频统计，就是我们做一个词汇表，然后每一篇文章都是一个这种词汇表构成的向量，向量的每一个值是这个词在该文章中出现的统计词频。但是这种方法其实只是大致估算了一个文章中词的含量信息，没有给出更加具体的语义和语序比较，因此往往也不能有很高的识别率。

第二种方法就是分类器，如果两篇文章的作者都是丰富的写手，他们以前写过很多文章的话，我们可以做分类器，就是作者1的文章是一类，作者2的文章是一类，如果作者2的新文章被分到作者1的类，那这篇文章可能就是抄袭的。

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
<img src="/static/images/ML/DL/rnn/embedding1.png" style="width:900px;height:300px;">
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
