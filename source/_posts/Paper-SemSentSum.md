---
title: 多文档摘要
date: 2020-07-23 17:20:08
categories: 
- 论文复现
tags:
- 多文档摘要
---
多文档摘要论文复现
 <!-- more -->
## 一.数据加载

 **- 1.数据集说明**
  文章采用了Document Understanding Conferences（DUC）上针对最常用的多文档摘要数据集，其中DUC 2001/2002用于训练，DUC 2003用于验证，最后DUC 2004用于测试。
  

> 数据集需要在https://www-nlpir.nist.gov/projects/duc/guidelines.html进行申请

 **- 2.读取数据**
 首先，从文档中读取数据，然后切分数据集，由于机器的性能，我没有读取全部的数据集，只读取了部分的数据集进行实验。我是用正则表达式，从对应的文档中读取了文档的正文以及文档对应的参考摘要。
 格式为{doc_no : 文档内容}和{doc_no : 参考摘要}

 **- 3.切分句子**
 读取完数据之后，需要将文档切分成句子，存储格式为{doc_no :{sen_id:句子1 ，sen_id2 :句子2 }}

 **- 4.将切分完的句子建立索引**
 建立（index->句子) 的字典，格式为{index1 : doc_no#sen_Id1，index2 : doc_no#sen_Id2}
  建立（句子->index) 的字典，格式为{doc_no#sen_Id1 :index1，doc_no#sen_Id2 : index2}
  建立 （文档->index）的字典，格式为 {doc_no :  [index1,index2...] }
  将训练集，验证集，测试集对应的index存储到对应的列表中。

 **- 5.代码实现**
 

```python
# coding = utf-8
import os
import re
import util


class LoadData(object):
    def __init__(self):
        self.data_set = {}
        self.data_set_summary = {}
        self.current_path = os.path.abspath(os.path.dirname(__file__))  # 当前文件路径
        self.train_set = {}  # 训练集
        self.validation_set = {}  # 验证集
        self.test_set = {}  # 测试集

        self.train_set_len = 0
        self.validation_set_len = 0
        self.test_set_len = 0

        self.train_set_sentences = {}  # 训练集  将文档切分成了句子 存储格式为 {doc_no :{sen_id:句子1 ,sen_id2 :句子2 }}
        self.validation_set_sentences = {}  # 验证集
        self.test_set_sentences = {}  # 测试集


        #数据集
        self.index_to_sentence = {}  # 下标与 文档句子的映射

        self.sentence_to_index = {}  # 文档中的句子与下标的映射
        # 训练集
        self.train_set_sencence_count = 0

        self.train_set_sentences_list = {}  # 文档 与index的映射    {doc_no :  [index1,index2...]}

        self.train_index = []  # 训练集的所有下标
        # 验证集
        self.validation_set_sencence_count = 0
        self.validation_set_sentences_list = {}  # 文档 与index的映射    {doc_no :  [index1,index2...]}

        self.validation_index = []  # 训练集的所有下标
        # 测试集
        self.test_set_sencence_count = 0
        self.test_set_sentences_list = {}  # 文档 与index的映射    {doc_no :  [index1,index2...]}

        self.test_index = []  # 训练集的所有下标

        self.train_set_summary = {}  # 训练集对应的 参考摘要
        self.validation_set_summary = {}  # 验证集对应的 参考摘要
        self.test_set_summary = {}  # 测试集对应的 参考摘要

    def read_data(self):
        DUC2001_path = self.current_path + '/data/DUC/DUC2001_Summarization_Documents/data/training'  # 待读取文件的文件夹地址
        DUC2001_files = os.listdir(DUC2001_path)  # 获得文件夹中所有文件的名称列表

        for file in DUC2001_files:
            if os.path.isdir(DUC2001_path + "/" + file):  # 判断是否是文件夹
                doc_path_list = DUC2001_path + "/" + file + "/docs"
                summary_path = DUC2001_path + "/" + file + "/" + file + str(file)[-1]
                for doc_path in os.listdir(doc_path_list):
                    if os.path.isfile(doc_path_list + "/" + doc_path):
                        self.get_data(doc_path_list + "/" + doc_path, summary_path + "/perdocs")
        DUC2001_path_test = self.current_path + '/data/DUC/DUC2001_Summarization_Documents/data/test'  # 待读取文件的文件夹地址
        DUC2001_files_test = os.listdir(DUC2001_path_test + "/docs")  # 获得文件夹中所有文件的名称列表
        for file in DUC2001_files_test:
            if os.path.isdir(DUC2001_path_test + "/docs/" + file):  # 判断是否是文件夹
                doc_path_list = DUC2001_path_test + "/docs/" + file
                summary_path = DUC2001_path_test + "/original.summaries/" + file + str(file)[-1]
                for doc_path in os.listdir(doc_path_list):
                    if os.path.isfile(doc_path_list + "/" + doc_path):
                        self.get_data(doc_path_list + "/" + doc_path, summary_path + "/perdocs")

        DUC2001_path_testtraining = self.current_path + '/data/DUC/DUC2001_Summarization_Documents/data/testtraining/duc2002testtraining'  # 待读取文件的文件夹地址
        DUC2001_files_testtraining = os.listdir(DUC2001_path_testtraining)  # 获得文件夹中所有文件的名称列表
        for file in DUC2001_files_testtraining:
            if os.path.isdir(DUC2001_path_testtraining + "/" + file):  # 判断是否是文件夹
                path_list = os.listdir(DUC2001_path_testtraining + "/" + file)
                for path in path_list:
                    doc_no = path
                    fr_doc = open(DUC2001_path_testtraining + "/" + file + "/" + path + "/" + path + ".body",
                                  encoding='utf-8')
                    content = fr_doc.read()
                    content = content.replace("\n", "")
                    self.data_set[doc_no] = content
                    fr_summary = open(DUC2001_path_testtraining + "/" + file + "/" + path + "/" + path + ".abs",
                                      encoding='utf-8')
                    summary = fr_summary.read()
                    summary = summary.replace("\n", "")
                    self.data_set_summary[doc_no] = summary
        length = len(self.data_set)
        self.train_set_len = int(length * 0.8)
        self.validation_set_len = int(length * 0.1)
        self.test_set_len = int(length * 0.1)
        # 切分读取的数据为训练集 验证集 测试集
        index = 0
        for doc_no,content in self.data_set.items():
            if index < self.train_set_len:
                self.train_set[doc_no] = content
                self.train_set_summary[doc_no] = self.data_set_summary[doc_no]
            elif index > self.train_set_len and index<self.train_set_len+self.validation_set_len:
                self.validation_set[doc_no] = content
                self.validation_set_summary[doc_no] = self.data_set_summary[doc_no]
            elif index > self.train_set_len+self.validation_set_len and index < length:
                self.test_set[doc_no] = content
                self.test_set_summary[doc_no] = self.data_set_summary[doc_no]
            index +=1
      """利用正则表达式，提取文档内容和文档的摘要"""
    def get_data(self, doc_path, summary_path):
        fr = open(doc_path, "r", encoding='utf-8')
        content = fr.read()
        content = content.replace("\n", "")
        doc = re.findall("<TEXT>(.*?)</TEXT>", content)[0]
        doc_no = re.findall("<DOCNO>(.*?)</DOCNO>", content)[0].replace(" ", "")
        self.data_set[doc_no] = doc.replace("<p>", "").replace("</p>", "").replace("<P>", "").replace(
            "</P>", "")
        fr = open(summary_path, "r", encoding='utf-8')
        summary_list = fr.read()
        summary_list = summary_list.replace("\n", "")
        summary = re.findall('<SUM.*?DOCREF="{}.*?">(.*?)</SUM>'.format(doc_no), summary_list)[0]
        self.data_set_summary[doc_no] = summary

    """将文档切分成句子"""

    def cut_doc_to_sentences(self, set="train_set"):
        sentences_count = 0  # 句子数量
        j = 0
        if set == "train_set":
            # 读取数据
            for doc_no, doc_content in self.train_set.items():
                # 将文档切分成句子
                sentence_list = util.cut_doc_to_sentences(doc_content)
                if (len(sentence_list)) == 0:
                    break

                document_dict = {}
                sentences_count += len(sentence_list)
                for i in range(len(sentence_list)):
                    # 将句子编号 然后存入字典中
                    document_dict["sen_id_" + str(i)] = sentence_list[i]
                self.train_set_sentences[doc_no] = document_dict
            self.train_set_sencence_count = sentences_count
        elif set == "validation_set":
            for doc_no, doc_content in self.validation_set.items():
                sentence_list = util.cut_doc_to_sentences(doc_content)
                if (len(sentence_list)) == 0:
                    break
                document_dict = {}
                sentences_count += len(sentence_list)
                for i in range(len(sentence_list)):
                    document_dict["sen_id_" + str(i)] = sentence_list[i]
                self.validation_set_sentences[doc_no] = document_dict
            self.validation_set_sencence_count = sentences_count
        elif set == "test_set":
            for doc_no, doc_content in self.test_set.items():
                sentence_list = util.cut_doc_to_sentences(doc_content)
                if (len(sentence_list)) == 0:
                    break
                document_dict = {}
                sentences_count += len(sentence_list)
                for i in range(len(sentence_list)):
                    document_dict["sen_id_" + str(i)] = sentence_list[i]
                self.test_set_sentences[doc_no] = document_dict
            self.test_set_sencence_count = sentences_count

    def create_index(self):
        # 构造句子和矩阵下标的映射
        index = 0
        for doc_no, sentences in self.train_set_sentences.items():
            # 遍历文档中的每个句子
            sentence_index_list = []
            for sen_id, sentence in sentences.items():
                self.index_to_sentence[index] = doc_no + "#" + sen_id
                self.sentence_to_index[doc_no + "#" + sen_id] = index
                sentence_index_list.append(index)
                self.train_index.append(index)
                index += 1

            self.train_set_sentences_list[doc_no] = sentence_index_list

        # 构造句子和矩阵下标的映射
        for doc_no, sentences in self.validation_set_sentences.items():
            # 遍历文档中的每个句子
            sentence_index_list = []
            for sen_id, sentence in sentences.items():
                self.index_to_sentence[index] = doc_no + "#" + sen_id
                self.sentence_to_index[doc_no + "#" + sen_id] = index
                sentence_index_list.append(index)
                self.validation_index.append(index)
                index += 1
            self.validation_set_sentences_list[doc_no] = sentence_index_list
        # 构造句子和矩阵下标的映射
        for doc_no, sentences in self.test_set_sentences.items():
            # 遍历文档中的每个句子
            sentence_index_list = []
            for sen_id, sentence in sentences.items():
                self.index_to_sentence[index] = doc_no + "#" + sen_id
                self.sentence_to_index[doc_no + "#" + sen_id] = index
                sentence_index_list.append(index)
                self.test_index.append(index)
                index += 1
            self.test_set_sentences_list[doc_no] = sentence_index_list

    def read(self):
        self.read_data()
        self.cut_doc_to_sentences(set="train_set")
        self.cut_doc_to_sentences(set="validation_set")
        self.cut_doc_to_sentences(set="test_set")
        self.create_index()

```


 


## 二.构建句子的语义关系图（Sentence Semantic Relation Graph）

 - **1.解释：**

（1）用图对句子建模，图的顶点为文档i中的句子j（Si，j），边为两个句子之间的相似程度。需要使用英语Wikipedia语料库上训练的的模型进行句子嵌入（sentence embeddings），产生句子的向量，然后根据向量计算句子之间的余弦相似度，然后构建矩阵。
（2）引入一个阈值t，去除相似度小于阈值t的边，以强调较高的句子相似度，避免模型无法显著地利用句子之间的语义结构

 - **2.sentence embeddings环境安装工作**

我找到了（Pagliardini et al. (2018)）的论文中描述的模型的开源实现，他可以在fasttext库的基础上，进行句子嵌入。

> sent2vec模型地址：https://github.com/epfml/sent2vec

（1）由于他基于fasttext，所以先下载并编译了fasttext，具体方法他在github中写的非常清楚，在他们目录文件中写好了Makefile，直接用本地的gcc环境进行编译

> https://github.com/facebookresearch/fastText

（2）下载论文中描述的的wiki百科600维的预训练unigram模型，在fasttext中进行使用

> https://drive.google.com/uc?id=0B6VhzidiLvjSa19uYWlLUEkzX3c&export=download

（3）下载了斯坦福解析器，配合nltk库进行Tokenizer

> 解析器下载：https://nlp.stanford.edu/software/lex-parser.shtml#Download
> 使用教程：https://blog.csdn.net/qq_36652619/article/details/75091327

 - **3.使用nltk库和斯坦福解析器，进行Tokenizer，然后使用wiki百科600维的预训练unigram模型进行sentence embeddings**

```python
import os
import time
import re
from subprocess import call
import numpy as np
from nltk.tokenize.stanford import StanfordTokenizer
class SentencesEmbeddings(object):
    def __init__(self):

        #fasttext执行
        self.FASTTEXT_EXEC_PATH = os.path.abspath("./sent2vec-master/fasttext")
        #斯坦福分词器的路径
        self.BASE_SNLP_PATH = "sent2vec-master/stanford-postagger-full/"
        self.SNLP_TAGGER_JAR = os.path.join(self.BASE_SNLP_PATH, "stanford-postagger.jar")
        #wiki百科预训练模型的路径
        self.MODEL_WIKI_UNIGRAMS = os.path.abspath("sent2vec-master/wiki_unigrams.bin")
        self.tknzr = StanfordTokenizer(self.SNLP_TAGGER_JAR, encoding='utf-8')
        print("SentencesEmbeddings初始化完成")
    #将句子去除符号，大小写转换后，进行分词
    def tokenize(self, sentence, to_lower=True):
        """Arguments:
            - tknzr: a tokenizer implementing the NLTK tokenizer interface
            - sentence: a string to be tokenized
            - to_lower: lowercasing or not
        """
        sentence = sentence.strip()
        sentence = ' '.join([self.format_token(x) for x in  self.tknzr.tokenize(sentence)])
        if to_lower:
            sentence = sentence.lower()
        sentence = re.sub('((www\.[^\s]+)|(https?://[^\s]+)|(http?://[^\s]+))','<url>',sentence) #replace urls by <url>
        sentence = re.sub('(\@[^\s]+)','<user>',sentence) #replace @user268 by <user>
        filter(lambda word: ' ' not in word, sentence)
        return sentence

    def format_token(self,token):
        """"""
        if token == '-LRB-':
            token = '('
        elif token == '-RRB-':
            token = ')'
        elif token == '-RSB-':
            token = ']'
        elif token == '-LSB-':
            token = '['
        elif token == '-LCB-':
            token = '{'
        elif token == '-RCB-':
            token = '}'
        return token

    def tokenize_sentences(self, sentences, to_lower=True):
        """Arguments:
            - tknzr: 斯坦福解析器
            - sentences:句子列表
            - to_lower: 是否转化成消协
        """
        #返回token化的结果
        return [self.tokenize( s, to_lower) for s in sentences]


    #
    def get_embeddings_for_preprocessed_sentences(self,sentences, model_path, fasttext_exec_path):
        """Arguments:
            - sentences:分词后的结果
            - model_path: wiki百科模型文件的路径
            - fasttext_exec_path: fasttext的路径
        """
        timestamp = str(time.time())
        test_path = os.path.abspath('./'+timestamp+'_fasttext.test.txt')
        embeddings_path = os.path.abspath('./'+timestamp+'_fasttext.embeddings.txt')
        self.dump_text_to_disk(test_path, sentences)
        call(fasttext_exec_path+
              ' print-sentence-vectors '+
              model_path + ' < '+
              test_path + ' > ' +
              embeddings_path, shell=True)
        embeddings = self.read_embeddings(embeddings_path)
        os.remove(test_path)
        os.remove(embeddings_path)
        assert(len(sentences) == len(embeddings))
        return np.array(embeddings)

    def read_embeddings(self,embeddings_path):
        """Arguments:
            - embeddings_path: path to the embeddings
        """
        with open(embeddings_path, 'r') as in_stream:
            embeddings = []
            for line in in_stream:
                line = '['+line.replace(' ',',')+']'
                embeddings.append(eval(line))
            return embeddings
        return []

    def dump_text_to_disk(self,file_path, X, Y=None):
        """Arguments:
            - file_path: where to dump the data
            - X: list of sentences to dump
            - Y: labels, if any
        """
        with open(file_path, 'w') as out_stream:
            if Y is not None:
                for x, y in zip(X, Y):
                    out_stream.write('__label__'+str(y)+' '+x+' \n')
            else:
                for x in X:
                    out_stream.write(x+' \n')
    #输入为一个句子的列表 返回embeeding之后的数据 600纬
    def get_sentence_embeddings(self,sentences):
        wiki_embeddings = None
        #加载斯坦福的分词器

        #进行分词操作
        s = ' <delimiter> '.join(sentences)
        tokenized_sentences_SNLP = self.tokenize_sentences([s])
        tokenized_sentences_SNLP = tokenized_sentences_SNLP[0].split(' <delimiter> ')
        assert(len(tokenized_sentences_SNLP) == len(sentences))
        #使用wiki百科预训练的模型进行embeddings
        wiki_embeddings = self.get_embeddings_for_preprocessed_sentences(tokenized_sentences_SNLP, \
                                         self.MODEL_WIKI_UNIGRAMS, self.FASTTEXT_EXEC_PATH)
        return wiki_embeddings




    def embeddings(self,sentences):

        my_embeddings = self.get_sentence_embeddings(sentences)
        return my_embeddings

```

 - **4.句子相似度的计算**

```python
#!/usr/bin/env Python
# coding=utf-8
"""
构造语义关系图  以及进行sentence encoder
"""
import SentencesEmbeddings
import numpy as np
import re
import util
class SentenceSemanticRelationGraph(object):
    def __init__(self, train_set_sentences,
                 validation_set_sentences,
                 test_set_sentences,
                 train_set_sencence_count,
                 validation_set_sencence_count,
                 test_set_sencence_count,
                 sentence_to_index,
                 index_to_sentence,
                 train_set_sentences_list,
                 validation_set_sentences_list,
                 test_set_sentences_list,
                 train_index,
                 validation_index,
                 test_index):
        self.sentences_embeddings = SentencesEmbeddings.SentencesEmbeddings()
        self.train_set_sentences = train_set_sentences  # 训练集  将文档切分成了句子 存储格式为 {doc_no :{sen_id:句子1 ,sen_id2 :句子2 }}
        self.validation_set_sentences = validation_set_sentences  # 验证集
        self.test_set_sentences = test_set_sentences  # 测试集
        self.train_set_sencence_count = train_set_sencence_count #训练集句子数量
        self.validation_set_sencence_count = validation_set_sencence_count  # 验证集句子数量
        self.test_set_sencence_count = test_set_sencence_count  # 测试集句子数量
        self.sentence_count = train_set_sencence_count+validation_set_sencence_count+test_set_sencence_count
        self.data_set_sentences_embeddings = {} #存储句子嵌入的集合

        self.data_set_sentences = {}
        self.tgsim = 0.35  #句子之间余弦相似度的阈值  大于阈值的边才会被保留

        #数据集相关
        self.index_to_sentence = index_to_sentence  # 余弦相似度矩阵的下标与 文档句子的映射

        self.sentence_to_index = sentence_to_index  # 文档中的句子与余弦相似度矩阵下标的映射
        self.data_set_cosine_similarity_matrix = None  # 存储余弦相似度的矩阵
        self.data_set_sentences_list = {}

        #训练集相关
        self.train_index = train_index #训练集的下标
        self.train_set_sentences_list = train_set_sentences_list  # 文档 与index的映射    {doc_no :  [index1,index2...]}
        #验证集相关
        self.validation_index = validation_index  # 验证集的下标

        self.validation_set_sentences_list = validation_set_sentences_list  # 文档 与index的映射    {doc_no :  [index1,index2...]}
        #测试集相关
        self.test_index = test_index  # 测试集的下标

        self.test_set_sentences_list = test_set_sentences_list  # 文档 与index的映射    {doc_no :  [index1,index2...]}
    """
    处理已经读入的训练集 测试集  验证集
    """
    def deal_data(self):
       for doc_no, sentences_list in self.train_set_sentences_list.items():
           self.data_set_sentences_list[doc_no] = sentences_list
       for doc_no, sentences_list in self.validation_set_sentences_list.items():
           self.data_set_sentences_list[doc_no] = sentences_list
       for doc_no, sentences_list in self.test_set_sentences_list.items():
           self.data_set_sentences_list[doc_no] = sentences_list
       for doc_no, sentences_list in self.train_set_sentences.items():
           self.data_set_sentences[doc_no] = sentences_list
       for doc_no, sentences_list in self.validation_set_sentences.items():
           self.data_set_sentences[doc_no] = sentences_list
       for doc_no, sentences_list in self.test_set_sentences.items():
           self.data_set_sentences[doc_no] = sentences_list

    """
    调用方法计算句子嵌入 
    """
    def calculate_sentences_embeddings(self):
        # 以文档为单位计算句子嵌入  遍历每个文档的每一句话的index
        for doc_no, sentence_index_list in self.data_set_sentences_list.items():
            index_list = []
            sentence_list = []
            # 根据index找到对应的句子
            for index in sentence_index_list:
                # sen_id = str(self.train_set_index_to_sentence[index]).split("#")[1]
                sen_id = str(self.index_to_sentence[index]).split("#")[1]
                sentence = self.data_set_sentences[doc_no][sen_id]
                sentence_list.append(sentence)
                index_list.append(index)
            # 将文档中所有的句子加入列表，统一计算句子嵌入
            sentence_embedding_list = self.sentences_embeddings.embeddings(sentence_list)
            # 然后加入句子嵌入的列表
            for i in range(len(index_list)):
                self.data_set_sentences_embeddings[index_list[i]] = sentence_embedding_list[i]

    """
    计算句子间的余弦相似度 构造语义关系图
    """
    def calculate_cosine_similarity_matrix(self):
        # 初始化余弦相似度的矩阵
        self.data_set_cosine_similarity_matrix = np.zeros(
            (self.sentence_count, self.sentence_count))
        # 遍历映射 进行余弦相似度的计算
        for index, id in self.index_to_sentence.items():
            for index2, id2 in self.index_to_sentence.items():
                if index == index2 :
                    break
                if self.data_set_cosine_similarity_matrix[index][index2] == 0:
                    # 句子1 的嵌入
                    embeddings1 = self.data_set_sentences_embeddings[index]
                    # 句子2的嵌入
                    embeddings2 = self.data_set_sentences_embeddings[index2]
                    cosine_similarity = self.calculate_cosine_similarity(embeddings1, embeddings2)
                    # 如果大于阈值 在进行存储
                    if cosine_similarity > self.tgsim:
                        self.data_set_cosine_similarity_matrix[index][index2] = cosine_similarity
                        self.data_set_cosine_similarity_matrix[index2][index] = cosine_similarity
                else:
                    continue
    """
    计算两个numpy向量之间的余弦相似度
    """
    def calculate_cosine_similarity(self,vector1,vector2):
        cosine_similarity = np.dot(vector1,vector2)/(np.linalg.norm(vector1)*(np.linalg.norm(vector2)))
        return cosine_similarity
```

解释：

 1. 获得读取到的数据，将其构成几个集合：
 （1）data_set_sentences_list——表示文档的句子的索引,存储方式为{doc_no : [index1,index2]}，表示一篇文档句子的存储位置
 （2）index_to_sentence——表示文档句子的索引与文档句子编号的映射（可以根据index找到属于那篇文档的那个句子），存储方式为{index : doc_no#sen_id}
 （3）sentence_to_index——表示文档句子编号与文档句子索引的的映射（可以根据某文档的某个句子找到他存储的index，存储方式为{doc_no#sen_id : index}
 （4）data_set_sentences——表示具体文档句子id与文档内容的映射，存储方式为{doc_no#sen_id : 句子内容}
 
 2. 然后调用方法计算每一篇文档的sentence embeddings，存储格式为{reviewer_id1：{sen_id1：embeddings1，sen_id2：embeddings2} ，reviewer_id2：{sen_id1：embeddings1，sen_id2：embeddings2} }。
 3. 最后借助sentence embeddings，来计算句子语义关系图data_set_cosine_similarity_matrix


至此，第一部分的工作完成。

## 三.句子编码（Sentence Encoder）

 **- 1.解释**
 在构建完语义关系图之后，还需要对训练集中的所有单词，使用300维的预先训练的GloVe嵌入（Pen- nington et al., 2014）进行单词嵌入（word embeddings），然后将单词嵌入输入句子编码器以计算句子嵌入，句子编码器使用了单层正向循环神经网络（ a single-layer forward recurrent neural network）的变体，长短时记忆神经网络LSTM，最后从隐藏层中提取句子嵌入，然后，将所有句子嵌入连接到一个矩阵X中，该矩阵X将构成图卷积网络将使用的输入节点特征。
 

 **- 2.循环神经网络RNN的学习**
 

> https://zhuanlan.zhihu.com/p/50915723
> https://zhuanlan.zhihu.com/p/30844905

![在这里插入图片描述](https://img-blog.csdnimg.cn/202007231606529.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
循环神经网络的隐藏层的值s不仅仅取决于当前这次的输入x，还取决于上一次隐藏层的值s。权重矩阵 W就是隐藏层上一次的值作为这一次的输入的权重。对于文本数据，句子前后是有语义关系的，所以RNN正好可以处理句子前后关系的影响。但是RNN也有序列过长的情况下梯度消失或者梯度爆炸的问题。


 **- 3.LSTM：**

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020072316103727.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200723161135460.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
用pytorch实现lstm：


> https://pytorch.org/docs/master/generated/torch.nn.LSTM.html
> https://zhuanlan.zhihu.com/p/79064602

```python
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')

class EncoderRNN(nn.Module):
    def __init__(self, input_size, hidden_size,batch_size):
        super(EncoderRNN, self).__init__()
        self.hidden_size = hidden_size #
        self.input_size = input_size #每个单词向量的长度
        self.batch_size = batch_size #句子的个数
        # requires_grad指定是否在训练过程中对词向量的权重进行微调
        # self.embedding.weight.requires_grad = True
        self.lstm = nn.LSTM(self.input_size, hidden_size)
        # self.gru = nn.GRU(hidden_size, hidden_size)

    def forward(self, input, hidden,cell,seq_len,batch_size):
        """

        :param input:
        :param hidden:
        :param cell:
        :param seq_len:     句子的长度
        :param batch_size:  句子的个数
        :return:
        """
        # embedded = self.embedding[input].view(1, 1, self.hidden_size)
        # embedded = input.view(len(input), 1, 300)
        embedded = input.view(seq_len, batch_size, 300)
        # output, hidden = self.gru(embedded, hidden)
        output , (hidden, cell)= self.lstm(embedded.float(), (hidden.float(),cell.float()))
        return output,hidden,cell
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200723162557845.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200723163050742.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

（1）torch.nn.LSTM(*args, kwargs)
参数列表：
– input_size ：输入特征维数，如我们输入的是300维的预先训练的GloVe嵌入产生的词向量
– hidden_size：隐藏层状态的维数，即隐藏层节点的个数，这里我选择输入的也是300维
– num_layers： LSTM 堆叠的层数，默认值是1层，我们的神经网络也是单层的
– bias： 隐层状态是否带bias，默认为true。bias是偏置值，或者偏移值。没有偏置值就是以0为中轴，或以0为起点。
– batch_first：输入输出的第一维是否为 batch_size，默认值 False。因为 Torch 中，人们习惯使用Torch中带有的dataset，dataloader向神经网络模型连续输入数据，这里面就有一个 batch_size 的参数，表示一次输入多少个数据。 在 LSTM 模型中，输入数据必须是一批数据，为了区分LSTM中的批量数据和dataloader中的批量数据是否相同意义，LSTM 模型就通过这个参数的设定来区分。 如果是相同意义的，就设置为True，如果不同意义的，设置为False。 torch.LSTM 中 batch_size 维度默认是放在第二维度，故此参数设置可以将 batch_size 放在第一维度。如：input 默认是(4,1,5)，中间的 1 是 batch_size，指定batch_first=True后就是(1,4,5)。所以，如果你的输入数据是二维数据的话，就应该将 batch_first 设置为True;

– dropout： 默认值0。是否在除最后一个 RNN 层外的其他 RNN 层后面加 dropout 层。输入值是 0-1 之间的小数，表示概率。0表示0概率dripout，即不dropout
– bidirectional： 是否是双向 RNN，默认为：false，若为 true，则：num_directions=2，否则为1。 
（2）向前传播时的输入
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200729175411378.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)


![在这里插入图片描述](https://img-blog.csdnimg.cn/20200723163451721.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200729175639695.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
输入的张量的每一个维度都有固定的含义，不能弄错，需要在理解之后，才能进行修改。


 - **4.word embeddings（使用预训练的glove进行词嵌入）**
 
 （1）下载预训练的模型
 

> https://nlp.stanford.edu/projects/glove/

（2）glove模型的使用
使用python的gensim工具包。首先需要将这个训练好的模型转换成gensim方便加载的格式(gensim支持word2vec格式的预训练模型格式）

```python
from gensim.scripts.glove2word2vec import glove2word2vec
glove_input_file = 'data/glove.6B.300d.txt'
word2vec_output_file = 'data/glove.6B.300d.word2vec.txt'
glove2word2vec(self.glove_input_file, self.word2vec_output_file)
```
转换过模型格式后，就可以使用里面的词向量了：

```python
from gensim.models import KeyedVectors


# 加载模型
glove_model = KeyedVectors.load_word2vec_format(word2vec_output_file, binary=False)
# 获得单词cat的词向量
cat_vec = glove_model['cat']
print(cat_vec)
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200723164311128.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)

 - **5.具体实现**

```python
#!/usr/bin/env Python
# coding=utf-8
import torch
import torch.nn as nn
from torch import optim
import torch.nn.functional as F
import re
from gensim.scripts.glove2word2vec import glove2word2vec
from gensim.models import KeyedVectors
import numpy as np
import json
import util

device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')

class EncoderRNN(nn.Module):
    def __init__(self, input_size, hidden_size,batch_size):
        super(EncoderRNN, self).__init__()
        self.hidden_size = hidden_size #
        self.input_size = input_size #每个单词向量的长度
        self.batch_size = batch_size #句子的个数
        # requires_grad指定是否在训练过程中对词向量的权重进行微调
        # self.embedding.weight.requires_grad = True
        self.lstm = nn.LSTM(self.input_size, hidden_size)
        # self.gru = nn.GRU(hidden_size, hidden_size)

    def forward(self, input, hidden,cell,seq_len,batch_size):
        """

        :param input:
        :param hidden:
        :param cell:
        :param seq_len:     句子的长度
        :param batch_size:  句子的个数
        :return:
        """
        # embedded = self.embedding[input].view(1, 1, self.hidden_size)
        # embedded = input.view(len(input), 1, 300)
        embedded = input.view(seq_len, batch_size, 300)
        # output, hidden = self.gru(embedded, hidden)
        output , (hidden, cell)= self.lstm(embedded.float(), (hidden.float(),cell.float()))
        return output,hidden,cell

    def initHidden(self):

        #各个维度的含义是 (Seguence, minibatch_size, hidden_dim)
        # 1个LSTM层，batch_size=句子的个数, 隐藏层的特征维度300
        return torch.zeros(1, self.batch_size, self.hidden_size, device=device,dtype=torch.float)



class SentenceEncoder(object):
    def __init__(self,data_set_sentences,
                 sentence_to_index,
                 index_to_sentence)
        self.data_set_sentences = data_set_sentences  #数据集  将文档切分成了句子 存储格式为 {doc_no :{sen_id:句子1 ,sen_id2 :句子2 }}
        #有关数据集
        self.data_set_word = {} # 将句子切分成单词  {doc_no:{sen_id:[word1,word2]}}
        self.sentence_to_index = sentence_to_index  # 训练集 将句子切分成单词  然后与下标的映射  句子→索引的字典  {reviewer_id+"#"+sen_id : index}
        self.index_to_sentence = index_to_sentence  # 索引→单词的字典 {index  :  reviewer_id+"#"+sen_id }
        self.data_set_sentence_word = {} #{index : [单词1  单词2  单词3 ]}
        self.data_set_sentence_count = 0  # 句子的数量
        self.max_sentence_length = 0#最长句子的长度
        self.data_set_sentence_word_to_vec = {}
        self.data_set_word_to_index = {}#单词→索引
        self.data_set_word_count = {}#每个单词的计数
        self.index_to_data_set_word = {}#索引→单词
        self.word_to_vec = {}  #单词与向量的映射
        #定义了一个unknown的词，也就是说没有出现在训练集里的词，我们都叫做unknown，词向量就定义为0
        self.number_word = 0#词的数量
        #词向量的维度
        self.embeddings_size = 300
        self.data_set_sentence_word_encoder = None

        self.glove_input_file = 'data/glove.6B.300d.txt'
        self.word2vec_output_file = 'data/glove.6B.300d.word2vec.txt'
        # 加载预训练的glove模型
        self.glove_model = KeyedVectors.load_word2vec_format(self.word2vec_output_file, binary=False)
    def deal_data(self):
        #索引标记
        for index, sentence in self.index_to_sentence.items():
            doc_no = str(sentence).split("#")[0]
            sen_id = str(sentence).split("#")[1]
            sentence = self.data_set_sentences[doc_no][sen_id]
            # 句子转化为 单词列表
            word_list = self.sentence_to_word(sentence)
            self.data_set_sentence_word[index] = word_list
            self.data_set_sentence_count += 1
    # 将句子转化成词语
    def sentence_to_word(self, sentence):
        sentence = util.normalize_string(sentence)
        word_list= []
        for word in sentence.split(' '):
            word = util.normalize_string(word)
            if word != "":
                self.addWord(word)
                word_list.append(word)
        if len(word_list) >self.max_sentence_length:
            self.max_sentence_length = len(word_list)
        return word_list

    def addWord(self, word):
        if word not in self.data_set_word_to_index:
            self.data_set_word_to_index[word] = self.number_word
            self.data_set_word_count[word] = 1
            self.index_to_data_set_word[self.number_word] = word
            self.number_word += 1
        else:
            self.data_set_word_count[word] += 1


    #首先需要将这个训练好的模型转换成gensim方便加载的格式(gensim支持word2vec格式的预训练模型格式）
    def glove_to_word2vec(self):
        glove2word2vec(self.glove_input_file, self.word2vec_output_file)

    """
    获得词向量
    """
    def get_word_vec(self,word):
        return self.glove_model[word]

    def word_to_vector(self):
        for index , word in self.index_to_data_set_word.items():
            try:
                word_vec = self.get_word_vec(word)
            except KeyError:
                word_vec = np.zeros(300)

            self.word_to_vec[word] = word_vec


    """
    将句子使用rnn进行encoder
    """
    def sentence_encoder(self):
        #遍历每句话中的每个单词
        hedden_size = 300
        encoder = EncoderRNN(300, hedden_size,self.data_set_sentence_count).to(device)
        hidden = encoder.initHidden()
        cell = encoder.initHidden()
        output = None
        #所有句子的 向量列表 三维的
        sentences_vector_list = []
        for index ,words in self.data_set_sentence_word.items():
            #每句话的单词进行word embeddings之后生成的矩阵
            sentence_vector_list = []
            count = 0
            for word in words:
            #将单词转化为词向量
                sentence_vector_list.append(self.word_to_vec[word])
                count +=1
            #然后将所有的句子都结合在一起
            sentence_vector_list = np.array(sentence_vector_list)
            # 在数组A的边缘填充constant_values指定的数值
            # （3,2）表示在A的第[0]轴填充（二维数组中，0轴表示行），即在0轴前面填充3个宽度的0，比如数组A中的95,96两个元素前面各填充了3个0；在后面填充2个0，比如数组A中的97,98两个元素后面各填充了2个0stant_values表示填充值，且(b
            # （2,3）表示在A的第[1]轴填充（二维数组中，1轴表示列），即在1轴前面填充2个宽度的0，后面填充3个宽度的0
            sentence_vector_list = np.pad(sentence_vector_list,((0,self.max_sentence_length-count),(0,0)),'constant',constant_values = (0,0))
            sentences_vector_list.append(sentence_vector_list)
        output,hidden, cell = encoder(self.sentence_to_tensor(sentences_vector_list), hidden, cell,self.max_sentence_length,self.data_set_sentence_count)
        print(hidden.shape)
        print(output.shape)
            # self.train_set_sentence_word_encoder[index] = hidden[0].detach().numpy()
        self.data_set_sentence_word_encoder = output[output.shape[0]-1].tolist()
        # print(self.train_set_sentence_word_encoder)
    """
    将句子向量转化成tensor
    """
    def sentence_to_tensor(self, vector_list):
        vector_array = np.array(vector_list)
        tensor = torch.tensor(vector_array, dtype=torch.float, device=device).view(self.data_set_sentence_count,self.max_sentence_length,300)
        return tensor
```

 说明：
 （1）读入数据
index_to_sentence——表示文档句子的索引与文档句子编号的映射（可以根据index找到属于那篇文档的那个句子），存储方式为{index : doc_no#sen_id}
sentence_to_index——表示文档句子编号与文档句子索引的的映射（可以根据某文档的某个句子找到他存储的index，存储方式为{doc_no#sen_id : index}
data_set_sentences——表示具体文档句子id与文档内容的映射，存储方式为{doc_no#sen_id : 句子内容}
 

（2）对数据进行处理

```python
self.train_set_word_to_index = {}#单词→索引
self.train_set_word_count = {}#每个单词的计数
self.index_to_train_set_word = {}#索引→单词
```
将训练集中所有出现过的单词进行统计，建立单个词语的索引。

```python
 def deal_data(self):
        #索引标记
        for index, sentence in self.index_to_sentence.items():
            doc_no = str(sentence).split("#")[0]
            sen_id = str(sentence).split("#")[1]
            sentence = self.data_set_sentences[doc_no][sen_id]
            # 句子转化为 单词列表
            word_list = self.sentence_to_word(sentence)
            self.data_set_sentence_word[index] = word_list
            self.data_set_sentence_count += 1
    # 将句子转化成词语
    def sentence_to_word(self, sentence):
        sentence = util.normalize_string(sentence)
        word_list= []
        for word in sentence.split(' '):
            word = util.normalize_string(word)
            if word != "":
                self.addWord(word)
                word_list.append(word)
        if len(word_list) >self.max_sentence_length:
            self.max_sentence_length = len(word_list)
        return word_list

    def addWord(self, word):
        if word not in self.data_set_word_to_index:
            self.data_set_word_to_index[word] = self.number_word
            self.data_set_word_count[word] = 1
            self.index_to_data_set_word[self.number_word] = word
            self.number_word += 1
        else:
            self.data_set_word_count[word] += 1
```
（3）  将句子使用LSTM进行encoder
 


使用之前处理过的句子的单词列表，求出每一句话中每一个单词的word embedding （1*300），然后将这句话中的所有单词的word embedding ，组成一个 （句子中单词个数*300）维的矩阵。然后将这个矩阵的行数，补全成了最长句子的单词个数，最终形成的矩阵是 （最长句子的单词个数*300）。

我将所有句子都这样操作，形成了一个三维的张量 （句子个数 * 最长句子的单词个数*300）作为LSTM的输入。

那么在初始化LSTM时 为nn.LSTM(每个单词向量的长度——300, hidden_size)
   
 然后我将hidden 和cell初始化成了 torch.zeros(1, 句子的个数, hidden_size)

 - 对于input(seq_len, batch, input_size)  的参数 ：

seq_len是序列的个数，对于句子来说，应该是句子的长度，应该是每一句话中单词的个数，这个是需要固定的 ，取了最长的句子的单词数。

batch表示一次性喂给网络多少条句子，初始化成句子的个数

input_size应该是每个具体的输入是多少维的向量，这里应该是300（word embedding的维度数量）

最终的hidden_size =300  最终生成了（句子的个数*300）的矩阵，作为Encoder的输出。

```python
  #首先需要将这个训练好的模型转换成gensim方便加载的格式(gensim支持word2vec格式的预训练模型格式）
    def glove_to_word2vec(self):
        glove2word2vec(self.glove_input_file, self.word2vec_output_file)

    """
    获得词向量
    """
    def get_word_vec(self,word):
        return self.glove_model[word]

    def word_to_vector(self):
        for index , word in self.index_to_data_set_word.items():
            try:
                word_vec = self.get_word_vec(word)
            except KeyError:
                word_vec = np.zeros(300)

            self.word_to_vec[word] = word_vec


    """
    将句子使用rnn进行encoder
    """
    def sentence_encoder(self):
        #遍历每句话中的每个单词
        hedden_size = 300
        encoder = EncoderRNN(300, hedden_size,self.data_set_sentence_count).to(device)
        hidden = encoder.initHidden()
        cell = encoder.initHidden()
        output = None
        #所有句子的 向量列表 三维的
        sentences_vector_list = []
        for index ,words in self.data_set_sentence_word.items():
            #每句话的单词进行word embeddings之后生成的矩阵
            sentence_vector_list = []
            count = 0
            for word in words:
            #将单词转化为词向量
                sentence_vector_list.append(self.word_to_vec[word])
                count +=1
            #然后将所有的句子都结合在一起
            sentence_vector_list = np.array(sentence_vector_list)
            # 在数组A的边缘填充constant_values指定的数值
            # （3,2）表示在A的第[0]轴填充（二维数组中，0轴表示行），即在0轴前面填充3个宽度的0，比如数组A中的95,96两个元素前面各填充了3个0；在后面填充2个0，比如数组A中的97,98两个元素后面各填充了2个0stant_values表示填充值，且(b
            # （2,3）表示在A的第[1]轴填充（二维数组中，1轴表示列），即在1轴前面填充2个宽度的0，后面填充3个宽度的0
            sentence_vector_list = np.pad(sentence_vector_list,((0,self.max_sentence_length-count),(0,0)),'constant',constant_values = (0,0))
            sentences_vector_list.append(sentence_vector_list)
        output,hidden, cell = encoder(self.sentence_to_tensor(sentences_vector_list), hidden, cell,self.max_sentence_length,self.data_set_sentence_count)
        print(hidden.shape)
        print(output.shape)
            # self.train_set_sentence_word_encoder[index] = hidden[0].detach().numpy()
        self.data_set_sentence_word_encoder = output[output.shape[0]-1].tolist()
        # print(self.train_set_sentence_word_encoder)
    """
    将句子向量转化成tensor
    """
    def sentence_to_tensor(self, vector_list):
        vector_array = np.array(vector_list)
        tensor = torch.tensor(vector_array, dtype=torch.float, device=device).view(self.data_set_sentence_count,self.max_sentence_length,300)
        return tensor
```

## 四.图卷积网络（Graph Convolutional Network）及显著性估计（Saliency Estimation）

 **- 1.前言**
（1）在计算完句子嵌入和句子语义关系图后，使用单层图卷积网络（GCN），以便捕获每个句子的高级隐藏特征，封装句子信息以及图结构。
图卷积网络的邻接矩阵为句子语义关系图加上单位矩阵的结果（A = A + I），特征矩阵为使用LSTM进行句子编码后的结果。
（2）然后使用以下等式，获取句子的隐藏特征
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200729202514147.png)
Wi是第i个图卷积层的权重矩阵，bi是偏差矢量。使用ELU作为激活函数。
（3）之后使用一个线性层来估计每个句子的显着性分数，然后通过softmax将分数归一化。

 **- 2.图卷积网络的学习与理解**

> 参考文章：https://zhuanlan.zhihu.com/p/54505069
> https://zhuanlan.zhihu.com/p/89503068
> https://www.cnblogs.com/SivilTaram/p/graph_neural_network_1.html

（1）图神经网络GNN
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200729203547914.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200729203558981.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200729203628762.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
（2）图卷积网络

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200729203718684.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200729203759813.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JhaWR1XzQxODcxNzk0,size_16,color_FFFFFF,t_70)
这个图也正好应证了论文中的公式
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200729202514147.png)
两层卷积层，每一层之后都有一个激活函数。

 **- 3.图卷积网络GCN的实现**
 **（1）卷积层的定义**
 卷积层输入维度为节点输入特征的维度，还可以设定偏差矢量。
 向前传播时，需要将节点的邻接矩阵和特征矩阵进行运算，然后进行相应的输出。

```python
class GraphConvolution(nn.Module):
    def __init__(self, input_dim, output_dim, use_bias=True):
        """图卷积：L*X*\theta
        Args:
        ----------
            input_dim: int
                节点输入特征的维度 D
            output_dim: int
                输出特征维度 D‘
            use_bias : bool, optional
                是否使用偏置
        """
        super(GraphConvolution, self).__init__()
        self.input_dim = input_dim
        self.output_dim = output_dim
        self.use_bias = use_bias
        # 定义GCN层的权重矩阵    input_dim=300  output_dim=300
        self.weight = nn.Parameter(torch.Tensor(input_dim, output_dim))
        if self.use_bias:
            self.bias = nn.Parameter(torch.Tensor(output_dim))
        else:
            self.register_parameter('bias', None)
        self.reset_parameters()  # 使用自定义的参数初始化方式

    def reset_parameters(self):
        # 自定义参数初始化方式
        # 权重参数初始化方式
        init.kaiming_uniform_(self.weight)
        if self.use_bias:  # 偏置参数初始化为0
            init.zeros_(self.bias)

    def forward(self, adjacency, input_feature):
        """邻接矩阵是稀疏矩阵，因此在计算时使用稀疏矩阵乘法

        Args:
        -------
            adjacency:
                邻接矩阵
            input_feature: torch.Tensor
                输入特征
        """
        #矩阵相乘
        # h =  ̃ELU(A X W0 +b0 )   A为邻接矩阵  X为特征矩阵  W0为权重矩阵  B0为偏差矢量
        # S = ̃ELU(A h W1 +b1 )
        support = torch.mm(input_feature, self.weight)  # X W (N,D');   X (N,D);W (D,D')   input_feature 维度为 句子个数*300  weight的维度为 300 * 300  输出为  句子个数 *300
        output = torch.mm(adjacency, support)  # (N,D')  #adjacency 为句子个数*句子个数     support 为 句子个数 *300   output为句子个数 *300
        #也可以使用稀疏矩阵的乘法
        # support = torch.mm(input_feature, self.weight) #XW (N,D');X (N,D);W (D,D')  
        #output = torch.sparse.mm(adjacency, support) #(N,D')

        if self.use_bias:
            output += self.bias
        return output
```
（2）GCN模型的定义
这个模型包含两个卷积层，并且在最终的输出前还使用了线性层以及softmax计算每一个句子的显著性分数。并且模型还使用了0.2的丢弃率。


```python
class GcnNet(nn.Module):
    """
    定义一个包含两层GraphConvolution的模型
    """
    def __init__(self, input_dim = 300 ,output_dim =300,dropout = 0.2):
        super(GcnNet, self).__init__()
        self.dropout = dropout
        self.gcn1 = GraphConvolution(input_dim, output_dim)
        self.gcn2 = GraphConvolution(input_dim, output_dim)
        #使用一个简单的线性层来估计每个句子的显着性分数,通过softmax将分数归一化并获得我们的显着性分数
        self.linear = nn.Linear(input_dim,output_dim)


    """
    S =ELU(A ̃ELU(A ̃XW +b )W +b )
    """
    def forward(self, adjacency, feature):
        #采用elu作为激活函数
        # h =  ̃ELU(A X W0 +b0 )   A为邻接矩阵  X为特征矩阵  W0为权重矩阵  B0为偏差矢量
        h = F.elu(self.gcn1(adjacency, feature))

        s = F.dropout(h,p = self.dropout)
        #S = ̃ELU(A h W1 +b1 )
        s = self.gcn2(adjacency, h)

        #使用一个简单的线性层来估计每个句子的显着性分数,通过softmax将分数归一化并获得我们的显着性分数
        output = self.linear(s)
        #输出为每个维度的得分 0-1之间
        output = F.softmax(output)
        return output

```

## 五.模型的训练及结果的评估

 **- 1.前言**
 模型SemSentSum以端到端的方式训练(end-to-end端到端指的是输入是原始数据，输出是最后的结果)，并且使每个句子的显着性得分预测和ROUGE-1 F1得分之间的等式2的交叉熵损失最小。
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200729210107803.png)

	

