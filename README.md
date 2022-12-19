![JavaRecord.png](https://pic6.58cdn.com.cn/nowater/webim/big/n_v26c143b7134fa421180c7828b493c3ef4.png)
<p align="center">
  <a href="#"><img src="https://img.shields.io/badge/Author-BookSea-orange.svg" alt="作者"></a>
  <a href="#公众号"><img src="https://img.shields.io/badge/%E5%85%AC%E4%BC%97%E5%8F%B7-Java随想录-lightgrey.svg" alt="公众号"></a>
  <a href="https://blog.csdn.net/bookssea"><img src="https://img.shields.io/badge/csdn-CSDN-red.svg" alt="投稿"></a>
  <a href="https://juejin.cn/user/2837192913204935"><img src="https://img.shields.io/badge/juejin-掘金-blue.svg" alt="公众号"></a>
  <a href="https://www.cnblogs.com/booksea/"><img src="https://img.shields.io/badge/cnblogs-博客园-important.svg" alt="投稿"></a>
</p>


**文章每周至少一更，首发公众号**。如果有帮助到大家，希望点个**Star**！让我有持续的动力，感谢🤝</br>

最近更新文章：[JVM篇 第6节：CMS意志的继承者—G1](https://mp.weixin.qq.com/s?__biz=Mzg4Nzc3NjkzOA==&mid=2247483838&idx=1&sn=65a9a2a0c77a46cd4de9ce96abdea149&chksm=cf84727bf8f3fb6d06b9c975bbf76bbea2bd5d8bf5337d27ec3edf71fa2bc6d68e4f20ae6058#rd)        — `更新时间：2022/12/16`</br>

由于最近在看《深入理解Java虚拟机 第3版本》这本书，所以开篇先从JVM篇开始

- :memo: 目录

   - 第1章：JVM
       - [第1节：根节点枚举与安全点](https://mp.weixin.qq.com/s?__biz=Mzg4Nzc3NjkzOA==&mid=2247483723&idx=1&sn=832533651b58f6c1725ca0e6ec5ba7b8&chksm=cf84728ef8f3fb981ef04f316974737457ce0b23909cb7407d00469af6776839c4a759fdbe7a#rd)
       - [第2节：记忆集与卡表](https://mp.weixin.qq.com/s?__biz=Mzg4Nzc3NjkzOA==&mid=2247483830&idx=1&sn=5d886e14a5a0d06f8bd61e6b99a4fe58&chksm=cf847273f8f3fb65a1a81dd38e54ad3393c7bada3161a71d5189436ba62e71e69dc403cd2d8a#rd)
       - [第3节：三色标记算法](https://mp.weixin.qq.com/s?__biz=Mzg4Nzc3NjkzOA==&mid=2247483832&idx=1&sn=db8168382d463b71a74983b9e9756d48&chksm=cf84727df8f3fb6b49613d1751f230ec55c86c1058dcc605ba007799a41dc2a2d1101ddfafea#rd)
       - [第4节：经典垃圾回收器](https://mp.weixin.qq.com/s?__biz=Mzg4Nzc3NjkzOA==&mid=2247483834&idx=1&sn=880998e6ba6295e4e20a7fdbbcbb8b33&chksm=cf84727ff8f3fb69f9dd7ad74e21162c248a81bf2e91879d938e0c4f877defa4cb5fe2bcd1be#rd)
       - [第5节：伟大的开端—CMS](https://mp.weixin.qq.com/s?__biz=Mzg4Nzc3NjkzOA==&mid=2247483836&idx=1&sn=c6ae10ec16de85421e9b9f728c0a4a21&chksm=cf847279f8f3fb6ffaaec103f6e1742bf7d55084b0e0aff7131ca7090c9ef43a4840203eeebd#rd)
       - [第6节：CMS意志的继承者—G1](https://mp.weixin.qq.com/s?__biz=Mzg4Nzc3NjkzOA==&mid=2247483838&idx=1&sn=65a9a2a0c77a46cd4de9ce96abdea149&chksm=cf84727bf8f3fb6d06b9c975bbf76bbea2bd5d8bf5337d27ec3edf71fa2bc6d68e4f20ae6058#rd)
       
   - 第2章：MySQL

# 关注我，我们一起交流技术

  <a name="微信"></a>  <a name="公众号"></a>
![公众号.jpg](https://pic8.58cdn.com.cn/nowater/webim/big/n_v21e163f266a554146b0a791b5d35839b3.jpg)
