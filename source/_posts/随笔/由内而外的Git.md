title: 由内而外的Git
author: 追梦人

toc: true

tags:

  - Git
  - 版本控制工具
  - 随笔
categories: []
date: 2020-02-21 16:57:00

---

# 由内而外的Git

本文转载自[Mary Rose Cook](https://codewords.recurse.com/about#Mary Rose Cook)，原文链接[点击这儿](https://codewords.recurse.com/issues/two/git-from-the-inside-out)。转载仅用于学习。

本文介绍了Git的工作原理。假定您足够了解Git，可以使用它对项目进行版本控制。

本文着重于支撑Git的图结构以及该图的属性指示Git行为的方式。从基础上看，您的思维模型是建立在事实之上，而不是根据在尝试API时收集的证据构建的假设。这个更真实的模型使您可以更好地了解Git所做的事情，正在做的事情以及它将做的事情。

文本的结构是在单个项目上运行的一系列Git命令。有时会观察到有关构建Git的图形数据结构的信息。这些观察结果说明了图形的属性以及该属性产生的行为。

阅读后，如果您想更深入地研究Git，可以查看我在JavaScript中实现Git的[注释严重的源代码](http://gitlet.maryrosecook.com/docs/gitlet.html)。

<!-- more -->

## 创建项目

```
~ $ mkdir alpha
~ $ cd alpha
```

用户`alpha`为项目创建一个目录。

```
~/alpha $ mkdir data
~/alpha $ printf 'a' > data/letter.txt
```

他们进入`alpha`目录并创建一个名为的目录`data`。在内部，他们创建了一个名为的文件`letter.txt`，其中包含`a`。alpha目录如下所示：

```
alpha
└── data
    └── letter.txt
```

## 初始化存储库

```
~/alpha $ git init
```

`git init`将当前目录放入Git存储库。为此，它将创建`.git`目录并向其中写入一些文件。这些文件定义了有关Git配置和项目历史的所有内容。它们只是普通文件。他们没有魔术。用户可以使用文本编辑器或外壳读取和编辑它们。也就是说：用户可以像编辑项目文件一样轻松地读取和编辑其历史记录。

`alpha`现在，目录如下所示：

```
alpha
├── data
|   └── letter.txt
└── .git
    ├── objects
    etc...
```

该`.git`目录及其内容是Git的。所有其他文件统称为工作副本。它们是用户的。

## 添加一些文件

```
~/alpha $ git add data/letter.txt
```

用户在`git add`上运行`data/letter.txt`。这有两个效果。

首先，它在`.git/objects/`目录中创建一个新的Blob文件。

此Blob文件包含的压缩内容`data/letter.txt`。它的名称是通过散列其内容而得出的。散列一段文字意味着在其上运行一个程序，将其转换为较小的[1](https://codewords.recurse.com/issues/two/git-from-the-inside-out#fn:1)条文字，该文字可以唯一地[2](https://codewords.recurse.com/issues/two/git-from-the-inside-out#fn:2)标识原始文字。例如，Git散列`a`为`2e65efe2a145dda7ee51d1741299f848e5bf752e`。前两个字符用作对象数据库中目录的名称：`.git/objects/2e/`。哈希的其余部分用作保存添加文件内容的Blob文件的名称：`.git/objects/2e/65efe2a145dda7ee51d1741299f848e5bf752e`。

注意仅将文件添加到Git如何将其内容保存到`objects`目录中。如果用户`data/letter.txt`从工作副本中删除，其内容在Git中仍然是安全的。

其次，`git add`将文件添加到索引。索引是一个列表，其中包含Git已被告知要跟踪的每个文件。它以文件形式存储在`.git/index`。文件的每一行都会在添加文件时将跟踪的文件映射到其内容的哈希。这是`git add`命令运行后的索引：

```
data/letter.txt 2e65efe2a145dda7ee51d1741299f848e5bf752e
```

用户创建一个名为的文件`data/number.txt`，其中包含`1234`。

```
~/alpha $ printf '1234' > data/number.txt
```

工作副本如下所示：

```
alpha
└── data
    └── letter.txt
    └── number.txt
```

用户将文件添加到Git。

```
~/alpha $ git add data
```

该`git add`命令将创建一个Blob对象，其中包含的内容`data/number.txt`。它为`data/number.txt`斑点处的该点添加了索引条目。这是`git add`第二次运行命令后的索引：

```
data/letter.txt 2e65efe2a145dda7ee51d1741299f848e5bf752e
data/number.txt 274c0052dd5408f8ae2bc8440029ff67d79bc5c3
```

请注意`data`，尽管用户已运行，但索引中仅列出了目录中的文件`git add data`。该`data`目录未单独列出。

```
~/alpha $ printf '1' > data/number.txt
~/alpha $ git add data
```

当用户最初创建时`data/number.txt`，他们的意思是键入`1`，而不是`1234`。他们进行更正并将文件再次添加到索引中。此命令使用新内容创建一个新的Blob。并更新索引条目`data/number.txt`以指向新的Blob。

## 提交

```
~/alpha $ git commit -m 'a1'
          [master (root-commit) 774b54a] a1
```

用户进行`a1`提交。Git打印有关提交的一些数据。这些数据很快就会有意义。

commit命令包含三个步骤。它创建一个树形图来表示要提交的项目版本的内容。它创建一个提交对象。它将当前分支指向新的提交对象。

### 创建树形图

Git通过从索引创建树形图来记录项目的当前状态。该树形图记录了项目中每个文件的位置和内容。

该图由两种类型的对象组成：斑点和树。

斑点由储存`git add`。它们代表文件的内容。

进行提交时将存储树。树表示工作副本中的目录。

以下是树对象，该树对象记录`data`了新提交的目录内容：

```
100664 blob 2e65efe2a145dda7ee51d1741299f848e5bf752e letter.txt
100664 blob 56a6051ca2b02b04ef92d5150c9ef600403cb1de number.txt
```

第一行记录了复制所需的一切`data/letter.txt`。第一部分说明文件的权限。第二部分指出该条目的内容由blob（而不是树）表示。第三部分说明了Blob的哈希值。第四部分说明文件的名称。

第二行记录`data/number.txt`。

以下是的树对象`alpha`，它是项目的根目录：

```
040000 tree 0eed1217a2947f4930583229987d90fe5e8e0b74 data
```

该树中的唯一行指向该`data`树。

![提交“ a1”的树形图](https://codewords.recurse.com/images/two/git-from-the-inside-out/1-a1-tree-graph.png)

提交“ a1”的树形图

在上图中，`root`树指向`data`树。将`data`在斑点的树点`data/letter.txt`和`data/number.txt`。

### 创建一个提交对象

`git commit`在创建树形图之后创建一个提交对象。commit对象只是位于的另一个文本文件`.git/objects/`：

```
tree ffe298c3ce8bb07326f888907996eaa48d266db4
author Mary Rose Cook <mary@maryrosecook.com> 1424798436 -0500
committer Mary Rose Cook <mary@maryrosecook.com> 1424798436 -0500

a1
```

第一行指向树形图。哈希用于代表工作副本根目录的树对象。即：`alpha`目录。最后一行是提交消息。

![`a1`提交对象指向其树形图](https://codewords.recurse.com/images/two/git-from-the-inside-out/2-a1-commit.png)

`a1`提交对象指向其树形图

### 将当前分支指向新提交

最后，commit命令将当前分支指向新的提交对象。

当前是哪个分支？Git转至位于的`HEAD`文件，`.git/HEAD`并发现：

```
ref: refs/heads/master
```

这就是说`HEAD`指向`master`。 `master`是当前分支。

`HEAD`并且`master`都是裁判。ref是Git或用户用来标识特定提交的标签。

表示`master`引用的文件不存在，因为这是对存储库的首次提交。Git在以下位置创建文件，`.git/refs/heads/master`并将其内容设置为提交对象的哈希值：

```
74ac3ad9cde0b265d2b4f1c778b283a6e2ffbafd
```

（如果您在阅读时输入这些Git命令，则`a1`提交的哈希将与我的哈希不同。斑点对象和树之类的内容对象始终会哈希为相同的值。提交不会，因为它们包括日期和其创建者的姓名。）

让我们增加`HEAD`和`master`到Git的图形：

![指向`a1`提交的`master`](https://codewords.recurse.com/images/two/git-from-the-inside-out/3-a1-refs.png)

指向`master`的`HEAD`和指向`a1` commit的`master`

`HEAD`指向`master`，就像提交之前一样。但是`master`现在存在并指向新的提交对象。

## 进行不是第一次提交的提交

下面是`a1`提交后的Git图。包括工作副本和索引。

![带有工作副本和索引的`a1`提交](https://codewords.recurse.com/images/two/git-from-the-inside-out/4-a1-wc-and-index.png)

带有工作副本和索引的`a1`提交

请注意，工作副本，指数，并`a1`提交所有有相同的内容`data/letter.txt`和`data/number.txt`。索引和`HEAD`提交都使用散列来引用blob对象，但是工作副本内容作为文本存储在不同的位置。

```
~/alpha $ printf '2' > data/number.txt
```

用户将的内容设置`data/number.txt`为`2`。这将更新工作副本，但保留索引并按`HEAD`原样提交。

![工作副本中的“ data / number.txt”设置为“ 2”](https://codewords.recurse.com/images/two/git-from-the-inside-out/5-a1-wc-number-set-to-2.png)

工作副本中的“ data / number.txt”设置为“ 2”

```
~/alpha $ git add data/number.txt
```

用户将文件添加到Git。这会将包含的blob添加`2`到`objects`目录中。它将索引条目指向`data/number.txt`新的Blob。

![在工作副本和索引中将“ data / number.txt”设置为“ 2”](https://codewords.recurse.com/images/two/git-from-the-inside-out/6-a1-wc-and-index-number-set-to-2.png)

在工作副本和索引中将“ data / number.txt”设置为“ 2”

```
~/alpha $ git commit -m 'a2'
          [master f0af7e6] a2
```

用户提交。提交步骤与之前相同。

首先，创建一个新的树形图来表示索引的内容。

的索引条目`data/number.txt`已更改。旧`data`树不再反映`data`目录的索引状态。`data`必须创建一个新的树对象：

```
100664 blob 2e65efe2a145dda7ee51d1741299f848e5bf752e letter.txt
100664 blob d8263ee9860594d2806b0dfd1bfd17528b0ba2a4 number.txt
```

新`data`树的哈希值与老树的哈希值不同`data`。`root`必须创建一棵新树来记录此哈希：

```
040000 tree 40b0318811470aaacc577485777d7a6780e51f0b data
```

其次，创建一个新的提交对象。

```
tree ce72afb5ff229a39f6cce47b00d1b0ed60fe3556
parent 774b54a193d6cfdd081e581a007d2e11f784b9fe
author Mary Rose Cook <mary@maryrosecook.com> 1424813101 -0500
committer Mary Rose Cook <mary@maryrosecook.com> 1424813101 -0500

a2
```

提交对象的第一行指向新的`root`树对象。第二行指向`a1`：提交的父级。为了找到父提交，Git转到`HEAD`，然后跟随到`master`并找到的提交哈希`a1`。

第三，将`master`分支文件的内容设置为新提交的哈希值。

![`a2`提交](https://codewords.recurse.com/images/two/git-from-the-inside-out/7-a2.png)

`a2`提交

![没有工作副本和索引的Git图](https://codewords.recurse.com/images/two/git-from-the-inside-out/8-a2-just-objects-commits-and-refs.png)

没有工作副本和索引的Git图

**Graph属性**：内容存储为对象树。这意味着仅差异存储在对象数据库中。看上面的图。该`a2`承诺重用`a`挖成的BLOB之前`a1`提交。同样，如果整个目录在提交之间没有变化，则它的树以及它下面的所有Blob和树都可以重用。通常，从提交到提交的内容更改很少。这意味着Git可以在少量空间中存储大量提交历史记录。

**Graph属性**：每个提交都有一个父级。这意味着存储库可以存储项目的历史记录。

**Graph属性**：refs是提交历史记录的一部分或另一部分的入口点。这意味着可以为提交赋予有意义的名称。用户使用诸如的具体参考将其工作组织成对他们的项目有意义的世系`fix-for-bug-376`。Git使用象征性的裁判一样`HEAD`，`MERGE_HEAD`并`FETCH_HEAD`于操纵提交历史支持的命令。

**Graph属性**：`objects/`目录中的节点是不可变的。这意味着内容是被编辑而不是被删除。`objects`目录中添加的每条内容和提交的每一次提交都位于目录[3中](https://codewords.recurse.com/issues/two/git-from-the-inside-out#fn:3)。

**Graph属性**：引用是可变的。因此，ref的含义可以改变。`master`指向的提交可能是当前项目的最佳版本，但是很快，它将被更新更好的提交所取代。

**Graph属性**：ref指向的工作副本和提交是随时可用的，而其他提交则不可用。这意味着更容易回忆起最近的历史记录，但它的更改频率也更高。或者：Git具有逐渐消失的记忆，必须与越来越恶性的产品混为一谈。

工作副本是历史上最容易回忆的点，因为它位于存储库的根目录中。调用它甚至不需要Git命令。这也是历史上最不固定的一点。用户可以制作文件的多个版本，但是除非添加了文件，否则Git不会记录其中的任何一个。

`HEAD`指向的提交很容易回忆。检出的是分支的顶端。要查看其内容，用户只需隐藏[4](https://codewords.recurse.com/issues/two/git-from-the-inside-out#fn:4)，然后检查工作副本即可。同时，`HEAD`是变化最频繁的参考。

具体引用指向的提交很容易回忆。用户可以简单地签出该分支。分支的提示更改的频率比少`HEAD`，但更改频率足以使分支名称的含义改变。

很难回忆起没有任何引用指向的提交。用户离引用越远，他们构造提交含义的难度就越大。但是他们走得越远，自从上次看[5](https://codewords.recurse.com/issues/two/git-from-the-inside-out#fn:5)以来某人改变历史的可能性就越小。

## 签出提交

```
~/alpha $ git checkout 37888c2
          You are in 'detached HEAD' state...
```

用户`a2`使用其哈希检出提交。（如果您正在运行这些Git命令，则此命令将不起作用。`git log`用于查找`a2`提交的哈希。）

签出有四个步骤。

首先，Git获取`a2`提交并获取其指向的树图。

其次，它将树图中的文件条目写入工作副本。这不会导致任何变化。工作副本已经拥有了树图的内容被写入到它，因为`HEAD`通过已经指向`master`在`a2`提交。

第三，Git将树形图中的文件条目写入索引。这也不会导致任何变化。索引已经具有`a2`提交的内容。

第四，的内容`HEAD`设置为`a2`提交的哈希值：

```
f0af7e62679e144bb28c627ee3e8f7bdb235eee9
```

将的内容设置`HEAD`为哈希将使存储库处于分离`HEAD`状态。请注意，在下图中，`HEAD`该`a2`指针直接指向提交，而不是指向`master`。

![在`a2` commit上分离的`HEAD`](https://codewords.recurse.com/images/two/git-from-the-inside-out/9-a2-detached-head.png)

在`a2` commit上分离的`HEAD`

```
~/alpha $ printf '3' > data/number.txt
~/alpha $ git add data/number.txt
~/alpha $ git commit -m 'a3'
          [detached HEAD 3645a0e] a3
```

用户将的内容设置`data/number.txt`为`3`并提交更改。Git转到`HEAD`获取`a3`提交的父级。它查找并返回`a2`提交的哈希值，而不是查找并跟随分支引用。

Git更新`HEAD`以直接指向新`a3`提交的哈希值。存储库仍处于分离`HEAD`状态。它不在分支上，因为没有提交指向`a3`其后代中的一个或一个。这意味着很容易丢失。

从现在开始，树图和斑点将大部分从图表中省略。

![不在分支上的`a3`提交](https://codewords.recurse.com/images/two/git-from-the-inside-out/10-a3-detached-head.png)

不在分支上的`a3`提交

## 创建一个分支

```
~/alpha $ git branch deputy
```

用户创建一个名为的新分支`deputy`。这只是创建一个新文件，`.git/refs/heads/deputy`其中包含`HEAD`指向的哈希：`a3`提交的哈希。

**Graph属性**：分支仅是引用，而引用仅是文件。这意味着Git分支是轻量级的。

`deputy`分支的创建将新`a3`提交安全地放在分支上。`HEAD`仍然是分离的，因为它仍然直接指向提交。

![`a3`现在在`deputy`分支上提交](https://codewords.recurse.com/images/two/git-from-the-inside-out/11-a3-on-deputy.png)

`a3`现在在`deputy`分支上提交

## 签出一个分支

```
~/alpha $ git checkout master
          Switched to branch 'master'
```

用户签出`master`分支。

首先，Git获取指向的`a2`提交，`master`并获取提交指向的树形图。

其次，Git将树状图中的文件条目写入工作副本的文件中。这会将的内容设置`data/number.txt`为`2`。

第三，Git将树形图中的文件条目写入索引。这会将条目更新为Blob `data/number.txt`的哈希`2`。

第四，Git的点`HEAD`在`master`由哈希改变它的内容：

```
ref: refs/heads/master
```

![`master`签出并指向`a2` commit](https://codewords.recurse.com/images/two/git-from-the-inside-out/12-a3-on-master-on-a2.png)

`master`签出并指向`a2` commit

## 签出与工作副本不兼容的分支

```
~/alpha $ printf '789' > data/number.txt
~/alpha $ git checkout deputy
          Your changes to these files would be overwritten
          by checkout:
            data/number.txt
          Commit your changes or stash them before you
          switch branches.
```

用户不小心将的内容设置`data/number.txt`为`789`。他们试图退房`deputy`。Git阻止签出。

`HEAD`点在`master`哪一点在`a2`哪里`data/number.txt`阅读`2`。`deputy`指向`a3`哪里`data/number.txt`阅读`3`。`data/number.txt`reads 的工作副本版本`789`。所有这些版本都是不同的，必须解决差异。

Git可以用`data/number.txt`检出的提交中的版本替换工作副本版本。但是，它不惜一切代价避免了数据丢失。

Git可以将工作副本版本与要签出的版本合并。但这很复杂。

因此，Git中止了结帐。

```
~/alpha $ printf '2' > data/number.txt
~/alpha $ git checkout deputy
          Switched to branch 'deputy'
```

用户注意到他们不小心进行了编辑`data/number.txt`，并将内容设置回`2`。他们`deputy`成功签出。

![副检票](https://codewords.recurse.com/images/two/git-from-the-inside-out/13-a3ondeputy.png)

副检票

## 合并祖先

```
~/alpha $ git merge master
          Already up-to-date.
```

用户合并`master`到中`deputy`。合并两个分支意味着合并两个提交。第一个提交`deputy`指向的是：接收者。第二个提交`master`指向：提交者。对于此合并，Git不执行任何操作。报告它是`Already up-to-date.`。

**Graph属性**：**图形中**的一系列提交被解释为对存储库内容所做的一系列更改。这意味着在合并中，如果给定提交是接收方提交的祖先，则Git将不执行任何操作。这些更改已被合并。

## 合并后代

```
~/alpha $ git checkout master
          Switched to branch 'master'
```

用户签出`master`。

![`master`签出并指向`a2` commit](https://codewords.recurse.com/images/two/git-from-the-inside-out/14-a3-on-master-on-a2.png)

`master`签出并指向`a2` commit

```
~/alpha $ git merge deputy
          Fast-forward
```

他们合并`deputy`为`master`。Git发现接收者的提交`a2`是提供者提交的祖先`a3`。它可以进行快速合并。

它获取给予者提交并获取其指向的树图。它将树图中的文件条目写入工作副本和索引。它`master`指向“快进” `a3`。

![来自`副`的`a3`提交被快速合并为`master`](https://codewords.recurse.com/images/two/git-from-the-inside-out/15-a3-on-master.png)

来自`副`的`a3`提交被快速合并为`master`

**Graph属性**：**图形中**的一系列提交被解释为对存储库内容所做的一系列更改。这意味着，在合并中，如果给予者是接收者的后代，则历史不会更改。已经有一系列提交来描述要进行的更改：提交方与接收方之间的提交顺序。但是，尽管Git历史记录没有更改，但Git图确实发生了更改。`HEAD`指向的具体引用已更新为指向给定者提交。

## 合并来自不同血统的两个提交

```
~/alpha $ printf '4' > data/number.txt
~/alpha $ git add data/number.txt
~/alpha $ git commit -m 'a4'
          [master 7b7bd9a] a4
```

用户将的内容设置`number.txt`为`4`，并将更改提交到`master`。

```
~/alpha $ git checkout deputy
          Switched to branch 'deputy'
~/alpha $ printf 'b' > data/letter.txt
~/alpha $ git add data/letter.txt
~/alpha $ git commit -m 'b3'
          [deputy 982dffb] b3
```

用户签出`deputy`。他们将的内容设置`data/letter.txt`为`b`并将更改提交到`deputy`。

![已将`a4`提交给`master`，将`b3`提交给`deputy`和`deputy`](https://codewords.recurse.com/images/two/git-from-the-inside-out/16-a4-b3-on-deputy.png)

已将`a4`提交给`master`，将`b3`提交给`deputy`和`deputy`

**图属性**：提交可以共享父母。这意味着可以在提交历史记录中创建新的血统。

**Graph属性**：提交可以有多个父级。这意味着可以通过具有两个父项的提交将单独的谱系加入：合并提交。

```
~/alpha $ git merge master -m 'b4'
          Merge made by the 'recursive' strategy.
```

用户合并`master`到中`deputy`。

Git发现接收者`b3`和给予者`a4`处于不同的世系。它进行合并提交。此过程包含八个步骤。

首先，Git将提供者提交的哈希值写入的文件`alpha/.git/MERGE_HEAD`。这个文件的存在告诉Git它正在合并中。

其次，Git查找基本提交：接收方和提供方提交具有共同点的最新祖先。

![a3，a4和b3的基本提交](https://codewords.recurse.com/images/two/git-from-the-inside-out/17-a4-b3-on-deputy.png)

a3，a4和b3的基本提交

**图属性**：提交有父母。这意味着可以找到两个谱系发散的点。Git从追溯到`b3`找到其所有祖先，从追溯到找到其`a4`所有祖先。它找到两个谱系共享的最新祖先`a3`。这是基本提交。

第三，Git从其树形图生成基础，接收方和给予者提交的索引。

第四，Git生成一个差异，该差异将接收者提交和给予者提交对基础所做的更改组合在一起。该差异是指向更改的文件路径的列表：添加，删除，修改或冲突。

Git获取出现在基本，接收者或给予者索引中的所有文件的列表。对于每个索引，它都会比较索引条目，以决定要对该文件进行的更改。它将相应的条目写入差异。在这种情况下，差异有两个条目。

第一个条目用于`data/letter.txt`。该文件的内容`a`位于基础，`b`接收方和`a`提供方中。基准站和接收方的内容不同。但这在基础和给予者中都是一样的。Git看到内容是由接收者而非提供者修改的。的diff条目`data/letter.txt`是修改，而不是冲突。

diff中的第二个条目是for `data/number.txt`。在这种情况下，内容在基础库和接收方中是相同的，在提供者中是不同的。的diff条目`data/letter.txt`也是一个修改。

**Graph属性**：可以找到合并的基本提交。这意味着，如果仅在接收方或给予者中从基础更改了文件，则Git可以自动解析该文件的合并。这减少了用户必须做的工作。

第五，差异中的条目所指示的更改将应用于工作副本。的内容`data/letter.txt`设置为，`b`而的内容`data/number.txt`设置为`4`。

第六，将差异中的条目指示的更改应用于索引。for的条目`data/letter.txt`指向`b`blob，for的条目`data/number.txt`指向`4`blob。

第七，提交更新的索引：

```
tree 20294508aea3fb6f05fcc49adaecc2e6d60f7e7d
parent 982dffb20f8d6a25a8554cc8d765fb9f3ff1333b
parent 7b7bd9a5253f47360d5787095afc5ba56591bfe7
author Mary Rose Cook <mary@maryrosecook.com> 1425596551 -0500
committer Mary Rose Cook <mary@maryrosecook.com> 1425596551 -0500

b4
```

请注意，提交有两个父母。

第八，Git将当前分支指向`deputy`新提交。

![b4，是由a4递归合并到b3中而产生的合并提交](https://codewords.recurse.com/images/two/git-from-the-inside-out/18-b4-on-deputy.png)

b4，是由a4递归合并到b3中而产生的合并提交

## 合并来自不同谱系的两个提交，这两个提交都修改同一文件

```
~/alpha $ git checkout master
          Switched to branch 'master'
~/alpha $ git merge deputy
          Fast-forward
```

用户签出`master`。他们合并`deputy`为`master`。这种快进`master`的`b4`承诺。`master`而`deputy`现在点同时提交。

![代理合并为master，使master达到最新的提交b4。](https://codewords.recurse.com/images/two/git-from-the-inside-out/19-b4-master-deputy-on-b4.png)

代理合并为master，使master达到最新的提交b4。

```
~/alpha $ git checkout deputy
          Switched to branch 'deputy'
~/alpha $ printf '5' > data/number.txt
~/alpha $ git add data/number.txt
~/alpha $ git commit -m 'b5'
          [deputy bd797c2] b5
```

用户签出`deputy`。他们将的内容设置`data/number.txt`为`5`并将更改提交到`deputy`。

```
~/alpha $ git checkout master
          Switched to branch 'master'
~/alpha $ printf '6' > data/number.txt
~/alpha $ git add data/number.txt
~/alpha $ git commit -m 'b6'
          [master 4c3ce18] b6
```

用户签出`master`。他们将的内容设置`data/number.txt`为`6`并将更改提交到`master`。

![`b5`提交给`代理`和`b6`提交给`master`](https://codewords.recurse.com/images/two/git-from-the-inside-out/20-b5-on-deputy-b6-on-master.png)

```
b5`提交给`代理`和`b6`提交给`master
~/alpha $ git merge deputy
          CONFLICT in data/number.txt
          Automatic merge failed; fix conflicts and
          commit the result.
```

用户合并`deputy`到中`master`。发生冲突，合并被暂停。发生冲突的合并的过程遵循与进行无冲突的合并的相同的前六个步骤：set `.git/MERGE_HEAD`，查找基本提交，生成基本提交的索引，接收方和给予者提交，创建差异，更新工作副本并更新指数。由于冲突，从不执行第七个提交步骤和第八个ref更新步骤。让我们再次执行步骤，看看会发生什么。

首先，Git将提供者提交的哈希值写入的文件`.git/MERGE_HEAD`。

![在将b5合并为b6期间写入的MERGE_HEAD](https://codewords.recurse.com/images/two/git-from-the-inside-out/21-b6-on-master-with-merge-head.png)

在将b5合并为b6期间写入的MERGE_HEAD

其次，Git找到基本提交`b4`。

第三，Git为基础，接收方和给予者提交生成索引。

第四，Git生成一个差异，该差异将接收者提交和给予者提交对基础所做的更改组合在一起。该差异是指向更改的文件路径的列表：添加，删除，修改或冲突。

在这种情况下，差异仅包含一个条目：`data/number.txt`。该条目被标记为冲突，因为`data/number.txt`接收方，给予方和基准的内容不同。

第五，差异中的条目所指示的更改将应用于工作副本。对于有冲突的区域，Git会将这两个版本都写入工作副本中的文件中。的内容`data/number.txt`设置为：

```
<<<<<<< HEAD
6
=======
5
>>>>>>> deputy
```

第六，将差异中的条目指示的更改应用于索引。索引中的条目由其文件路径和阶段的组合唯一标识。未冲突文件的条目的阶段为`0`。在合并之前，索引看起来像这样，其中`0`s是阶段值：

```
0 data/letter.txt 63d8dbd40c23542e740659a7168a0ce3138ea748
0 data/number.txt 62f9457511f879886bb7728c986fe10b0ece6bcb
```

将合并差异写入索引后，索引如下所示：

```
0 data/letter.txt 63d8dbd40c23542e740659a7168a0ce3138ea748
1 data/number.txt bf0d87ab1b2b0ec1a11a3973d2845b42413d9767
2 data/number.txt 62f9457511f879886bb7728c986fe10b0ece6bcb
3 data/number.txt 7813681f5b41c028345ca62a2be376bae70b7f61
```

`data/letter.txt`阶段的条目`0`与合并之前的条目相同。`data/number.txt`阶段的条目`0`已消失。在它的位置有三个新条目。阶段的条目`1`具有基本`data/number.txt`内容的哈希值。阶段的条目`2`具有接收者`data/number.txt`内容的哈希值。阶段的条目`3`具有提供者`data/number.txt`内容的哈希值。这三个条目的存在告诉Git `data/number.txt`发生冲突。

合并暂停。

```
~/alpha $ printf '11' > data/number.txt
~/alpha $ git add data/number.txt
```

用户通过将的内容设置为`data/number.txt`来整合两个冲突版本的内容`11`。他们将文件添加到索引。Git添加了一个包含的Blob `11`。添加有冲突的文件会告诉Git冲突已解决。Git的删除`data/number.txt`的阶段条目`1`，`2`并`3`从索引。它使用新Blob的哈希值`data/number.txt`在阶段添加一个条目`0`。索引现在显示为：

```
0 data/letter.txt 63d8dbd40c23542e740659a7168a0ce3138ea748
0 data/number.txt 9d607966b721abde8931ddd052181fae905db503
~/alpha $ git commit -m 'b11'
          [master 251a513] b11
```

第七，用户提交。Git `.git/MERGE_HEAD`在存储库中看到，它告诉它正在进行合并。它检查索引，发现没有冲突。它创建一个新的提交，`b11`以记录已解析合并的内容。它将删除文件`.git/MERGE_HEAD`。这样就完成了合并。

第八，Git将当前分支指向`master`新提交。

![b11，是由b5冲突，递归合并到b6而产生的合并提交](https://codewords.recurse.com/images/two/git-from-the-inside-out/22-b11-on-master.png)

b11，是由b5冲突，递归合并到b6而产生的合并提交

## 移除档案

Git图的此图包括提交历史记录，最新提交的树和Blob以及工作副本和索引：

![工作副本，索引，`b11`提交及其树形图](https://codewords.recurse.com/images/two/git-from-the-inside-out/23-b11-with-objects-wc-and-index.png)

工作副本，索引，`b11`提交及其树形图

```
~/alpha $ git rm data/letter.txt
          rm 'data/letter.txt'
```

用户告诉Git删除`data/letter.txt`。该文件将从工作副本中删除。该条目将从索引中删除。

![从工作副本和索引中取出“ data / letter.txt” rm后](https://codewords.recurse.com/images/two/git-from-the-inside-out/24-b11-letter-removed-from-wc-and-index.png)

从工作副本和索引中取出“ data / letter.txt” rm后

```
~/alpha $ git commit -m '11'
          [master d14c7d2] 11
```

用户提交。作为提交的一部分，Git一如既往地构建一个树形图，该树形图表示索引的内容。`data/letter.txt`不包含在树形图中，因为它不在索引中。

![在`data / letter.txt`rm`编辑后进行`11`提交](https://codewords.recurse.com/images/two/git-from-the-inside-out/25-11.png)

在`data / letter.txt`rm`编辑后进行`11`提交

## 复制存储库

```
~/alpha $ cd ..
      ~ $ cp -R alpha bravo
```

用户将`alpha/`存储库的内容复制到`bravo/`目录中。这将产生以下目录结构：

```
~
├── alpha
|   └── data
|       └── number.txt
└── bravo
    └── data
        └── number.txt
```

现在`bravo`目录中还有另一个Git图：

![当alpha cp改为bravo时创建的新图形](https://codewords.recurse.com/images/two/git-from-the-inside-out/26-11-cp-alpha-to-bravo.png)

当alpha cp改为bravo时创建的新图形

## 将存储库链接到另一个存储库

```
      ~ $ cd alpha
~/alpha $ git remote add bravo ../bravo
```

用户移回`alpha`存储库。他们在上设置`bravo`为远程存储库`alpha`。这会向文件添加一些行`alpha/.git/config`：

```
[remote "bravo"]
	url = ../bravo/
```

这些行指定`bravo`在目录中有一个称为的远程存储库`../bravo`。

## 从远程获取分支

```
~/alpha $ cd ../bravo
~/bravo $ printf '12' > data/number.txt
~/bravo $ git add data/number.txt
~/bravo $ git commit -m '12'
          [master 94cd04d] 12
```

用户进入`bravo`存储库。他们设定的内容`data/number.txt`来`12`并提交变更`master`上`bravo`。

![在`bravo`仓库上的`12` commit](https://codewords.recurse.com/images/two/git-from-the-inside-out/27-12-bravo.png)

在`bravo`仓库上的`12` commit

```
~/bravo $ cd ../alpha
~/alpha $ git fetch bravo master
          Unpacking objects: 100%
          From ../bravo
            * branch master -> FETCH_HEAD
```

用户进入`alpha`存储库。他们获取`master`从`bravo`成`alpha`。此过程分为四个步骤。

首先，Git获取master指向的提交的哈希值`bravo`。这是`12`提交的哈希值。

其次，Git列出了`12`提交所依赖的所有对象的列表：提交对象本身，其树形图中的对象，该提交的祖先`12`提交以及它们的树形图中的对象。它从该列表中删除`alpha`对象数据库已具有的所有对象。它将其余部分复制到`alpha/.git/objects/`。

第三，将具体参考文件at的`alpha/.git/refs/remotes/bravo/master`内容设置为`12`提交的哈希值。

第四，的内容`alpha/.git/FETCH_HEAD`设置为：

```
94cd04d93ae88a1f53a4646532b1e8cdfbc0977f branch 'master' of ../bravo
```

这表明最新的提取命令从中提取了`12`提交。`master``bravo`

![在获取`bravo / master`之后的`alpha`](https://codewords.recurse.com/images/two/git-from-the-inside-out/28-12-fetched-to-alpha.png)

在获取`bravo / master`之后的`alpha`

**Graph属性**：可以复制对象。这意味着可以在存储库之间共享历史记录。

**Graph属性**：存储库可以存储类似的远程分支引用`alpha/.git/refs/remotes/bravo/master`。这意味着存储库可以在本地记录远程存储库上分支的状态。它在获取时是正确的，但是如果远程分支发生更改，它将过时。

## 合并FETCH_HEAD

```
~/alpha $ git merge FETCH_HEAD
          Updating d14c7d2..94cd04d
          Fast-forward
```

用户合并`FETCH_HEAD`。`FETCH_HEAD`只是另一个参考。它解析为`12`提交者，即给予者。`HEAD`指向`11`提交，接收者。Git的的确快进合并和点`master`的`12`承诺。

![`FETCH_HEAD`合并后的`alpha`](https://codewords.recurse.com/images/two/git-from-the-inside-out/29-12-merged-to-alpha.png)

```
FETCH_HEAD`合并后的`alpha
```

## 从遥控器拉分支

```
~/alpha $ git pull bravo master
          Already up-to-date.
```

用户`master`从`bravo`进入`alpha`。Pull是“获取并合并`FETCH_HEAD`”的简写。Git执行这两个命令并报告`master`为`Already up-to-date`。

## 克隆存储库

```
~/alpha $ cd ..
      ~ $ git clone alpha charlie
          Cloning into 'charlie'
```

用户移到上面的目录。他们克隆`alpha`到`charlie`。克隆到`charlie`具有与`cp`用户生成`bravo`存储库相似的结果。Git创建一个名为的新目录`charlie`。它`charlie`作为Git存储库初始化，`alpha`作为远程调用添加`origin`，获取`origin`并合并`FETCH_HEAD`。

## 将分支推送到遥控器上的已签出分支

```
      ~ $ cd alpha
~/alpha $ printf '13' > data/number.txt
~/alpha $ git add data/number.txt
~/alpha $ git commit -m '13'
          [master 3238468] 13
```

用户返回到`alpha`存储库。他们设定的内容`data/number.txt`来`13`并提交变更`master`上`alpha`。

```
~/alpha $ git remote add charlie ../charlie
```

他们在上设置`charlie`为远程存储库`alpha`。

```
~/alpha $ git push charlie master
          Writing objects: 100%
          remote error: refusing to update checked out
          branch: refs/heads/master because it will make
          the index and work tree inconsistent
```

他们推`master`到`charlie`。

`13`提交所需的所有对象都复制到`charlie`。

此时，推送过程停止。Git一如既往地告诉用户出了什么问题。它拒绝推送到在远程上签出的分支。这是有道理的。推送将更新远程索引和`HEAD`。如果有人在遥控器上编辑工作副本，这将引起混乱。

此时，用户可以创建一个新分支，将`13`提交合并到其中，然后将该分支推送到`charlie`。但是，实际上，他们想要一个可以随时随地推送的存储库。他们想要一个中央存储库，可以将其推入或拉出，但是没有人直接提交。他们想要像GitHub远程之类的东西。他们想要一个裸仓库。

## 克隆裸仓库

```
~/alpha $ cd ..
      ~ $ git clone alpha delta --bare
          Cloning into bare repository 'delta'
```

用户移到上面的目录。他们克隆`delta`为裸仓库。这是一个普通的克隆，有两个区别。该`config`文件表明存储库是裸露的。通常存储在`.git`目录中的文件存储在存储库的根目录中：

```
delta
├── HEAD
├── config
├── objects
└── refs
```

![将“ alpha”克隆为“ delta”后的“ alpha”和“ delta”图](https://codewords.recurse.com/images/two/git-from-the-inside-out/30-13-alpha-cloned-to-delta-bare.png)

将“ alpha”克隆为“ delta”后的“ alpha”和“ delta”图

## 将分支推送到裸仓库

```
      ~ $ cd alpha
~/alpha $ git remote add delta ../delta
```

用户返回到`alpha`存储库。他们在上设置`delta`为远程存储库`alpha`。

```
~/alpha $ printf '14' > data/number.txt
~/alpha $ git add data/number.txt
~/alpha $ git commit -m '14'
          [master cb51da8] 14
```

他们设定的内容`data/number.txt`来`14`并提交变更`master`上`alpha`。

![`14`在`alpha`上提交](https://codewords.recurse.com/images/two/git-from-the-inside-out/31-14-alpha.png)

`14`在`alpha`上提交

```
~/alpha $ git push delta master
          Writing objects: 100%
          To ../delta
            3238468..cb51da8 master -> master
```

他们推`master`到`delta`。推送有三个步骤。

首先，`14`将`master`分支上提交所需的所有对象都从复制`alpha/.git/objects/`到`delta/objects/`。

其次，`delta/refs/heads/master`更新为指向`14`提交。

第三，`alpha/.git/refs/remotes/delta/master`设置为指向`14`提交。`alpha`拥有的状态的最新记录`delta`。

![14提交从alpha推送到delta](https://codewords.recurse.com/images/two/git-from-the-inside-out/32-14-pushed-to-delta.png)

14提交从alpha推送到delta

## 摘要

Git建立在图形上。几乎每个Git命令都会操纵该图。要深入了解Git，请专注于此图的属性，而不是工作流或命令。

要了解有关Git的更多信息，请调查`.git`目录。这并不可怕。看看里面。更改文件的内容，然后看看会发生什么。手动创建一个提交。尝试看看您能使回购协议多么糟糕。然后修复它。

1. 在这种情况下，哈希值比原始内容长。但是，所有比散列中字符数长的内容都将比原始内容更简洁地表示。 [↩](https://codewords.recurse.com/issues/two/git-from-the-inside-out#fnref:1)
2. 两个不同的内容有可能会散列为相同的值。但这机会[很少](http://crypto.stackexchange.com/a/2584)。 [↩](https://codewords.recurse.com/issues/two/git-from-the-inside-out#fnref:2)
3. `git prune`删除引用中无法访问的所有对象。如果用户运行此命令，则他们可能会丢失内容。 [↩](https://codewords.recurse.com/issues/two/git-from-the-inside-out#fnref:3)
4. `git stash`将工作副本和`HEAD`提交之间的所有差异存储在安全的地方。以后可以检索它们。 [↩](https://codewords.recurse.com/issues/two/git-from-the-inside-out#fnref:4)
5. 该`rebase`命令可用于添加，编辑和删除历史记录中的提交。 [↩](https://codewords.recurse.com/issues/two/git-from-the-inside-out#fnref:5)

