# Caffe 源码梳理  
几个原则:  
- 从源头到结束, 按照流程来熟悉各个组件;   
- 抽离其中用到的数学库和其他开源库, 以便将来用到自己的项目中;  
- 熟悉 cpp-python 的接口调用方式;  

搞清楚流程之后再深入细节.    
[0. protobuf 的使用](./doc/protobuf.md)   
[1. Caffe 的入口](./doc/caffe_main.md)   
[2. Caffe 中的矩阵计算](./doc/math_functions.md)  
[3. solver 求解器的实现](./doc/solver.md)   
[4. synced_memory 同步内存的实现](./doc/synced_memory.md)   
[5. Blob 的实现](./doc/blob.md)   