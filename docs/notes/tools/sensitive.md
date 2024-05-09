# 敏感词过滤
工作中C端的业务一般都会遇到敏感词过滤 

sensitive-word 基于 DFA 算法实现的高性能敏感词工具。

## 项目地址

github: https://github.com/houbb/sensitive-word

gitee: https://gitee.com/hywang2017/sensitive-word


## JAVA使用
###  Maven 引入

```xml
<dependency>
    <groupId>com.github.houbb</groupId>
    <artifactId>sensitive-word</artifactId>
    <version>0.12.0</version>
</dependency>
```

### 核心方法

`SensitiveWordHelper` 作为敏感词的工具类，核心方法如下：

| 方法                                     | 参数                       | 返回值    | 说明           |
|:---------------------------------------|:-------------------------|:-------|:-------------|
| contains(String)                       | 待验证的字符串                  | 布尔值    | 验证字符串是否包含敏感词 |
| replace(String, ISensitiveWordReplace) | 使用指定的替换策略替换敏感词           | 字符串    | 返回脱敏后的字符串    |
| replace(String, char)                  | 使用指定的 char 替换敏感词         | 字符串    | 返回脱敏后的字符串    |
| replace(String)                        | 使用 `*` 替换敏感词             | 字符串    | 返回脱敏后的字符串    |
| findAll(String)                        | 待验证的字符串                  | 字符串列表  | 返回字符串中所有敏感词  |
| findFirst(String)                      | 待验证的字符串                  | 字符串    | 返回字符串中第一个敏感词 |
| findAll(String, IWordResultHandler)    | IWordResultHandler 结果处理类 | 字符串列表  | 返回字符串中所有敏感词  |
| findFirst(String, IWordResultHandler)  | IWordResultHandler 结果处理类 | 字符串    | 返回字符串中第一个敏感词 |
| tags(String)       | 获取敏感词的标签                 | 敏感词字符串 | 返回敏感词的标签列表   |

 
#### 常见用法
```java 
public class SensitiveWordTest{

    public static void main(String[] args) {
        //判断是否包含敏感词
        final String text = "五星红旗迎风飘扬，毛主席的画像屹立在天安门前。"; 
        Assert.assertTrue(SensitiveWordHelper.contains(text));
        
        //返回第一个敏感词
        final String text = "五星红旗迎风飘扬，毛主席的画像屹立在天安门前。";

        String word = SensitiveWordHelper.findFirst(text);
        Assert.assertEquals("五星红旗", word);
        //SensitiveWordHelper.findFirst(text) 等价于：
        String word = SensitiveWordHelper.findFirst(text, WordResultHandlers.word());
        
        //WordResultHandlers.raw() 可以保留对应的下标信息：
        final String text = "五星红旗迎风飘扬，毛主席的画像屹立在天安门前。"; 
        IWordResult word = SensitiveWordHelper.findFirst(text, WordResultHandlers.raw());
        Assert.assertEquals("WordResult{startIndex=0, endIndex=4}", word.toString());
        
        //返回所有敏感词
        final String text = "五星红旗迎风飘扬，毛主席的画像屹立在天安门前。"; 
        List<String> wordList = SensitiveWordHelper.findAll(text);
        Assert.assertEquals("[五星红旗, 毛主席, 天安门]", wordList.toString());

        //返回所有敏感词用法上类似于 SensitiveWordHelper.findFirst()，同样也支持指定结果处理类。
        //SensitiveWordHelper.findAll(text) 等价于：
        List<String> wordList = SensitiveWordHelper.findAll(text, WordResultHandlers.word());
        
        //WordResultHandlers.raw() 可以保留对应的下标信息：
        final String text = "五星红旗迎风飘扬，毛主席的画像屹立在天安门前。"; 
        List<IWordResult> wordList = SensitiveWordHelper.findAll(text, WordResultHandlers.raw());
        Assert.assertEquals("[WordResult{startIndex=0, endIndex=4}, WordResult{startIndex=9, endIndex=12}, WordResult{startIndex=18, endIndex=21}]", wordList.toString());

         //默认的替换策略
        final String text = "五星红旗迎风飘扬，毛主席的画像屹立在天安门前。";
        String result = SensitiveWordHelper.replace(text);
        Assert.assertEquals("****迎风飘扬，***的画像屹立在***前。", result);

        //指定替换的内容
        final String text = "五星红旗迎风飘扬，毛主席的画像屹立在天安门前。";
        String result = SensitiveWordHelper.replace(text, '0');
        Assert.assertEquals("0000迎风飘扬，000的画像屹立在000前。", result);
    }
}

``` 
  
#### 自定义替换策略

V0.2.0 支持该特性。

场景说明：有时候我们希望不同的敏感词有不同的替换结果。比如【游戏】替换为【电子竞技】，【失业】替换为【灵活就业】。

诚然，提前使用字符串的正则替换也可以，不过性能一般。

使用例子：

```java
public class SensitiveWordTest {
    /**
     * 自定替换策略
     * @since 0.2.0
     */
    @Test
    public void defineReplaceTest() {
        final String text = "五星红旗迎风飘扬，毛主席的画像屹立在天安门前。";

        ISensitiveWordReplace replace = new MySensitiveWordReplace();
        String result = SensitiveWordHelper.replace(text, replace);

        Assert.assertEquals("国家旗帜迎风飘扬，教员的画像屹立在***前。", result);
    }
}
```

其中 `MySensitiveWordReplace` 是我们自定义的替换策略，实现如下：

```java
public class MyWordReplace implements IWordReplace {

    @Override
    public void replace(StringBuilder stringBuilder, final char[] rawChars, IWordResult wordResult, IWordContext wordContext) {
        String sensitiveWord = InnerWordCharUtils.getString(rawChars, wordResult);
        // 自定义不同的敏感词替换策略，可以从数据库等地方读取
        if("五星红旗".equals(sensitiveWord)) {
            stringBuilder.append("国家旗帜");
        } else if("毛主席".equals(sensitiveWord)) {
            stringBuilder.append("教员");
        } else {
            // 其他默认使用 * 代替
            int wordLength = wordResult.endIndex() - wordResult.startIndex();
            for(int i = 0; i < wordLength; i++) {
                stringBuilder.append('*');
            }
        }
    }

}
```

我们针对其中的部分词做固定映射处理，其他的默认转换为 `*`。

## IWordResultHandler 结果处理类

IWordResultHandler 可以对敏感词的结果进行处理，允许用户自定义。

内置实现见 `WordResultHandlers` 工具类：

- WordResultHandlers.word()

只保留敏感词单词本身。

- WordResultHandlers.raw()

保留敏感词相关信息，包含敏感词的开始和结束下标。

- WordResultHandlers.wordTags()

同时保留单词，和对应的词标签信息。
