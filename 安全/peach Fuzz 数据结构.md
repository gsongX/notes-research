---
title: peach Fuzz 数据结构
tags: yunpiao
date: 2017-04-24 16:08:16
---

> 开源Fuzz框架。


Michael Eddington等人开发的Peach是一个遵守MIT开源许可证的模糊测试框架，最初采用Python语言编写，发布于2004年，第二版于2007年发布，最新的第三版使用C#重写了整个框架。

Peach支持对文件格式、ActiveX、网络协议、API等进行Fuzz测试；Peach Fuzz的关键是编写Peach Pit配置文件。

Windows下使用Peach3需要预先安装.net 4和windbg；Linux、OS X下需要安装Mono .net开发框架。

<!--more-->
## peach配置文件
    peach中最重要一部分就是peach Pit配置文件。Peach Pit文件包含以下内容：

    1、General Configuration(通用配置）
    2、Data Modeling（数据模型）
    3、State Modeling（状态模型）
    4、Agents and Monitors（代理和监视）	
    5、Test Configuration（测试配置）
具体文件结构如下：

    <?xml 版本，编码之类?>
    <Peach 创建时间，地址，作者等等>
    <Include 包含外部文件 />
    <DataModel> 类型信息，关系（大小，计数，偏移）、可嵌套等<\DataMode>
    <StateModel>测试逻辑，状态转换</StateModel>
    <Agent>监视被测目标的情况，崩溃信息等</Agent>
    <Test>指定使用哪个StateModel，Agent，Publisher、Strategy、Logger等</Test>
    </Peach>
	
## 配置项之一 数据模型（data modeling）
	DataModel是Peach根元素的子元素之一，它通过添加子元素（比如Number、Blob或者String）的方式定义了数据块的结构。
	数据模型的构建关系到模糊测试的关键。

### 1 Data Model

（1）属性：
- Name---必须的。当引用模型或者调试时，友好的DataModel名字是非常有用的。
- Ref---可选的。引用一个DataModel模板。
- Mutable---可选的，默认为真。该元素是否可变异。
- Constraint---可选的。确定一个表达式，它帮助Peach确定数据元素是否已被适当的消耗。

（2）子元素：
 - Block、
 - Choice、
 - Custom、
 - Flag、
 - Flags、
 - Number、
 - Padding、
 - String、
 - XmlAttribute、
 - XmlElement、
 - Relation、
 - Fixup、
 - Transformer、
 - Placement

（3）例子：

![DataModel例子][1]

![引用冲突例子][2]

当一个DataModel被解析时，自定义DataModel看起来像是两个数据模型的组合，如下所示：

![冲突时真实解析格式][3]
一个Peach文件中可以指定任意多个DataModel元素，但每个DataModel的名字必须唯一。通过DataModel可以将复杂的格式按照逻辑分解为更小的模型，使数据模型更易阅读、调试和重用。一个名字为“HelloWorld”的DataModel包含一个字符串和输出“Hello World！”如下所示：




### 2、 Blob
Blob元素是DataModel或Block的一个子元素。Blob元素常常用于代表缺少类型定义或格式的数据。如下所示：

#### （1）属性（除非声明，所有的属性都是可选的）：
- Name---必须的。Blob的名字。
- Value---含有Blob的默认值。
- Length---Blob的大小，单位为字节。
- Ref---引用一个数据模型来作为Blob的模板。
- valueType---默认格式的值，hex，string，或者literal，默认为string。
- minOccurs---该Blob元素必须发生变化的最小次数，默认为1。
- maxOccurs---该Blob元素能够发生变化的最大次数，默认为1。
- Token---当解析时该元素应该作为一个令牌来信任，默认是假。
- lengthType---长度的类型，指定长度。
- Constraint---一个约束的形式表达，用于数据破解。
- Mutable---Blob元素是否可变异（是否能被fuzzing），默认为真。
#### （2）子元素：
- Anayzers

#### （3）例子：
一个简单的Blob。这个Blob中，任何类型或长度的数据能破解。
![任意长度的数据][4]

### 3、  Block
与DataModel 类似
Block是DataModel或Block元素的子元素。Block用于在一个逻辑结构中将一个或者多个数据元素（Number或String）组织在一起，它和Datamodel非常相似，仅有的差异是它们的位置。DataModel是个顶层元素，Block是DataModel的一个子元素，它们都可以作为其他Block或DataModel的模板。

#### （1）属性（除非声明，所有的属性都是可选的）：
- Name---Block的名字。
- Ref---引用一个数据模型来作为Block的模板。。
- minOccurs---该Block必须发生变化的最小次数。
- maxOccurs--该Block可以发生变化的最大次数。
- Mutable---元素是否可变异，默认为真。
#### （2）子元素：
Blob、 block、 Choice、 Custom、 Fixup、 Flag、 Flags、 Number、 Padding、Placement、 Relation、Seek、 String、 Transformer、 XmlAttribute、 XmlElement。

#### （3）例子：

嵌套的Block。Block可以根据需要多层的嵌套，它可以帮助创建逻辑结构而不改变数据包含的内容。
![Block嵌套][5]
这个嵌套的Block定义产生的输出为：1 2 3 4 。


引用属性允许构建健壮的模板，如下所示模板的名字为“Key”，值为“\r\n”。
![enter description here][6]
使用该模板作为一个引用
![enter description here][7]
输出为：
![enter description here][8]

两个关键字符串在这里发生冲突。当解析时，自定义的Block将代替它的DataModel模板的结构。添加字符串值“：\r\n”。同时“customized”将覆盖String元素的“Key”和“Value”的值，用“Content-Length”和55代替。最终的DataModel将被解析如下：
![enter description here][9]

### 4、 Choice
Choice是DataModel或者Block元素的的子元素之一。Choice元素用于指示任何子元素是有效的，但是只应选择一个，很像编程语言中的switch语句。

#### （1）属性（除非声明，所有的属性都是可选的）：
- Name---choice元素的名字。
- minOccurs---该Choice必须发生改变的最小次数。
- maxOccurs---该Choice能发生改变的最大次数。
- Occurs---该choice能发生改变的迭代次数。
#### （2）子元素：
Block、Choice、String、Number、Blob、Flags、Fixup、Transformer、XmlAttribute、XmlElement。
#### （3）例子：
一个基本的Choice。这个例子将破解或消耗1,2,3类型的数据，很像一个需要在令牌上做出决定的常规切换语句。它的前8个字节是1，剩下的数据被视为一个32位数字。如果前8位是2，剩下的数据被视为一个255字节的二进制数据。如果前8位是3，剩下的数据被视为一个8字节字符串。当fuzzing时，Peach将选择其中的1个类型并进行fuzzing，它的输出为一个8位数字，后跟相应的类型。Peach将会尝试所有的3个类型。
![enter description here][10]
一系列的Choice。第一个例子适合构建单个Choice，但如果有许多Type1 、Type2和Type3块都是彼此跟随的，该怎么做呢？。通过设置minoccurs、maxoccurs或者occurs属性，可以指定Choice应该被重复。这个例子尝试来破解至少3个，最多6不同的Choice。
![enter description here][11]

### 5、 Flags
Flags定义了一组Flag的大小。
#### （1）属性：
- Name---可选的。元素的名字。
- Size---必须的。大小，以位为单位。
- Mutable---可选的。元素是否可以变异，默认为真。
#### （2）子元素：
- Fixup、Flag、Placement、Relation、Transformer。
#### （３）例子
![enter description here][12]

### 6、 Number
该元素定义了长度为8,16,24,32，或64位长度的二进制数。它是DataModel、Block或者Choice的子元素。
#### （1）属性：
- Name---必须的。Number的名字。
- Size---必须的。Number的大小，以位为单位。有效值为1到64。
- Value---分配给Number的默认值。
- valueType---可选的。value的表现方式。有效选项为string（字符串）和hex（十进制）。
- Token---当解析的时候，该元素被视为一个令牌，默认值为假。有效选项为真和假。
- Endian---Number的字节顺序。默认为小端。有效选项为大端、小端和网络。网络一样是大端。
- Signed---Number是否为有符号数据。默认为真。有效选项为真和假。
- Constraint---一个以Python表达式为形式的约束。用于数据破解。
- Mutable---元素是否可改变（fuzzing时是否可变异），默认为真。有效选项为真和假。
- minOccurs---Number必须发生改变的最小次数，默认为1。有效选项为正整数值。
- maxOcuurs---Number能够发生改变的最大次数，没有默认值。有效选项为正整数值。
#### （2）有效子元素：
- Anayzers、Fixup、Relation、Transformer、Hint。
#### （3）例子：

有符号。为了表明这是一个无符号数据元素，设置signed属性等于“false”。默认为真。
![enter description here][13]

Value类型。值类型定义了怎么解释Value的属性。有效选项为string和hex，默认为string。将值1000分配给Hi5。
![enter description here][14]
将43981以十六进制形式分配给Hi5。
![enter description here][15]

小端。为了改变Number的字节顺序，请设置endian属性。
![enter description here][16]
    上图将产生如下字节顺序：AB CD。
![enter description here][17]
    上图将产生如下字节顺序：CD AB。
	
### 7、Padding
Padding元素用来填充大小变化的块或数据模型。
#### （1）属性：
- Name---必须的。Number元素的名字。
- Aligned---将父元素对齐到8字节边界，默认为假。
- Alignment---对齐到这个位边界，比如（8、16等），默认为8。
- alignedTo---基于我们要填充的元素名字。
- Lengthcalc---计算结果为整数的脚本表达式。
- Constraint---一个以Python表达式形式的约束。用于数据破解。
- Mutable---元素是否可变异，默认为真，有效选项为真和假。
#### （2）有效子元素：
- Fixup、Relation、Transformer、Hint。
#### （3）例子：
![enter description here][18]

### 8、 String
该元素定义了一个单字节或者双字节的字符串，它是DataModel或者Block的子元素。为了指定这是一个数值的字符串，请用 NumericalString元素。
     
#### （1）属性：
- Name---可选的，数据模型的名字。
- Length---可选的，字符串的长度。
- lengthType---可选的，Length属性的单位。
- Type---可选的。字符编码类型，默认为“ASCII”，有效选项为ASCII、utf7、utf8、utf6、utf6be、utf32。
- nullTerminated---可选的。是否为以空字节结尾的字符串（真或者假）。
- padCharacter---可选的。根据length参数填充字符串的字符，默认为（0x00）。
- Token---可选的。当解析的时候，该元素应该被视为一个令牌，默认为假。
- Constraint---一个脚本表达式形式的约束。用于数据破解。
- Mutable---可选的。元素是否可变异，默认为真。
- Minoccurs---可选的。这个块必须发生改变的最小次数，默认为1。
- Maxoccurs---可选的。这个块会发生改变的最大次数，默认为1。
#### （2）有效子元素：
Analyzer、Fixup、Relation、Transformer、Hint。
#### （3）NumericalString：
![enter description here][19]
该元素只能用于String来说明它的值是一个数字。当使用这个提示时，它激活所有的数字突变以及标准的字符串突变。请注意：如果默认情况下一个字符串的值是数字，NumericalString元素被自动添加。
![enter description here][20]
### 9、 Relation
Peach允许构建数据间的关系。关系是类似这样的东西“X是Y的大小”、“X是Y的数量”、或者“X是y的偏移（字节单位）”。
#### （1）大小关系：
![enter description here][21]

在这个例子中，我们将提供两个python表达式，它允许在获取或设置size属性的时候修改它的大小，有两个变量可用，分别为self和size。Self是Number元素的一个引用，size是一个整数。获取操作和设置操作应该是彼此的数学逆操作。在破解过程中应用获取操作，在发布过程中应用设置操作。
expressionGet---该表达式的结果用于内部，它确定名字为TheValue的String元素读取多少字节。如果Peach取10，它将在内部存储一个5，然后Peach将读取5个字节到String中。
ExpressionSet---为 publisher生成一个值。 在以下示例中，为TheValue存储的Size的值“5”（TheValue的长度），因此Peach通过publisher输出的值将为“5 * 2”或10。
![enter description here][22]
#### （2）数量关系：
在这个例子中，Number将会说明String列表的数目。
![enter description here][23]
在这个例子中，我们将提供两个python表达式，它允许在获取或设置size属性的时候修改它的大小。有两个变量可用，分别为self和count。Self是Number元素的一个引用，count是一个整数。这里的让count可用与前面的表达式不同。虽然self在表达式对中始终可用，但其他可用的变量的名字是Relation元素type属性的值。  
expressionGet---该表达式的结果用于内部，它确定String元素将扩展到多少项。maxOccurs 是Peach循环计算中遇到的最大值，由于maxOccurs = 1024的限制，Peach在CountIndicator元素中破解时遇到的最大值是2048。
ExpressionSet---设置要生成的值。 以下示例中，count根据读入的String元素数目确定。
![enter description here][24]

#### （3）偏移关系：
![enter description here][25]
##### 1、相对偏移量：
从Peach 2.3开始支持相对偏移的概念。 相对偏移来自附加到relation的数据元素。 请考虑以下示例。，当确定StringData的偏移量时，Peach将根据需要，对OffsetToString的位置进行加上/减去它的值，以确定正确的偏移量。
![enter description here][26]
##### 2、相对于偏移量：
Peach还支持相对于另一个元素的偏移。它用于以下情况，一个元素包含另外一个元素的偏移，而被包含的偏移的元素是一个结构体的开头。 以下示例中，StringData偏移量将通过添加OffsetToString的值到Structure位置的方式来计算。
![enter description here][27]
包含expressionGet / expressionSet
当使用带有偏移关系的expressionGet / Set时，将提供两个变量：self和offset。 Self引用了一个父元素的引用，offset是一个整数。
包含Placement的偏移关系
这个模型中，将使用一个典型的模式。在这个模式中，一个偏移列表会给我们另一个元素的位置。 使用Placement元素将创建的Data字符串移动到Chunks块之后。
![enter description here][28]

### 10 .Fixup
Fixup是一些代码函数，它通常操作另一个元素数据来产生一个值。校验和算法就是这样一个示例。Peach默认包含有以下fixups。
#### Utility Fixups
CopyValueFixup
SequenceIncrementFixup
SequenceRandomFixup
#### Check-sum Fixups
Crc32DualFixup
Crc32Fixup
EthernetChecksumFixup
ExpressionFixup
IcmpChecksumFixup
LRCFixup
#### Hashing Fixups
MD5Fixup
SHA1Fixup
SHA224Fixup
SHA256Fixup
SHA384Fixup
SHA512Fixup

### Transformers
Transformers在父元素上执行静态转换或编码。通常来说Transformers是两个方向：编码和解码，但也不全是。 比如ZIP压缩，Base64编码，HTML编码等。与Fixups不同，Transformers对父元素操作，而Fixups引用另一个元素的数据。
![enter description here][29]

上述数据模型的输出为0x01 <len(b64(Data))> <b64(Data)>
Peach 3中的默认Transformers：

####  Compression
Bz2CompressTransformer
Bz2DecompressTransformer
GzipCompressTransformer
GzipDecompressTransformer
#### Crypto
Aes128Transformer
ApacheMd5CryptTransformer
CryptTransformer
CvsScrambleTransformer
HMACTransformer
MD5Transformer
SHA1Transformer
TripleDesTransformer
UnixMd5CryptToolTransformer
UnixMd5CryptTransformer
#### Encode
Base64EncodeTransformer
Base64DecodeTransformer
HexTransformer
HexStringTransformer
HtmlEncodeTransformer
HtmlDecodeTransformer
HtmlEncodeAgressiveTransformer
Ipv4StringToOctetTransformer
Ipv4StringToNetworkOctetTransformer
Ipv6StringToOctetTransformer
JsEncodeTransformer
NetBiosEncodeTransformer
NetBiosDecodeTransformer
SidStringToBytesTransformer
UrlEncodeTransformer
UrlEncodePlusTransformer
Utf8Transformer
Utf16Transformer
Utf16LeTransformer
Utf16BeTransformer
WideCharTransformer
#### Type
AsInt8Transformer
AsInt16Transformer
AsInt24Transformer
AsInt32Transformer
AsInt64Transformer
IntToHexTransformer
NumberToStringTransformer
StringToFloatTransformer
StringToIntTransformer
Misc
EvalTransformer1


### 11 Placement
Placemend 元素告诉数据破解者， 在数据流被解析后， 特定元素应该被移动, 结合偏移关系是peach支持的文件处理形式，它通过偏移来包含元素的引用
![enter description here][30]
- 属性：
以下其中之一是必须的：
After---元素移动到之后。
Before---元素移动到之前。


  [1]: ./images/1493038838246.jpg "DataModel例子"
  [2]: ./images/1493038717603.jpg "引用冲突例子"
  [3]: ./images/1493038975493.jpg "冲突时真实解析格式"
  [4]: ./images/1493039197835.jpg "Blob例子"
  [5]: ./images/1493039409395.jpg "Block多重嵌套例子"
  [6]: ./images/1493039518872.jpg "构建Block模板"
  [7]: ./images/1493039523065.jpg "引用Block"
  [8]: ./images/1493039526399.jpg "输出"
  [9]: ./images/1493039609176.jpg "1493039609176"
  [10]: ./images/1493039851919.jpg "1493039851919"
  [11]: ./images/1493039859648.jpg "1493039859648"
  [12]: ./images/1493040374857.jpg "1493040374857"
  [13]: ./images/1493040571879.jpg "1493040571879"
  [14]: ./images/1493040675580.jpg "1493040675580"
  [15]: ./images/1493040679063.jpg "1493040679063"
  [16]: ./images/1493040684008.jpg "1493040684008"
  [17]: ./images/1493040690838.jpg "1493040690838"
  [18]: ./images/1493040786958.jpg "1493040786958"
  [19]: ./images/1493040968504.jpg "1493040968504"
  [20]: ./images/1493040874809.jpg "1493040874809"
  [21]: ./images/1493041068222.jpg "1493041068222"
  [22]: ./images/1493041094060.jpg "1493041094060"
  [23]: ./images/1493041352609.jpg "1493041352609"
  [24]: ./images/1493041360183.jpg "1493041360183"
  [25]: ./images/1493711366257.jpg
  [26]: ./images/1493711382014.jpg
  [27]: ./images/1493711385056.jpg
  [28]: ./images/1493711400975.jpg
  [29]: ./images/1493711562792.jpg
  [30]: ./images/1493711930260.jpg