# 原比赛链接：https://www.datafountain.cn/competitions/337
这个题目是一个回归任务，通过身份特征、消费能力、人脉关系、位置轨迹、应用行为偏好信息来预测中国移动用户的信用分。

由于最近找实习需要，所以将其上传到github，之前的一些探索过程代码没有保留下来，只有最后一版代码文件。



**对初始数据进行预处理（缺失值处理、异常值检测等）**

缺失值的特征有 [‘用户年龄’, ‘缴费用户最近一次缴费金额（元）（缺失最多高达1/4，剩下的最多只缺失几百）’, ‘用户近6个月平均消费值（元）’,‘用户账单当月总费用（元）’，‘用户话费敏感度’（缺失最少）] ，对于缺失值的处理，其实尝试过很多方法来着，考虑到该数据是多元缺失，且各特征缺失量不一致，因此对于缺失数据较少的特征 ‘用户话费敏感度’ 我直接采用中位数填充了。其余相对缺失较多的主要尝试了映射到高维、不处理、多重插补等方法，最后线下验证结果表明不处理直接交给模型处理的效果最好。

异常值处理
长尾截断

“对当月通话交往圈人数进行截尾”；为什么只取消一个特征的拖尾，其它特征拖尾为什么保留，
即使线下提高分数也要保留,这是因为在线下中比如逛商场拖尾的数据真实场景下可能为保安，
在训练集中可能只有一个保安，所以去掉以后线下验证会提高，但是在测试集中也存在一个保安，
如果失去拖尾最终会导致测试集保安信用分精度下降 
从信息的角度理解：看上去是异常值的数据其实也包含着一定的信息，不能轻易删去。
从算法的角度理解：虽说异常值对于树模型的影响较小，但
1、若是该异常值在树中没有单独一个分枝，而是与较大的正常值一起划分到一个结点的话
（即二者对应的y值相差不大），就会影响到这些正常的值和自己的精度了。
2、若直接删除这些异常值，那么在测试集中这些实际包含着某些信息的异常数据所对应的y值
就会被所谓相对接近异常值的“正常值”对应y值所替代。
原比赛第一的博主的做法是保留这些拖尾的值，避免了情况2；但我认为可以再进行一个分箱操作，避免情况1的

**对原始特征进行处理、构造、选取出优质特征**
首先，分别对于一些离散特征和连续特征分别进行分箱与编码处理

然后由于原始特征不多因此第一轮我暴力生成了上千个特征
主要操作包括对连续型数据（年龄、网龄、消费金额、APP使用次数等）的分箱，提炼一些连续型数据为0的单独标记做新特征，对bool型数据做两两组合、三三组合操作，求与、求或生成新的特征。
首先筛除大量无用特征（离散变量用方差法），然后通过决策树对特征价值排序选出部分有用信息（后面对结果再用XGB筛选一遍，否则耗时），这么做的目的是因为，我认为有一些隐藏的业务特征是可以通过各个原始特征之间的组合变换得到的，但是有时候这些特征的意义是不好理解的，所以就用了暴力法。然后再利用业务知识尝试构建了一些特征。

后面我又尝试将原始的具有偏态的特征取对数让它们更加稳定，然后再一次用暴力特征，结果得到的特征价值反而没有未取对数之前高了，取舍之下，我放弃了取对数。

**最后利用XGBoost模型对数据进行预测，最后成绩0.06400656**
