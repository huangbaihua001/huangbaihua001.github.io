+++
author = "柏华"
title = "在Hubo中使用Mermaid"
date = "2021-05-01"
description = "在Hubo中使用Mermaid"
featured = true
categories = [
"Hugo",
]

tags = [
    "Hugo",
    "Mermaid",
    "原创",
]

isCJKLanguage = true
+++


本文章简要描述了在Hugo中如何引入[Mermaid](https://mermaidjs.github.io/)支持以及几个简单的图例

<!--more-->

# Hugo中引入Mermaid

1. 在 static 目录下创建 mermaid 目录, 去[Mermaid](https://mermaidjs.github.io/)上面下载 mermaid.js 并拷贝至该目录中

![img.png](/images/img.png)

2. 在 layouts/shortcodes 目录下新建 mermaid.html 文件，内容如下

```html
{{ $_hugo_config := `{ "version": 1 }` }}
<div class="mermaid" align="{{ if .Get "align" }}{{ .Get "align" }}{{ else }}center{{ end }}">{{ safeHTML .Inner }}</div>

```

3. 在 config/_default/params.toml 文件中添加 enableMermaid 参数

```
enableMermaid = true
```

4. 在 layouts/partials/footer.html 中加入如下代码

```js
{{ if (ne .Params.enableMermaid false) }}
<script src="{{"mermaid/mermaid.js" | relURL}}{{ if not .Site.Params.disableAssetsBusting }}?{{ now.Unix }}{{ end }}"></script>
{{- end }}
```

5. 在需要使用的地方使用如下代码块

```js

   {{</* mermaid */>}}
   sequenceDiagram
   participant Alice
   participant Bob
   Alice->>John: Hello John, how are you?
   loop Healthcheck
   John->John: Fight against hypochondria
   end
   Note right of John: Rational thoughts <br/>prevail...
   John-->Alice: Great!
   John->Bob: How about you?
   Bob-->John: Jolly good!
   {{</* /mermaid */>}}

```
以上代码渲染如下:

{{< mermaid >}}
sequenceDiagram
participant Alice
participant Bob
Alice->>John: Hello John, how are you?
loop Healthcheck
John->John: Fight against hypochondria
end
Note right of John: Rational thoughts <br/>prevail...
John-->Alice: Great!
John->Bob: How about you?
Bob-->John: Jolly good!
{{< /mermaid >}}

# 主要图例

## 流程图


    {{</*mermaid align="left"*/>}}
    graph LR;
        A[Hard edge] -->|Link text| B(Round edge)
        B --> C{Decision}
        C -->|One| D[Result one]
        C -->|Two| E[Result two]
    {{</* /mermaid */>}}


渲染如下:

{{<mermaid align="left">}}
graph LR;
A[Hard edge] -->|Link text| B(Round edge)
B --> C{Decision}
C -->|One| D[Result one]
C -->|Two| E[Result two]
{{</mermaid>}}

## 序列图

    {{</* mermaid */>}}
    sequenceDiagram
        participant Alice
        participant Bob
        Alice->>John: Hello John, how are you?
        loop Healthcheck
            John->John: Fight against hypochondria
        end
        Note right of John: Rational thoughts <br/>prevail...
        John-->Alice: Great!
        John->Bob: How about you?
        Bob-->John: Jolly good!
    {{</* /mermaid */>}}

渲染如下:

{{<mermaid>}}
sequenceDiagram
participant Alice
participant Bob
Alice->>John: Hello John, how are you?
loop Healthcheck
John->John: Fight against hypochondria
end
Note right of John: Rational thoughts <br/>prevail...
John-->Alice: Great!
John->Bob: How about you?
Bob-->John: Jolly good!
{{</mermaid>}}

## 甘特图

    {{</* mermaid */>}}
    gantt
            dateFormat  YYYY-MM-DD
            title Adding GANTT diagram functionality to mermaid
            section A section
            Completed task            :done,    des1, 2014-01-06,2014-01-08
            Active task               :active,  des2, 2014-01-09, 3d
            Future task               :         des3, after des2, 5d
            Future task2               :         des4, after des3, 5d
            section Critical tasks
            Completed task in the critical line :crit, done, 2014-01-06,24h
            Implement parser and jison          :crit, done, after des1, 2d
            Create tests for parser             :crit, active, 3d
            Future task in critical line        :crit, 5d
            Create tests for renderer           :2d
            Add to mermaid                      :1d
    {{</* /mermaid */>}}


渲染如下:

{{<mermaid>}}
gantt
dateFormat  YYYY-MM-DD
title Adding GANTT diagram functionality to mermaid
section A section
Completed task            :done,    des1, 2014-01-06,2014-01-08
Active task               :active,  des2, 2014-01-09, 3d
Future task               :         des3, after des2, 5d
Future task2               :         des4, after des3, 5d
section Critical tasks
Completed task in the critical line :crit, done, 2014-01-06,24h
Implement parser and jison          :crit, done, after des1, 2d
Create tests for parser             :crit, active, 3d
Future task in critical line        :crit, 5d
Create tests for renderer           :2d
Add to mermaid                      :1d
{{</mermaid>}}


### 类图

    {{</* mermaid */>}}
    classDiagram
      Class01 <|-- AveryLongClass : Cool
      Class03 *-- Class04
      Class05 o-- Class06
      Class07 .. Class08
      Class09 --> C2 : Where am i?
      Class09 --* C3
      Class09 --|> Class07
      Class07 : equals()
      Class07 : Object[] elementData
      Class01 : size()
      Class01 : int chimp
      Class01 : int gorilla
      Class08 <--> C2: Cool label
    {{</* /mermaid */>}}

渲染如下:


{{<mermaid>}}
classDiagram
Class01 <|-- AveryLongClass : Cool
Class03 *-- Class04
Class05 o-- Class06
Class07 .. Class08
Class09 --> C2 : Where am i?
Class09 --* C3
Class09 --|> Class07
Class07 : equals()
Class07 : Object[] elementData
Class01 : size()
Class01 : int chimp
Class01 : int gorilla
Class08 <--> C2: Cool label
{{</mermaid>}}


### Git

    {{</* mermaid */>}}
    gitGraph:
    options
    {
      "nodeSpacing": 150,
      "nodeRadius": 10
    }
    end
      commit
      branch newbranch
      checkout newbranch
      commit
      commit
      checkout master
      commit
      commit
      merge newbranch
    {{</* /mermaid*/>}}

渲染如下:

{{<mermaid>}}
gitGraph:
options
{
"nodeSpacing": 150,
"nodeRadius": 10
}
end
commit
branch newbranch
checkout newbranch
commit
commit
checkout master
commit
commit
merge newbranch
{{</mermaid>}}

### 参考资料
1. https://learn.netlify.app/en/shortcodes/mermaid/
2. https://github.com/matcornic/hugo-theme-learn
3. https://mermaid-js.github.io

(全文完)




