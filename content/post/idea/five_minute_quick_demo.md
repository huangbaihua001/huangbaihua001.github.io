+++
author = "柏华"
title = "5分钟 IDEA 插件开发快速入门 Demo"
date = "2021-11-20"
description = "5分钟 IDEA 插件开发快速入门 Demo"
featured = true
categories = [
"Intellij IDEA",
"插件开发",
"5分钟入门系列",
]

tags = [
"5分钟入门系列",
"插件开发",
"Intellij IDEA",
]

isCJKLanguage = true
+++
![](/images/idea/import-maven-intellij-thumbnail.png)

基于 Gradle 开发 IntelliJ 插件是官方推荐的方式。本 Demo
使用 [IntelliJ Platform Plugin Template](https://github.com/JetBrains/intellij-platform-plugin-template) 快速构建一个插件项目 。
利用 [IntelliJ Platform](https://plugins.jetbrains.com/docs/intellij/intellij-platform.html) 平台开发自己的第一个插件！

<!--more-->

# 5分钟 IDEA 插件开发快速入门 Demo

基于 Gradle 开发 IntelliJ 插件是官方推荐的方式。本 Demo
使用 [IntelliJ Platform Plugin Template](https://github.com/JetBrains/intellij-platform-plugin-template) 快速构建一个插件项目 。
利用 [IntelliJ Platform](https://plugins.jetbrains.com/docs/intellij/intellij-platform.html) 平台开发自己的第一个插件！

![Requirement](https://img.shields.io/badge/JDK-8+-green.svg)

## Demo简介

本Demo主要演示如何快速构建一个插件项目，实现一个Action，对IDEA插件开发有个最基本的了解。

本Demo目标：

- 快速搭建一个插件项目
- 完成一个Menu Action：选中Java代码并替换成固定字符串
- 了解PSI概念:  编写一个Action,弹出框显示Java类文件中光标所在位置的方法相关的PSI信息

---
项目结构预览
```
~5-Minutes-Demo~\QUICK-PLUGIN-DEMO
|   .gitignore
|   build.gradle.kts
|   CHANGELOG.md
|   CODE_OF_CONDUCT.md
|   gradle.properties
|   gradlew
|   gradlew.bat
|   LICENSE
|   qodana.yml
|   README.md
|   settings.gradle.kts
|   
+---.github
|   |   dependabot.yml
|   |   
|   +---ISSUE_TEMPLATE
|   |       bug_report.md
|   |       
|   +---readme
|   |       draft-release.png
|   |       intellij-platform-plugin-template.png
|   |       qodana.png
|   |       run-debug-configurations.png
|   |       run-logs.png
|   |       settings-secrets.png
|   |       ui-testing.png
|   |       use-this-template.png
|   |       
|   +---template-cleanup
|   |   |   CHANGELOG.md
|   |   |   gradle.properties
|   |   |   README.md
|   |   |   settings.gradle.kts
|   |   |   
|   |   \---.github
|   |           dependabot.yml
|   |           
|   \---workflows
|           build.yml
|           release.yml
|           run-ui-tests.yml
|           template-cleanup.yml
|           
+---.idea
|       icon.png
|       
+---.run
|       Run IDE for UI Tests.run.xml
|       Run IDE with Plugin.run.xml
|       Run Plugin Tests.run.xml
|       Run Plugin Verification.run.xml
|       Run Qodana.run.xml
|       
+---gradle
|   \---wrapper
|           gradle-wrapper.jar
|           gradle-wrapper.properties
|           
\---src
    +---main
    |   +---java
    |   |   \---jiux
    |   |       \---net
    |   |           \---idea
    |   |               \---plugin
    |   |                   \---demo
    |   |                       \---action
    |   |                               EditorReplaceAction.java
    |   |                               PsiDemoAction.java
    |   |                               
    |   +---kotlin
    |   |   \---org
    |   |       \---jetbrains
    |   |           \---plugins
    |   |               \---template
    |   |                   |   MyBundle.kt
    |   |                   |   
    |   |                   +---listeners
    |   |                   |       MyProjectManagerListener.kt
    |   |                   |       
    |   |                   \---services
    |   |                           MyApplicationService.kt
    |   |                           MyProjectService.kt
    |   |                           
    |   \---resources
    |       +---messages
    |       |       MyBundle.properties
    |       |       
    |       \---META-INF
    |               plugin.xml
    |               pluginIcon.svg
    |               
    \---test
        +---kotlin
        |   \---org
        |       \---jetbrains
        |           \---plugins
        |               \---template
        |                       MyPluginTest.kt
        |                       
        \---testData
            \---rename
                    foo.xml
                    foo_after.xml

```

## 第一步：利用 IntelliJ Platform Plugin Template 创建项目

``` bash
# clone 项目到本地 

git clone git@github.com:JetBrains/intellij-platform-plugin-template.git quick-plugin-demo

# 删除 github 的远程仓库地址， 切换成自己的

git remote rm origin

git remote add origin 自己的远程仓库地址

```

该步骤有可能因为墙的原因网络被中断，多试几次。

## 第二步：导入项目，进行相关配置

### 修改配置支持Java8

- gradle.properties

```properties

# 插件支持的最小版本改为 202
pluginSinceBuild=202
#增加 2020.2版本
pluginVerifierIdeVersions=2020.2, 2020.3.4, 2021.1.3, 2021.2.1
# IC是社区版，这里用IU企业版
platformType=IU
#该版本为要求Java8的最高版本,在此之后的版本最低要求Java11
platformVersion=2020.2
#JDK1.8
javaVersion=1.8


```

IDEA更多版本号，参见 [最新IDEA版本](https://plugins.jetbrains.com/docs/intellij/build-number-ranges.html#intellij-platform-based-products-of-recent-ide-versions)

- build.gradle.kts

```bash
// 将 Gradle IntelliJ Plugin 的版本修改为 1.0
id("org.jetbrains.intellij") version "1.0"

```

完成以上修改后， 重新加载项目，点击运行 Run Plugin 启动插件。
![first-run.png](/images/idea/first-run.png)

### 支持Java平台

默认只引入了基础平台相关的Jar包，要支持Java语言，需要自己添加。

- gradle.properties

```bash

# 引入Java支持
platformPlugins=com.intellij.java

```

- src/main/resources/META-INF/plugin.xml

```xml

<idea-plugin>

    <!-- 引入Java依赖 -->
    <depends>com.intellij.java</depends>
    <depends>com.intellij.modules.lang</depends>

</idea-plugin>

```

- 创建Java源代码目录

IDEA插件既支持Kotlin,Java语言独立开发，也支持两者混合开发，写的类可以互相调用。 默认只有 kotlin 源代码目录。 Java源代码目录需要手动创建。 创建 src/main/java 即可开始写 Java 代码了。

默认的 kotlin 目录可以删除，但个人建议保留。 因为 kotlin 下的代码可以直接拿来做国际化，有些开源库是 kotlin 写的，可以直接 拿来用，混合开发还是比较有优势的。

完成以上步骤后，刷新下项目，相关的依赖就加入到工程中了, 可以正式开始写代码了。

## 第三步 创建一个 Action

Action 是一个具有状态，展示和行为的实体。 通过继承 AnAction 重写 actionPerformed 方法实现 Action 的行为控制。
通过可选择地重写 update 方法实现 Action 的展示控制。

- com.dmall.rdp.plugin.demo.action.EditorReplaceAction

```java
/**
 * Menu Action 将代码中选中的字符替换成固定的字符
 */
public class EditorReplaceAction extends AnAction {
    /**
     * Action 事件处理
     *
     * @param e 事件对象
     */
    @Override
    public void actionPerformed(@NotNull AnActionEvent e) {
        // 获取当前工程，当前编辑器，当前文档
        final Editor editor = e.getRequiredData(CommonDataKeys.EDITOR);
        final Project project = e.getRequiredData(CommonDataKeys.PROJECT);
        final Document document = editor.getDocument();

        // 获取编辑器当前光标
        Caret primaryCaret = editor.getCaretModel().getPrimaryCaret();
        // 光标选中的开始位置和结束位置
        int start = primaryCaret.getSelectionStart();
        int end = primaryCaret.getSelectionEnd();
        // 将选中的字符替换成固定字符
        // 注意不能直接使用  document.replaceString() 方法
        // document.replace相关操作文档的方法，不能在直接在事件处理上下文线程中执行。
        // 而是 必须 在 写操作的上下文线程中执行，即使用 WriteCommandAction.runWriteCommandAction 方法
        // 因为 这类文档操作 被认为是耗时的操作，不能阻塞UI事件主线程。
        WriteCommandAction.runWriteCommandAction(project, () ->
                document.replaceString(start, end, "~这是替换的~")
        );
        // 刚才替换的字符串取消选中
        primaryCaret.removeSelection();
    }

    /**
     * 控制在菜单中的展示，满足以下条件时可见且可用:
     * <ul>
     *   <li>工程打开</li>
     *   <li>编辑器打开</li>
     *   <li>字符串被选中</li>
     * </ul>
     *
     * @param e Event related to this action
     */
    @Override
    public void update(@NotNull AnActionEvent e) {
        // 获取当前工程
        final Project project = e.getProject();
        // 获取编辑器
        final Editor editor = e.getData(CommonDataKeys.EDITOR);
        // 仅在 当前工程和编辑器不为空时(它们都处于打开的状态)，且存在选中的字符时，设置该 Menu Action 可见且可用
        e.getPresentation().setEnabledAndVisible(
                project != null && editor != null && editor.getSelectionModel().hasSelection());
    }
}



```

- src/main/resources/META-INF/plugin.xml

```xml

<actions>
    <!-- 将此 Action 放到弹出菜单的第一个位置; 如果项目和编辑器是打开的，有字符被选中时，它总是可用的 -->
    <action id="com.dmall.rdp.plugin.demo.action.EditorReplaceAction"
            class="com.dmall.rdp.plugin.demo.action.EditorReplaceAction"
            text="选中替换"
            description="选中替换">
        <!-- 设置快捷键 Ctrl+Alt+G -->
        <keyboard-shortcut keymap="$default" first-keystroke="control alt G"/>
        <!-- 放到编辑器弹出菜单第一个位置 -->
        <add-to-group group-id="EditorPopupMenu" anchor="first"/>
    </action>
</actions>

```

## 第四步 创建一个 Action 展示 PSI 信息

### PSI 简介

程序结构接口(Program Structure Interface)，通常简称为PSI。
是IntelliJ平台中负责解析文件和创建语法和语义代码模型的层。

- PSI 文件

PSI 文件是一个程序逻辑结构的根，该结构将文件的内容表示为一种特定编程语言中的元素的层次结构。
**PsiFile** 类是所有 PSI 文件的通用基类，而特定语言的文件通常由其子类表示。
例如，**PsiJavaFile** 类表示一个Java文件，而 **XmlFile** 类表示一个XML文件

- PSI 元素

一个 PSI 文件代表了 PSI 元素（也叫 PSI 树）的层次结构。
PSI 元素表示源代码的内部结构，是由 IntelliJ 平台解析的。
在 PSI 元素上的操作一般用于代码分析，代码检查等。

**PsiElement** 类是 PSI 元素的通用基类。

有关 PSI 的更多介绍，参见 [PSI](https://plugins.jetbrains.com/docs/intellij/psi.html)

### 展示 PSI 信息

- com.dmall.rdp.plugin.demo.action.PsiDemoAction

```java
/**
 * PSI 演示 Action，展示光标所处位置上的 PSI 元素信息。
 *
 */
public class PsiDemoAction extends AnAction {
    @Override
    public void actionPerformed(AnActionEvent anActionEvent) {
        //获取当前编辑器和 PSIFile 对象
        Editor editor = anActionEvent.getData(CommonDataKeys.EDITOR);
        PsiFile psiFile = anActionEvent.getData(CommonDataKeys.PSI_FILE);
        if (editor == null || psiFile == null) {
            return;
        }
        //获取光标在文档中的偏移量
        int offset = editor.getCaretModel().getOffset();
        final StringBuilder infoBuilder = new StringBuilder();
        //获取光标所在位置的 PSI 树中的元素
        PsiElement element = psiFile.findElementAt(offset);
        infoBuilder.append("当前光标所指元素: ").append(element).append("\n");
        if (element != null) {
            //查找 方法
            PsiMethod containingMethod = PsiTreeUtil.getParentOfType(element, PsiMethod.class);
            infoBuilder
                    .append("方法: ")
                    .append(containingMethod != null ? containingMethod.getName() : "none")
                    .append("\n");
            if (containingMethod != null) {
                //查找方法所属的类
                PsiClass containingClass = containingMethod.getContainingClass();
                infoBuilder
                        .append("类: ")
                        .append(containingClass != null ? containingClass.getName() : "none")
                        .append("\n");
                //查找方法中的本地变量
                infoBuilder.append("本地变量:\n");
                containingMethod.accept(new JavaRecursiveElementVisitor() {
                    @Override
                    public void visitLocalVariable(PsiLocalVariable variable) {
                        super.visitLocalVariable(variable);
                        infoBuilder.append(variable.getName()).append("\n");
                    }
                });
            }
        }
        Messages.showMessageDialog(anActionEvent.getProject(), infoBuilder.toString(), "PSI信息", null);
    }

    @Override
    public void update(AnActionEvent e) {
        Editor editor = e.getData(CommonDataKeys.EDITOR);
        PsiFile psiFile = e.getData(CommonDataKeys.PSI_FILE);
        e.getPresentation().setEnabled(editor != null && psiFile != null);
    }
}
```
- src/main/resources/META-INF/plugin.xml

```xml

    <!-- 将此 Action 放在工具菜单的最后一个位置; 如果项目和编辑器是打开的，它总是可用的 -->
    <action
            id="com.dmall.rdp.plugin.demo.action.PsiDemoAction"
            class="com.dmall.rdp.plugin.demo.action.PsiDemoAction"
            text="查看PSI信息 ">
        <!-- 放到工具菜单第后一个位置 -->
        <add-to-group group-id="ToolsMenu" anchor="last"/>
    </action>

```

## 第五步运行插件，查看效果

点击 Run Plugin 或 Debug Plugin 运行插件，试着操作，查看效果。

![img.png](/images/idea/img.png)

![psi_review.png](/images/idea/psi_review.png)

## 一些特别有用的参考

下面收集的文档对开发插件是非常有帮助的，建议深入熟悉。

- 开发及API文档参考 [Intellij 平台官方文档](https://plugins.jetbrains.com/docs/intellij/intellij-platform.html)
- 入门参考 [Intellij SDK代码示例](https://github.com/JetBrains/intellij-sdk-code-samples)
- 插件高级应用参考 [Intellij IDEA 社区版源码](https://github.com/JetBrains/intellij-community)
- 插件高级应用参考 [IntelliJ IDEA Ultimate和其它基于IntelliJ平台的IDE发行版中包含的开源插件源码](https://github.com/JetBrains/intellij-plugins)
- 代码质量检查类插件参考 [阿里代码规范检查插件源码](https://github.com/alibaba/p3c)
- 插件开发必备 [PSIViewer 查看PSI结构的插件](https://plugins.jetbrains.com/plugin/227-psiviewer)
- 代码辅助生成类插件参考 [Lombok插件源码](https://github.com/mplushnikov/lombok-intellij-plugin)
- 搜索扩展 Intellij IDEA 平台功能扩展点的开源实现代码，相当有用，能给你灵感 [Intellij Platform Explorer](https://plugins.jetbrains.com/intellij-platform-explorer)
