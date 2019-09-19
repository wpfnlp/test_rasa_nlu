#### Data Formats
您可以提供Markdown或JSON格式的训练数据，作为单个文件或包含多个文件的目录。请注意，Markdown通常更容易使用。

#### 1. Markdown Format
Markdown是易于人们读写的最简单的Rasa NLU格式。使用无序列表语法列出示例，例如，减号 `-` ，星号`*`或加号`+`。示例按意图(intent)分组，实体(entities)注释为Markdown链接形式，例如，` [entity]（entity name）`。
```markdown
## intent:check_balance
- what is my balance <!-- no entity -->
- how much do I have on my [savings](source_account) <!-- entity "source_account" has value "savings" -->
- how much do I have on my [savings account](source_account:savings) <!-- synonyms, method 1-->
- Could I pay in [yen](currency)?  <!-- entity matched by lookup table -->

## intent:greet
- hey
- hello

<!-- synonyms, method 2 -->
## synonym:savings   
- pink pig

## regex:zipcode
- [0-9]{5}

<!-- lookup table list -->
## lookup:currencies   
- Yen
- USD
- Euro

<!-- no list to specify lookup table file -->
## lookup:additional_currencies  
path/to/currencies.txt
```
Rasa NLU的训练数据分为不同部分：
- common examples
- synonyms
- regex features and
- lookup tables

虽然`common examples`是强制性的唯一部分，包括其他部分将帮助NLU模型以更少的示例学习域，并且还有助于它对其预测更有信心。

`Synonyms`将提取的实体映射到相同的名称，例如将“my savings account”映射到简单的“saving”。但是，这仅在**提取实体后**才会发生，因此您需要提供包含同义词的示例，以便Rasa可以学习如何选择它们。

`Lookup tables`可以直接指定为列表，也可以指定为包含换行符分隔的单词或短语的txt文件。加载训练数据后，这些文件将用于生成添加到正则表达式功能的不区分大小写的正则表达式模式。例如，在这种情况下，提供货币名称列表，以便更容易选择此实体。

#### 2. JSON Format
JSON格式由名为`rasa_nlu_data`的顶层对象组成，其中包含`common_examples`，`entity_synonyms`和`regex_features`。最重要的是`common_examples`。

```
{
    "rasa_nlu_data": {
        "common_examples": [],
        "regex_features" : [],
        "lookup_tables"  : [],
        "entity_synonyms": []
    }
}
```
`common_examples`用于训练您的模型。您应该将所有训练示例放在`common_examples`数组中。`Regex features(正则表达式)`功能是一种帮助分类器检测实体或意图并提高性能的工具。

#### Improving Intent Classification and Entity Recognition

#### 1. Common Examples
`Common examples`包含三个组件：`text`，`intent`和`entities`。前两个是字符串，而最后一个是数组。

- `text`是用户消息[必填]
- `intent`与文本相关的意图[可选]
- `entities`需要识别的文本的特定部分[可选]

使用`start`和`end`指定实体，这些实体一起使python类型范围应用于字符串，例如，在下面的例子中，`text =“show me chinese restaurants”`，然后是`text [8:15] =='chinese'`。实体可以跨越多个单词，实际上，`value`字段不必与示例中的子字符串完全对应。这样，您可以将**同义词**或**拼写错误**映射到相同的`value`。
```markdown
## intent:restaurant_search
- show me [chinese](cuisine) restaurants
```

#### 2. Regular Expression Features
`Regular expressions`可用于支持意图分类和实体提取。例如，如果您的实体具有确定性结构（如邮政编码或电子邮件地址），则可以使用正则表达式来轻松检测该实体。对于zipcode示例，它可能如下所示：
```markdown
## regex:zipcode
- [0-9]{5}

## regex:greet
- hey[^\\s]*
```

上述名称不定义实体也不定义意图，它只是一个人们可读的描述，让您记住这个正则表达式的用途，并且是相应模式特征的标题。正如您在上面的示例中所看到的，您还可以使用正则表达式功能来提高意图分类性能。

尝试以尽可能少的单词匹配的方式创建正则表达式。例如。使用`hey [^ \ s] *`而不是`hey。*`，因为后者可能匹配整个消息，而第一个只匹配一个单词。

用于实体提取的正则表达式功能目前仅由`CRFEntityExtractor`组件支持！因此，其他实体提取器（如`MitieEntityExtractor`或`SpacyEntityExtractor`）将不使用生成的功能，并且它们的存在不会改善这些提取器的实体识别。目前，所有意图分类器都使用可用的正则表达式功能。

#### 3. Lookup Tables
也可以在训练数据中指定外部文件或元素列表形式的`Lookup tables(查找表)`。外部提供的查找表必须采用换行符分隔格式。例如，`data / test / lookup_tables / plates.txt`可能包含：
```
tacos
beef
mapo tofu
burrito
lettuce wrap
```
并且可以加载为：
```markdown
## lookup:plates
data/test/lookup_tables/plates.txt
```
或者，可以将查找元素直接包括为无序列表
```markdown
## lookup:plates
- beans
- rice
- tacos
- cheese
```
当在训练数据中提供查找表时，内容被组合成一个大的，不区分大小写的正则表达式模式，该模式在训练示例中查找完全匹配。这些正则表达式匹配多个令牌，因此`lettuce wrap`将匹配`get me a lettuce wrap ASAP`得到[0 0 0 1 1 0]。这些正则表达式与训练数据中直接指定的常规正则表达式模式处理相同。

