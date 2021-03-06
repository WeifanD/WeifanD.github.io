---
title: "An analysis and visualization of a holiday classic"
layout: post
date: 2017-01-28 20:20
image: /assets/images/markdown.jpg
headerImage: false
tag:
- Gary's Anatomy
- Text Mining
category: blog
author: WeifanD
---

今天是大年初一，是个好日子，耳边传来的是小岳岳的歌声，手下正敲打着键盘，想说一定要把这个小坑给填了！！
数据挖掘中文本挖掘一直觉得是挺有意思的主题，今天就模仿着David Robinson的[love actually](http://varianceexplained.org/r/love-actually-network/)

首先当然就是获得数据文本，就在百度上搜了一部我最喜欢的医学美剧，'Gary's Anatomy'!

![center](assets/images/2017-01-28/001.jpg)
 
## 数据

我在网上找到的这个剧本是格雷第二季，包括了本集名字／演员的台词以及场景转换，当然还有我觉得最有意思的旁白。 元数据中`form`列是`content`对应台词的角色，同时还包含了每集的名字，通过增加辅助特征来标注旁白和集名。

```r
raw <- readLines("gary's anatomy.txt")
dialogue <- data_frame(raw = raw) %>% 
  filter(raw != "", !is.na(raw)) %>% 
  separate(raw, c("form","content"),sep = ": ",fill = "left") %>% 
  mutate(form = ifelse(is.na(form), "Others", form)) %>% 
  mutate(is_scene = str_detect(form, "2x"),
         scene = cumsum(is_scene))
head(dialogue)
```

![center](assets/images/2017-01-28/002.PNG)

## 主题和高频词

来看看究竟第二季有哪些主题，都是谁写的，剧本中用到的高频单词是什么，以及台词频率最高的是不是就是我们印象中的那些主角？

我们来看一下，第二季总共27集，有没有你印象最深的一集呢？我想第一集的writer可能没想到多年之后，有一个胖胖的女生唱了一首足够震撼动人心魄的歌曲，她的第一句音起就是“When the rain is blowing in your eyes and the whole world is on your case”～

```r
library(tidytext)
reg <- "([^A-Za-z\\d#@']|'(?![A-Za-z\\d#@]))"
dialogue_words <- dialogue %>%
  select(content) %>% 
  unnest_tokens(word, content, token = "regex", pattern = reg) %>% 
  filter(!word %in% stop_words$word)

library(wordcloud2)
dialogue_words %>% 
  count(word, sort = T) %>% 
  as.data.frame(.) %>% 
  wordcloud2(size = 1.5, shape = 'star')

senti_stat <- dialogue_words %>% 
  inner_join(sentiments)
  
senti_stat %>% 
  filter(sentiment == "positive") %>% 
  count(word, sort = T) %>% 
  as.data.frame(.) %>% 
  wordcloud2(size = 1)
```
![center](assets/images/2017-01-28/004.PNG)

![center](assets/images/2017-01-28/003.PNG)

作为一部典型的美剧，格雷的台词除却医学专用此外，日常常用词还是占多数的，像什么god／ah／guy之类的口头语，chief／Dr的title，当然也有医学题材不可避免的surgery，接下来看看这部剧积极的情绪词频。
跟想象中差不多，lucky／love／happy占首位

接下来我们挖一下人物关系吧

![](http://p1.bqimg.com/567571/0aae117158654918.png)

```r  
lines <- character_df %>%
    filter(!is_scene) %>%
    rename(speaker = form, dialogue = content) %>% 
    group_by(scene, line = cumsum(!is.na(speaker))) %>%
    summarize(speaker = speaker[1], dialogue = str_c(dialogue, collapse = " ")) %>% 
  mutate(speaker = str_replace(speaker,"\\(.+", "")) %>% 
  filter(!is.na(speaker))
```

现在每集每个人每句台词形成一行，也就是one scene one line one observation, 将其转变成“speaker-by-scene matrix”，为之后的聚类做准备。

啦啦啦我们五大主角果然聚在一起了，谁叫他们工作在一起住还住一起～

![center](http://p1.bpimg.com/567571/8ab454ce2effc2ca.png)

我们可以看到每一集出现人物的关系线，Alex居然第九集没有出场，Mark第18集才出场，看来我得去回顾一下第二季了，先到这里等我review剧之后再来理一下。

