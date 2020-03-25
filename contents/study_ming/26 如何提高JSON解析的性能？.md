> 在学习戴铭老师的《iOS开发高手课》，这里是学习笔记。
> 
> 课程链接： [《iOS开发高手课》](https://time.geekbang.org/column/intro/161?code=PbktFs%2Fw7EHB9TJpCcw1bc9KoCR%2FYLnpUmqrB0uOruk%3D)
> 
> 戴铭老师的 GitHub：[ming1016](https://github.com/ming1016)

### 26 如何提高JSON解析的性能？

#### JSON

##### JSON 基于两种结构:

- 名字/值对集合:这种结构在其他编程语言里被实现为对象、字典、Hash 表、结构体或者关联数组。
- 有序值列表:这种结构在其他编程语言里被实现为数组、向量、列表或序列。

#### JSONDecoder 如何解析 JSON

JSONDecoder 的代码，可以在 [Swift 的官方 GitHub](https://github.com/apple/swift/blob/master/stdlib/public/Darwin/Foundation/JSONEncoder.swift) 上查看。


解析 JSON 的入口， JSONDecoder 的 decode 方法。下面是 decode 方法的定义代
码:

```
    /// Decodes a top-level value of the given type from the given JSON representation.
    ///
    /// - parameter type: The type of the value to decode.
    /// - parameter data: The data to decode from.
    /// - returns: A value of the requested type.
    /// - throws: `DecodingError.dataCorrupted` if values requested from the payload are corrupted, or if the given data is not valid JSON.
    /// - throws: An error if any value throws an error during decoding.
    open func decode<T : Decodable>(_ type: T.Type, from data: Data) throws -> T {
        let topLevel: Any
        do {
           topLevel = try JSONSerialization.jsonObject(with: data)
        } catch {
            throw DecodingError.dataCorrupted(DecodingError.Context(codingPath: [], debugDescription: "The given data was not valid JSON.", underlyingError: error))
        }

        let decoder = __JSONDecoder(referencing: topLevel, options: self.options)
        guard let value = try decoder.unbox(topLevel, as: type) else {
            throw DecodingError.valueNotFound(type, DecodingError.Context(codingPath: [], debugDescription: "The given data did not contain a top-level value."))
        }

        return value
    }
```

decode 方法在解析完后会将解析到的数据保存到传入的结构体中，然后返回。在 decode 方法里可以看到，对于传入的 Data 数据会首先通过 JSONSerialization 方法转化成 topLevel 原生对象，然后 topLevel 原生对象通过 JSONDecoder 初始化成一个 JSONDecoder 对象，最后使用 JSONDecoder 的 unbox 方法将数据和传入的结构体对应上，并保存在结构体里进行返回。

可以看出，目前 JSONSerialization 已经能够很好地解析 JSON，JSONDecoder 将其包装以后，通过 unbox 方法使得 JSON 解析后能很方便地匹配 JSON 数据结构和 Swift 原生结构体。

#### 提高 JSON 解析性能

[simdjson](https://github.com/simdjson/simdjson) 是一款发布于 2019 年 2 月的 JSON 解析器，号称每秒可解析千兆字节 JSON 文件。

simdjson 和其他 JSON 解析器对比如下图所示:

![](https://github.com/liuzhongning/Articles/blob/master/resources/study_ming/study_ming_simdjson.jpg)

simdjson 解析 JSON 的两个阶段：
 
第一个阶段，使用 simdjson 去发现需要 JSON 里重要的字符集，比如大括号、中括号、逗号、冒号等，还 有类似 true、false、null、数字这样的原子字符集。第一个阶段是没有分支处理的，这个阶段与词法分析非 常类似。

第二个阶段，simdjson 也没有做分支处理，而是采用的堆栈结构，嵌套关系使用 goto 的方式进行导航。 simdjson 通过索引可以处理所有输入的 JSON 内容而无需使用分支，这都归功于聪明的条件移动操作，使 得遍历过程变得高效了很多。

通过 simdjson 解析 JSON 的两个阶段可以看出，simdjson 的主要思路是尽可能地以最高效的方式将 JSON 这种可读性高的数据格式转换为计算机能更快理解的数据格式。如果你想更详细地了解这两个阶段的解析思路，可以查看这篇论文“[Parsing Gigabytes of JSON per Second](https://arxiv.org/abs/1902.08318)”。

而如果你想要在工程中使用 simdjson的话，直接使用它提供的一个简单接口即可。具体的使用代码如下:

```
#include "simdjson/jsonparser.h"
  /...
const char * filename = ... // JSON 文件
std::string_view p = get_corpus(filename);
ParsedJson pj = build_parsed_json(p); // 解析方法
// you no longer need p at this point, can do aligned_free((void*)p.data()) if( ! pj.isValid() ) {
// 出错处理 }
  aligned_free((void*)p.data());
```

---

关于 simdjson，我个人翻看资料后觉得，如果项目里用到了很多很大的 JSON，那么可以使用 simdjson；如果没有则没必要使用 simdjson 了，直接使用系统的解析即可。

想对 simdjson 有更多了解可以点击下面链接：

- [simdjson github](https://github.com/simdjson/simdjson)
- [Parsing Gigabytes of JSON per Second](https://arxiv.org/abs/1902.08318)
- [为什么simdjson这么快?](https://www.zhihu.com/question/313943804/answer/611131982)