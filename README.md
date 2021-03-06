# CAIL
![比赛图标](https://github.com/renjunxiang/Competition_CAIL/blob/master/picture/比赛图标.png)<br>

## **比赛简介**
为了促进法律智能相关技术的发展，在最高人民法院信息中心、共青团中央青年发展部的指导下，中国司法大数据研究院、中国中文信息学会、中电科系统团委联合清华大学、北京大学、中国科学院软件研究所共同举办“2018中国‘法研杯’法律智能挑战赛（CAIL2018）”。<br>

## 比赛任务：
**罪名预测：根据刑事法律文书中的案情描述和事实部分，预测被告人被判的罪名；**<br>
**法条推荐：根据刑事法律文书中的案情描述和事实部分，预测本案涉及的相关法条；**<br>
**刑期预测：根据刑事法律文书中的案情描述和事实部分，预测被告人的刑期长短。**<br>



## **分析思路：**
**
第一轮： 分词 》 剔除停用词 》 特征表达 》 单一模型训练 》评估 》优化<br>
第二轮： 数据增强 》 单一模型训练 》评估 》优化<br>
第三轮： 模型融合 》 评估 》优化<br>
<br>
**基于此，我的做法是：**<br>
**分词**：结巴，可以考虑引入外部词库提高分词精确性<br>
<br>
**剔除停用词**：我直接把长度为1的字符串删了，这个也是偷懒但还挺有效的方法，标点、语气词等一般都是1个长度<br>
<br>
**特征表达**：我是先将词语转成id，一开始保留4万个词语效果不太好，成绩大致是86-83-72；后来保留8万个词语，成绩大概是87.5-84-73.5，然后接入embedding层计算词向量。词语少了特征表达不足，多了很多低频词汇干扰词向量的表达效果。这里可以尝试tf-idf来选择保留哪些词语，不过很多人名地名会有特殊性，有能力的还可以采用实体识别的方式做过滤<br>
<br>
**单一模型训练**：受限于硬件，Embedding(80000词语，400长度，词向量512维)，接一层CNN1D(512个卷积核、卷积核大小3)，接全局最大池化(GlobalMaxPool1D)，再正则化(BN)后接全连接，最后用sigmod做类别得分，第三问用relu。因为没有足够的时间去训练深度网络和模型融合，只能力求增加模型宽度+BN加速收敛<br>
另外第三问有很多人用分类模型，我在训练赛试过成绩不太好。我猜测是分类的话可能稍有不慎就归到了差距很大的类别，如果当成连续变量至多是两类的平均数，所以我更倾向于用连续变量来做。<br>
<br>
**评估**：一开始我是用准确率(accu)来做的，因为一开始探索的时候训练一般都是不充分的，accu可以看出多类别是不是能有效预测出来，训练赛感觉提交分数和accu的结果正相关；后来大数据集的时候多标签的样本显著减少，并且数据大了容易达到数据和模型的瓶颈，就考虑和官网一样用f1来评估，关注样本少的类别效果。<br>
<br>
**优化**：我能做的也只是1层变2层、RNN换CNN、增加词向量维度、增加卷积核数量和改变卷积核尺寸。测试下来1-3层卷积的成绩差不多，2层是最好的，但也只是高了0.1%左右，很可能只是运气问题。词向量512要高于256、卷积核数量512要高于256，rnn效果不如cnn(后面我会谈一下原因，但是多层的话我就不知道了)<br>
<br>
**数据增强**：因为句子长度差距比较大(我记得200长度好像92%，300长度95%，400长度98%)，我最终是取了400长度。<br>
尝试过对句子内部词语做混洗打乱顺序，成绩只降低了一点点。结合断案的思路，一般也是看fact的关键词，所以我认为CNN在这里的特征提取表现会好于RNN的上下文理解。<br>尝试过对类别样本少的做重抽样，试了几次扩大不超过60000、扩大倍数不超过10的效果比较好，扩大不超过100000、扩大倍数不超过100效果很差。可能因为类别样本少的特征不具有代表性，重抽样过拟合的可能性很大。<br>
<br>
**模型融合**：提交过一次RNN、CNN和残差CNN的融合，因为那天比赛服务器断电提交失败就没试过，一般情况下都会有提升。

