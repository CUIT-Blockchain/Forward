## InterPlanetary Ecosystem

#### IPFS

IPFS可以理解为多个模块组装成的一个程序：

- IPLD，描述数据
- libp2p，传输数据
- 一些数据存储的具体实现
- features...

#### IPLD

`InterPlanetary Linked Data`，ipld定义了我们传输数据、链接数据的方式

IPLD是一系列规范，用来帮助开发者开发去中心化程序，而不是一个程序本身

> IPFS依赖IPLD，IPLD不依赖IPFS

#### libp2p

libp2p是一个处理网络和数据传输的项目

#### CIDs

`Content IDentifiers`，内容标识符，基于内容的加密哈希运算。

通常我们使用cid来进行IPLD的链接，即`linking`，它是不可篡改的。

#### Multiformats

定义数据格式

- multihash
- multicodecs
- multiaddrs
