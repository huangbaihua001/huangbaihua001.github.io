+++
author = "柏华"
title = "PSI(程序结构接口)-Intellij 平台的核心抽象"
date = "2021-12-04"
description = "PSI-Intellij Platform 平台的核心抽象"
featured = true
categories = [
"译文",
"IDEA插件开发",
]

tags = [
"译文",
"IDEA插件开发",
]

isCJKLanguage = true
#thumbnail = "/images/idea/import-maven-intellij-thumbnail.png"
+++


Intellij IDEA 是业界公认最智能，最强大的 Java IDE。个人认为 Intellij 平台底层的抽象能力是非常强的。 PSI 就是其中之一。

**PSI** 是程序结构接口 **(Program Structure Interface)** 的简称，在 Intellij 平台中负责解析文件、创建语法和语义代码模型。
它是平台的核心抽象层，支持平台的许多功能。

<!--more-->

{{< toc >}}

译自: [PSI](https://plugins.jetbrains.com/docs/intellij/psi.html) (intellij)

# 1. PSI 文件

PSI 文件是一个结构的根，在特定的编程语言中将文件内容表示为一种具有层次结构的元素集合。

**PsiFile** 类是所有 PSI 文件的共同基类，而特定语言的文件通常由其子类表示。例如，**PsiJavaFile** 类代表一个Java文件，而 **XmlFile** 类代表一个 XML 文件。

与 **VirtualFile** 和 **Document** 不同，**VirtualFile** 和 **Document** 有其应用范围（即使打开了多个项目，每个文件都由同一个 **VirtualFile** 实例表示），
PSI 有其项目范围：如果文件属于同时打开的多个项目，同一个文件由多个 **PsiFile** 实例表示。

## 1.1 如何获取 PSI 文件？

- 从一个 Action：     **e.getData(CommonDataKeys.PSI_FILE).** 
- 从一个 VirtualFile:  **PsiManager.getInstance(project).findFile()**
- 从一个 Document: **PsiDocumentManager.getInstance(project).getPsiFile()**
- 从一个文件中的元素: **PsiElement.getContainingFile()**
- 在项目任意地方获取具有特定名称的文件，使用 **FilenameIndex.getFilesByName(project, name, scope)**

## 1.2 可以用 PSI 文件做什么？

大多数有趣的修改操作是在单个 PSI 元素的层面上进行的，而不是整个文件。

要遍历一个文件中的元素，可以使用

```java
    psiFile.accept(new PsiRecursiveElementWalkingVisitor() {
      // visitor implementation ...
    });
```
## 1.3 PSI 文件来自哪里？

因为 PSI 是依赖于语言的，因此 PSI 文件使用 **Language** 实例创建:

```java
LanguageParserDefinitions.INSTANCE
      .forLanguage(MyLanguage.INSTANCE)
      .createFile(fileViewProvider);
```
与文件一样，PSI 文件是在访问特定文件的 PSI 时按需创建的。

## 1.4 PSI 文件能保存多久？

像文档一样，PSI 文件从相应的 **VirtualFile** 实例中被弱引用，如果不被任何实例引用，可以被垃圾收集器收集。

## 1.5 如何创建PSI文件？

**PsiFileFactory createFileFromText()** 方法以指定的内容在内存中创建一个 PSI 文件。

要将 PSI 文件保存到磁盘，请使用 **PsiDirectory add()** 方法。

## 1.6 当 PSI 文件改变时，如何得到通知？

**PsiManager.getInstance(project).addPsiTreeChangeListener()** 允许你接收有关项目中的 PSI 树的所有变化的通知。
或者，在 **com.intellij.psi.treeChangeListener** 扩展点中注册 **PsiTreeChangeListener**。

请参见 [PsiTreeChangeEvent](https://upsource.jetbrains.com/idea-ce/file/idea-ce-2020312d547dd0b755241d10a1a3eec025f8efe7/platform/core-api/src/com/intellij/psi/PsiTreeChangeEvent.java?_ga=2.194139565.1831407401.1638449592-1206068809.1622726639) ,了解处理 PSI 事件时的常见问题.

## 1.7 如何扩展 PSI ?

PSI 可以通过自定义语言插件扩展支持更多的语言。关于开发自定义语言插件的更多细节，
请参见 [自定义语言支持](https://plugins.jetbrains.com/docs/intellij/custom-language-support.html) 参考指南。

## 1.8 使用 PSI 有什么规则 ?
对 PSI 文件内容所做的任何改变都会反映在文档中，因此所有 
[处理文档的规则](https://plugins.jetbrains.com/docs/intellij/documents.html#what-are-the-rules-of-working-with-documents) （读/写操作、命令、只读状态处理）都是有效的。

# 2. 文件视图提供者(File View Providers)

一个文件视图提供者（**FileViewProvider**）管理对单个文件中多个 PSI 树的访问。

例如，一个 JSP 页面中的 Java 代码有一个单独的 PSI 树（**PsiJavaFile**），
XML 代码有一个单独的树（**XmlFile**），整个 JSP 有一个单独的树（**JspFile**）。

每个 PSI 树都涵盖了文件的全部内容，并在可以找到不同语言内容的地方包含了特殊的 "外部语言元素"。

一个 **FileViewProvider** 实例对应于一个 VirtualFile，一个 Document，并且可以检索到多个 PsiFile 实例。

## 2.1 如何获得 FileViewProvider？

- 从一个 VirtualFile：**PsiManager.getInstance(project).findViewProvider()**
- 从一个 PsiFile: **psiFile.getViewProvider()**

## 2.2 可以用 FileViewProvider 做什么？

- 获取文件中存在 PSI 树的所有语言集：**fileViewProvider.getLanguages()**
- 获取某一特定语言的 PSI 树：**fileViewProvider.getPsi(language)**。例如，要获得 XML的 PSI 树，使用 **fileViewProvider.getPsi(XMLLanguage.INSTANCE)。**
- 要在文件的指定偏移处找到一个特定语言的元素：**fileViewProvider.findElementAt(offset, language)**

## 2.3 如何扩展 FileViewProvider?

要创建一个新的文件类型，该插件必须包含一个 **com.intellij.fileType.fileViewProviderFactory** 扩展点的扩展。

实现 **FileViewProviderFactory** 并从 **createFileViewProvider()** 方法返回你的 **FileViewProvider** 实现。

在 **plugin.xml** 中注册如下:
```xml
<extensions defaultExtensionNs="com.intellij">
  <fileType.fileViewProviderFactory
          filetype="$file_type$" 
          implementationClass="com.plugin.MyFileViewProviderFactory" />
</extensions>
```
其中 $file_type$ 指的是正在创建的文件类型(例如，“JFS”)。

# 3 PSI 元素

一个 PS I(程序结构接口)文件表示一个 PSI 元素的层次结构(所谓的 PSI 树)。 
一个 PSI 文件(本身就是一个 PSI 元素)可能包含几个特定编程语言的 PSI 树。 
反过来，一个 PSI 元素也可以有子 PSI 元素。

PSI 元素和单个 PSI 元素层面的操作被用来探索源代码的内部结构，因为它是由 IntelliJ 平台解释的。
例如，你可以使用 PSI 元素来进行代码分析，如 [代码检查](https://www.jetbrains.com/help/idea/code-inspection.html?_ga=2.176256293.1831407401.1638449592-1206068809.1622726639) 或 [意图操作](https://www.jetbrains.com/idea/help/intention-actions.html?_ga=2.176256293.1831407401.1638449592-1206068809.1622726639) 。

**PsiElement** 类是 PSI 元素的通用基类。

## 3.1 怎么获取 PSI 元素

- 从一个 Action： **e.getData（LangDataKeys.PSI_ELEMENT）**。注意：如果当前打开了一个编辑器，并且被关注的元素是一个引用，这将返回解析该引用的结果。这可能是你需要的，也可能不是。
- 从一个文件中的偏移量： **PsiFile.findElementAt()**。注意：这将返回指定偏移量的最低级别元素（"叶子元素"），通常是一个词法标记。 最有可能的是，你应该使用 **PsiTreeUtil.getParentOfType()** 来找到你真正需要的元素。
- 通过迭代 PSI 文件：使用 **PsiRecursiveElementWalkingVisitor** 。
- 通过解析一个引用: **PsiReference.resolve()**

## 3.2 可以用 PSI 元素做什么？

请参见 [PSI Cookbook](#7-psi-cookbook)


# 4. PSI 导航

有三种主要方式来进行 PSI 导航：自上而下法，自下而上法，以及引用法。

第一种方法适用场景：你有一个 PSI 文件或另一个更高层次的元素（例如，一个方法），
需要找到所有符合指定条件的元素（例如，所有变量声明）。

第二种方法适用场景：你在 PSI 树中有一个特定的点（例如，位于光标所在位置的元素），
需要找出关于它的上下文信息（例如，它被声明的元素）。

最后，引用允许你从一个元素的使用（例如，一个方法的调用）导航到声明（被调用的方法），再返回。
引用将在一个单独的 [章节](#5-psi-) 中讲述。

## 4.1 自上而下导航

执行自上而下导航的最常见的方法是使用 [**Visitor**](https://en.wikipedia.org/wiki/Visitor_pattern) 模式 。
要使用一个 Visitor ，你要创建一个类（通常是一个匿名的内部类），它扩展了 Visitor 类，
重写了处理元素的方法，并将 Visitor 实例传递给 PsiElement.accept() 方法。

**Visitor** 的基类是特定于语言的。例如，如果需要处理 Java 文件中的元素，可以扩展 **JavaRecursiveElementVisitor** 并重写相关方法。

下面的代码显示了使用 **Visitor** 查找所有 Java 局部变量声明的情况。

```java
file.accept(new JavaRecursiveElementVisitor() {
  @Override
  public void visitLocalVariable(PsiLocalVariable variable) {
    super.visitLocalVariable(variable);
    System.out.println("Found a variable at offset " + variable.getTextRange().getStartOffset());
  }
});
```

在大多数情况下，你也可以使用更具体的 API 进行自上而下的导航。例如，如果你需要获得一个 Java 类中所有方法的列表，
你可以使用一个 Visitor ，但更简单的方法是调用 **PsiClass.getMethods()**。

**PsiTreeUtil** 包含一些通用的、独立于语言的 PSI 树状检索函数，其中一些函数（例如 **findChildrenOfType()** ）执行自顶向下的导航。

## 4.2 自下而上导航

自下而上导航的起点是 PSI 树中的一个特定元素（例如，解析一个引用的结果）或一个偏移量。
如果已知一个偏移量，你可以通过调用 **PsiFile.findElementAt()** 找到相应的 PSI 元素。
这个方法返回树中最低层的元素（例如，一个标识符），如果你想确定更广泛的上下文，你需要向上导航这颗树。

在大多数情况下，自下而上的导航是通过调用 **PsiTreeUtil.getParentOfType()** 进行的。这个方法在树中往上查找，直到找到你指定的类型的元素。
例如，为了找到包含的方法，你调用 **PsiTreeUtil.getParentOfType(element, PsiMethod.class)** 方法。

在某些情况下，你也可以使用特定的导航方法。例如，要找到一个方法所在的类，你可以调用 **PsiMethod.getContainingClass()** 方法。

下面的代码片段显示了这些调用如何一起使用。

```java
PsiFile psiFile = anActionEvent.getData(CommonDataKeys.PSI_FILE);
PsiElement element = psiFile.findElementAt(offset);
PsiMethod containingMethod = PsiTreeUtil.getParentOfType(element, PsiMethod.class);
PsiClass containingClass = containingMethod.getContainingClass();
```

要了解导航的实际运行，请参考 [代码实例](https://github.com/JetBrains/intellij-sdk-code-samples/blob/main/psi_demo/src/main/java/org/intellij/sdk/psi/PsiNavigationDemoAction.java) 。

# 5. PSI 引用

PSI 树中的引用是一个对象，它代表了从代码中某一特定元素的使用到相应声明的链接。解析一个引用意味着找到一个特定使用所指向的声明。

最常见的引用类型是由语言语义定义的。例如，考虑一个简单的 Java 方法。

```java
public void hello(String message) {
    System.out.println(message);
}
```
这个简单的代码片段包含五个引用。由标识符 String、System、out 和 println 创建的引用可以被解析为 JDK 中的相应声明：
String 和 System 类、out 字段和 println 方法； println(message) 中第二次出现的 **message** 标识符所创建的引用可以被解析为方法参数，
在方法头中由 String message 声明。

请注意，String message 不是一个引用，不能被解析。相反，它是一个声明。它没有引用在其他地方定义的任何名称; 相反，它自己定义一个名称。

要解析引用--找到被引用的声明--请调用 **PsiReference.resolve()** 。理解 **PsiReference.getElement()** 和 **PsiReference.resolve()** 之间的区别非常重要。
前一个方法返回引用的来源，而后一个方法返回其目标。在上面的例子中，对于 **message** 引用，getElement() 将返回片段第二行的 **message** 标识符，
而 resolve() 将返回第一行（参数列表内）的 **message** 标识符。

解析引用的过程与解析不同，不是同时进行的。此外，它并不总是成功的。如果当前在 IDE 中打开的代码不能编译，或者在其他情况下，
PsiReference.resolve() 返回 null 是正常的，所有处理引用的代码都必须准备好处理这个问题。

## 5.1 提供的引用

除了由编程语言的语义定义的引用外，IDE 还能识别由代码中使用的 API 和框架的语义决定的许多引用。请看下面的例子。

```java
File f = new File("foo.txt");
```
在这里，"foo.txt "从 Java 语法的角度来看没有什么特殊的含义--它只是一个字符串。
然而，在 IntelliJ IDEA 中打开这个例子，并在同一目录下有一个名为 "foo.txt "的文件，可以按 Ctrl/Cmd+ 点击 "foo.txt "并导航到该文件。
这是因为 ID E识别了 new File(...) 的语义，并在作为参数传递给该方法的字符串提供了一个引用。

通常情况下，引用可以被提供给那些没有自己的引用的元素，如字符串和注释。引用也经常被提供给非代码文件，如 XML 或 JSON。

提供引用是扩展现有语言的最常见方式之一。例如，你的插件可以提供对 Java 代码的引用，尽管 Java PSI 是平台的一部分，并不需要在你的插件中定义。

实现 **PsiReferenceContributor**, 在扩展点 **com.intellij.psi.referenceContributor** 中注册。
然后在调用 **PsiReferenceRegistrar.registerReferenceProvider()** 时使用 [Element Patterns](https://plugins.jetbrains.com/docs/intellij/element-patterns.html) 指定要引用的地方。

另请参见 [引用提供指南](https://plugins.jetbrains.com/docs/intellij/reference-contributor.html)

## 5.2 具有可选或多个解析结果的引用

在最简单的情况下，一个引用会解析到一个元素，如果解析失败，代码就不正确，IDE 需要将其作为一个错误突出显示。然而，在有些情况下，情况会有所不同。

第一种情况是软引用。考虑一下上面的 new File("foo.txt") 例子。如果 IDE 找不到 "foo.txt "这个文件，这并不意味着需要突出显示一个错误--也许这个文件只在运行时可用。
这样的引用从 **PsiReference.isSoft()** 方法返回真。

第二种情况是多变体引用。考虑一下 JavaScript 程序的情况。JavaScript 是一种动态类型的语言，所以 IDE 不能总是精确地确定在某一特定位置正在调用哪个方法。
为了处理这个问题，它提供了一个可以被解析为多种可能元素的引用。这种引用实现了 **PsiPolyVariantReference** 接口。

为了解析一个 **PsiPolyVariantReference**，你调用它的 **multiResolve()** 方法。该调用返回一个 **ResolveResult** 对象的数组。
每一个对象都标识了一个 PSI 元素，并且还指定了结果是否有效。例如，假设你有多个 Java 方法的重载，以及一个参数不匹配任何重载的调用。
在这种情况下，你将得到所有重载的 ResolveResult 对象，并且 isValidResult() 对所有这些对象都返回 false。

## 5.3 查找引用

正如你所知，解析一个引用意味着从使用到相应的声明。要进行相反方向的导航--从声明到其使用--执行引用搜索。

要使用 **ReferencesSearch** 进行搜索，需要指定要搜索的元素，以及可选择的其他参数，如需要搜索的引用的范围。
创建的 **Query** 允许一次获得所有的结果，或者一个一个地遍历结果。后者允许在找到第一个（匹配）结果后立即停止处理。


## 5.4 实现引用

更多信息请参考 [指南](https://plugins.jetbrains.com/docs/intellij/references-and-resolve.html) 和相应的 [教程](https://plugins.jetbrains.com/docs/intellij/reference-contributor.html) 。

# 6. 修改 PSI

PSI 是对源代码的一种读写表示，是对应于源文件结构的元素树。你可以通过添加、替换和删除 PSI 元素来修改 PSI。

为了执行这些操作，你可以使用 **PsiElement.add()**、**PsiElement.delete()** 和 **PsiElement.replace()** 等方法，
以及 **PsiElement** 接口中定义的其他方法，这些方法可以让你在一次操作中处理多个元素，或者指定树中需要添加元素的确切位置。

和文档操作一样，PSI 的修改需要用写操作和命令来包装（只能在事件调度线程中执行）。更多关于命令和写操作的信息，请参见 [文档](https://plugins.jetbrains.com/docs/intellij/documents.html#what-are-the-rules-of-working-with-documents) 。

## 6.1 创建 PSI

要添加到树中或替换现有 PSI 元素的 PSI 元素通常是由文本创建的。在一般的情况下，
使用 **PsiFileFactory** 的 **createFileFromText()** 方法来创建一个新的文件，该文件包含了你需要添加到树中的代码结构，
或者作为现有元素的替换，遍历树定位你需要的位置，然后将该元素传递给 **add()** 或 **replace()** 方法。

大多数语言都提供了工厂方法，让你更容易创建特定的代码结构。
例如，**PsiJavaParserFacade** 类包含诸如 **createMethodFromText()** 的方法，它从给定的文本中创建一个 Java 方法。

当你实现重构、意图或检查快速修复，与现有的代码一起工作时，传递给各种 **createFromText()** 方法的文本会结合**硬编码片段**和从现有文件中**提取的代码片段**。
对于小的代码片段（单个标识符），你可以简单地将现有代码中的文本附加到你要构建的代码片段的文本中。
在这种情况下，你需要确保产生的文本在语法上是正确的。否则，createFromText() 方法将抛出一个异常。

对于较大的代码片段，最好分几步进行修改。

1. 从文本中创建一个替换树形片段，为用户代码片段预留占位符。

2. 用用户代码片段替换占位符。

3. 用替换树替换原始源文件中的元素。

这确保了用户代码的格式被保留下来，并且修改不会引入任何不需要的空格变化。

参照该方法，请看 **ComparingReferencesInspection** 快速修复示例。

```java
// binaryExpression 持有形式为 "x == y "的 PSI 表达式，需要用 "x.equals(y) "替换。
PsiBinaryExpression binaryExpression = (PsiBinaryExpression) descriptor.getPsiElement();
IElementType opSign = binaryExpression.getOperationTokenType();
PsiExpression lExpr = binaryExpression.getLOperand();
PsiExpression rExpr = binaryExpression.getROperand();

// 第1步：从文本中创建一个替换片段，以 "a "和 "b "作为占位符
PsiElementFactory factory = JavaPsiFacade.getInstance(project).getElementFactory();
PsiMethodCallExpression equalsCall =
    (PsiMethodCallExpression) factory.createExpressionFromText("a.equals(b)", null);

// 第2步：用原文件中的元素替换 "a "和 "b"。
equalsCall.getMethodExpression().getQualifierExpression().replace(lExpr);
equalsCall.getArgumentList().getExpressions()[0].replace(rExpr);

// 第3步：用替换树替换原始文件中的一个较大的元素
PsiExpression result = (PsiExpression) binaryExpression.replace(equalsCall);
```

就像 IntelliJ 平台 API 中的其他地方一样，传递给 createFileFromText()和其他 createFromText() 方法的文本必须只使用 /n 作为换行符。

## 6.2 保持树结构的一致性

PSI 的修改方法并不会限制你构建结果树结构的方式。例如，在处理一个 Java 类时，你可以添加一个 for 语句作为 PsiMethod 元素的直接子元素，
尽管 Java 解析器永远不会产生这样的结构（for语句永远是 PsiCodeBlock 的子元素）代表方法体。
产生不正确的树状结构的修改可能看起来是有效的，但它们将导致后续的问题和异常。
因此，你总是需要确保你用 PSI 修改操作建立的结构与解析器在解析你所创建的代码时产生的结构相同。

为了确保你没有引入不一致，你可以在你修改 PSI 的操作的测试中调用 PsiTestUtil.checkFileStructure（）。
这个方法可以确保你建立的结构与解析器产生的结构是一致的。

## 6.3 空格和包导入

当使用 PSI 修改函数时，你不应该从文本中创建单独的空白节点（空格或换行符）。相反，所有的空白修改都由格式化处理器执行，它遵循用户选择的代码风格设置。
格式化是在每个命令的末尾自动执行的，如果需要，也可以使用 **CodeStyleManager** 类中的 **reformat(PsiElement)** 方法手动执行。

另外，在处理 Java 代码时（或处理其他具有类似导入机制的语言的代码，如 Groovy 或 Python），你不应该手动创建导入。
相反，你应该在你生成的代码中插入完全限定的名称，然后调用 **JavaCodeStyleManager** 中的 **shortenClassReferences()** 方法（或者你正在使用的语言的同等API）。
这可以确保导入是根据用户的代码风格设置创建的，并插入到文件的正确位置。

## 6.4 结合 PSI 和文件的修改

在某些情况下，你需要进行 PSI 修改，然后通过 PSI 对你刚刚修改的文档进行操作（例如，启动一个实时模板）。
在这种情况下，你需要调用一个特殊的方法来完成基于 PSI 的后期处理（如格式化），并将修改提交给文档。
你需要调用的方法叫做 **doPostponedOperationsAndUnblockDocument()**，它被定义在 **PsiDocumentManager** 类中。



# 7 PSI Cookbook
本章节给出了使用 PSI（程序结构接口）的最常用操作。与开发自定义语言插件不同的是，它讲述了与现有语言（如Java）的 PSI 有关的操作。

另请参见 [《PSI 高效操作》](https://plugins.jetbrains.com/docs/intellij/performance.html#working-with-psi-efficiently) 。

## 7.1 通用

- 已知一个文件的名字，但不知道它的路径，如何找到它？

**[FilenameIndex.getFilesByName()](https://upsource.jetbrains.com/idea-ce/file/idea-ce-2020312d547dd0b755241d10a1a3eec025f8efe7/platform/indexing-api/src/com/intellij/psi/search/FilenameIndex.java?_ga=2.126964109.1831407401.1638449592-1206068809.1622726639)**

- 如何找到一个特定的 PSI 元素在哪些地方使用？

[**ReferencesSearch.search()**](https://upsource.jetbrains.com/idea-ce/file/idea-ce-2020312d547dd0b755241d10a1a3eec025f8efe7/platform/indexing-api/src/com/intellij/psi/search/searches/ReferencesSearch.java?_ga=2.206747795.1831407401.1638449592-1206068809.1622726639)


- 如何重命名一个 PSI 元素？

[**RefactoringFactory.createRename()**](https://upsource.jetbrains.com/idea-ce/file/idea-ce-2020312d547dd0b755241d10a1a3eec025f8efe7/platform/lang-api/src/com/intellij/refactoring/RefactoringFactory.java?_ga=2.127103373.1831407401.1638449592-1206068809.1622726639)


- 如何重建一个虚拟文件的 PSI？

[**FileContentUtil.reparseFiles()**](https://upsource.jetbrains.com/idea-ce/file/idea-ce-2020312d547dd0b755241d10a1a3eec025f8efe7/platform/analysis-api/src/com/intellij/util/FileContentUtil.java?_ga=2.143291573.1831407401.1638449592-1206068809.1622726639)

## 7.2 Java
- 如何找到一个类的所有子类？

[ClassInheritorsSearch.search()](https://upsource.jetbrains.com/idea-ce/file/idea-ce-2020312d547dd0b755241d10a1a3eec025f8efe7/java/java-indexing-api/src/com/intellij/psi/search/searches/ClassInheritorsSearch.java?_ga=2.163165372.1831407401.1638449592-1206068809.1622726639)

- 如何通过限定名称找到一个类？

[JavaPsiFacade.findClass()](https://upsource.jetbrains.com/idea-ce/file/idea-ce-2020312d547dd0b755241d10a1a3eec025f8efe7/java/java-psi-api/src/com/intellij/psi/JavaPsiFacade.java?_ga=2.230717855.1831407401.1638449592-1206068809.1622726639)

- 如何通过类名找到一个类？

PsiShortNamesCache.getClassesByName()

- 如何找到一个 Java 类的超类？

[PsiClass.getSuperClass()](https://upsource.jetbrains.com/idea-ce/file/idea-ce-2020312d547dd0b755241d10a1a3eec025f8efe7/java/java-indexing-api/src/com/intellij/psi/search/PsiShortNamesCache.java?_ga=2.230717855.1831407401.1638449592-1206068809.1622726639)

- 如何获得一个 Java 类所在包的引用？

```java
PsiJavaFile javaFile = (PsiJavaFile) psiClass.getContainingFile()
PsiPackage psiPackage = JavaPsiFacade.getInstance(project)
                       .findPackage(javaFile.getPackageName())
```
或

[PsiUtil.getPackageName()](https://upsource.jetbrains.com/idea-ce/file/idea-ce-2020312d547dd0b755241d10a1a3eec025f8efe7/java/java-psi-api/src/com/intellij/psi/util/PsiUtil.java?_ga=2.230717855.1831407401.1638449592-1206068809.1622726639)

- 如何找到指定方法的重写方法？

[OverridingMethodsSearch.search()](https://upsource.jetbrains.com/idea-ce/file/idea-ce-2020312d547dd0b755241d10a1a3eec025f8efe7/java/java-indexing-api/src/com/intellij/psi/search/searches/OverridingMethodsSearch.java?_ga=2.230717855.1831407401.1638449592-1206068809.1622726639)

---
全文完