#### Normalizing Data
#### 1. Entity Synonyms
如果将实体定义为具有相同的值，则将它们视为同义词。这是一个例子：
```markdown
## intent:search
- in the center of [NYC](city:New York City)
- in the centre of [New York City](city)
```
如您所见，实体`city`在两个示例中都具有`New York City`值，即使第一个示例中的文本指出`NYC`。通过将value属性定义为与实体的开始和结束索引之间的文本中找到的值不同，您可以定义同义词。每当找到相同的文本时，该值将使用同义词而不是消息中的实际文本。

要使用训练数据中定义的同义词，您需要确保管道包含`EntitySynonymMapper`组件（请参阅 [`Components`](https://rasa.com/docs/rasa/nlu/components/#components)
）。

或者，您可以添加“entity_synonyms”数组以定义一个实体值的多个同义词。这是一个例子：
```markdown
## synonym:New York City
- NYC
- nyc
- the big apple
```

#### 使用工具生成更多实体示例
生成一堆实体示例有时很有用，例如，如果您有一个餐馆名称数据库。社区建立了一些工具来帮助解决这个问题。

您可以使用[`Chatito`](https://rodrigopivi.github.io/Chatito/)，这是一种使用简单的DSL或 [`Tracy`](https://yuukanoo.github.io/tracy)生成rasa格式的训练数据集的工具，这是一个简单的GUI，用于为rasa创建训练数据集。

但是，创建合成示例通常会导致过度拟合，如果您拥有大量实体值，则最好使用`Lookup Tables(查找表)`。

#### 部分功能测试
1. 初始化一个新工程。
```Markdown
rasa init
```
2. nlu.md数据文件中增加数据
```Markdown
## intent:check_balance
- what is my balance <!-- no entity -->
- how much do I have on my [pink pig](source_account)
- how much do I have on my [savings](source_account) <!-- entity "source_account" has value "savings" -->
- how much do I have on my [savings account](source_account:savings) <!-- synonyms, method 1-->
- Could I pay in [yen](currency)?  <!-- entity matched by lookup table -->
- The U.S. zip code is [51000](zipcode)
- [87233](zipcode) is my home zip code

## synonym:savings   <!-- synonyms, method 2 -->
- pink pig

## regex:zipcode  <!-- My home zip code is 87666 注释掉正则表达式，当前样例无法被识别出实体。说明正则表达式的特征可以增强实体识别功能 -->
- [0-9]{5}

## lookup:currencies   <!-- lookup table list -->
- Yen
- USD
- Euro
```
3. 单独训练NLU
```
rasa train nlu
```

4. 命令行运行
```
rasa shell nlu
```

#### 测试近义词功能
1. 输入`how much do I have on my pink pig`。
```
Next message:
how much do I have on my pink pig 
{
  "intent": {
    "name": "check_balance",
    "confidence": 0.5444962824909438
  },
  "entities": [
    {
      "start": 25,
      "end": 33,
      "value": "savings",
      "entity": "source_account",
      "confidence": 0.804806426342895,
      "extractor": "CRFEntityExtractor",
      "processors": [
        "EntitySynonymMapper"
      ]
    }
  ],
  "intent_ranking": [
    {
      "name": "check_balance",
      "confidence": 0.5444962824909438
    },
    {
      "name": "greet",
      "confidence": 0.09999720963836044
    },
    {
      "name": "deny",
      "confidence": 0.09487487439750099
    },
    {
      "name": "affirm",
      "confidence": 0.07553684292916384
    },
    {
      "name": "goodbye",
      "confidence": 0.07541183096346527
    },
    {
      "name": "mood_great",
      "confidence": 0.07020477061265415
    },
    {
      "name": "mood_unhappy",
      "confidence": 0.03947818896791153
    }
  ],
  "text": "how much do I have on my pink pig"
}
```
#### 测试正则表达式功能
1. 输入`My home zip code is 87666`。
```
Next Message:
My home zip code is 87666
{
  "intent": {
    "name": "check_balance",
    "confidence": 0.5372258767193705
  },
  "entities": [
    {
      "start": 20,
      "end": 25,
      "value": "87666",
      "entity": "zipcode",
      "confidence": 0.5015339564584156,
      "extractor": "CRFEntityExtractor"
    }
  ],
  "intent_ranking": [
    {
      "name": "check_balance",
      "confidence": 0.5372258767193705
    },
    {
      "name": "deny",
      "confidence": 0.1271869917021482
    },
    {
      "name": "greet",
      "confidence": 0.12010681112164015
    },
    {
      "name": "affirm",
      "confidence": 0.07854821827684225
    },
    {
      "name": "mood_great",
      "confidence": 0.05469110380348843
    },
    {
      "name": "goodbye",
      "confidence": 0.052225971507388995
    },
    {
      "name": "mood_unhappy",
      "confidence": 0.03001502686912203
    }
  ],
  "text": "My home zip code is 87666"
}
```
2. 注释掉正则表达式定义后训练模型并运行。同样输入`My home zip code is 87666`。
```
Next message:
My home zip code is 87666
{
  "intent": {
    "name": "check_balance",
    "confidence": 0.473317332154941
  },
  "entities": [],
  "intent_ranking": [
    {
      "name": "check_balance",
      "confidence": 0.473317332154941
    },
    {
      "name": "deny",
      "confidence": 0.14858894989885554
    },
    {
      "name": "greet",
      "confidence": 0.1252977708910065
    },
    {
      "name": "affirm",
      "confidence": 0.11399173106726823
    },
    {
      "name": "goodbye",
      "confidence": 0.05819426376682528
    },
    {
      "name": "mood_great",
      "confidence": 0.05102853472334429
    },
    {
      "name": "mood_unhappy",
      "confidence": 0.02958141749775906
    }
  ],
  "text": "My home zip code is 87666"
}
```

总结：对比可以看出`正则表达式`对实体的提取有增强作用。

3. Lookup Tables（查找表）
