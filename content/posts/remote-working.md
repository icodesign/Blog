---
title: "谈谈团队远程协作"
date: 2020-01-28T11:39:54+08:00
description: "Outlook Mobile 团队是一个全球团队，仅在美国就有Redmond，San Francisco 和 New York 分部。全球范围来说，有中国苏州，印度班加罗尔，以及分布在欧洲的 Home Office（在家办公），这样的分布基本涵盖了全球时区，更不用说其他伙伴团队分布在天南海北了。对于这样一个高度分布式的团队，势必会对团队合作带来一定挑战。"
images: 
- /images/remote-working/meetup.jpg
---

Outlook Mobile 团队是一个全球团队，仅在美国就有Redmond，San Francisco 和 New York 分部。全球范围来说，有中国苏州，印度班加罗尔，以及分布在欧洲的 Home Office（在家办公），这样的分布基本涵盖了全球时区，更不用说其他伙伴团队分布在天南海北了。对于这样一个高度分布式的团队，势必会对团队合作带来一定挑战。

# 工具链
人与动物最大的区别是能否制造和使用工具，而一定规模的团队能否顺利运转的一个关键是能否利用各种工具提高效率，要不然也不会存在那么多 To B 的公司了。这一点对于远程团队来说就显得更为重要了。

企业办公市场经过近些年的发展涌现出了一批优秀的工具软件，既有老牌的 Atlansian 全家桶，也有企业 IM 新秀 Slack，国内阿里巴巴、腾讯、字节跳动亦在这个领域开始发力。而选择什么样的工具，或者什么样的软件更适合团队是个见仁见智的问题，既取决于工具本身的易用性，也掺杂着决策者的偏好，又或是公司的整体战略。

当然，很多时候适不适合得试了才知道，我们团队在协作工具链方面也经过了一系列的迭代。

早期的时候：

| IM    | Email   | 文件共享   | 任务管理 | Wiki       | 代码托管 | 产品设计讨论  |
| ----- | ------- | ---------- | -------- | ---------- | -------- | ------------- |
| Slack | Outlook | Office 365 | Jira     | Confluence | GitHub   | GitHub Issues |

现在：

| IM    | Email   | 文件共享   | 任务管理      | Wiki          | 代码托管      | 产品设计讨论  |
| ----- | ------- | ---------- | ------------- | ------------- | ------------- | ------------- |
| Teams | Outlook | Office 365 | Azure Dev Ops | Azure Dev Ops | Azure Dev Ops | GitHub Issues |

明眼人都能看出现在的工具链更微软化了。在这里，我不想去讨论究竟哪个工具更好，正如我前面说到的，选择不分对错，只看合不合适。现在的工具链显然更适合微软这样的大公司中的一个团队。为什么？在更大规模的公司，我们要考虑的不仅仅是某一个团队的个性，更要考虑公司整体上的协同。从公司范围内的代码共享、知识共享、沟通交流角度出发，基于同一套认证系统的工具链显然更为方便，权限管理也更为清晰。

回到最初的问题，我们是怎么利用工具来解决不同时区，远程合作的问题呢？

__邮件 & 日历__

![Email & Calendar](/images/remote-working/email_calendar.jpg)

邮件是不可或缺的一环，在微软工作之后我才知道工作用邮件是多么的重要，以至于我需要特定的方法论来处理每天的邮件，这个有兴趣可以以后再聊。在之前的公司，我并没有意识到这一点，因为绝大部分沟通都可以面对面进行，这也可能导致大家更喜欢用 IM 而不是邮件。

但这带来了一个巨大的问题，很多时候需要严肃的沟通变得很随意，对话和决定也缺少了深思熟虑。另一个问题就是归档和分享，当你需要重新寻找之前的讨论或者把当前讨论分享给其他同事的时候，IM 就变得非常困难。

对于全球团队来说，邮件的严肃性减少了很多噪音，第二天起来不需要去看冗长的无效对话，而且每一个话题都被归档在各自的会话中，极大地提高了效率。

我很佩服 Outlook 将日历和邮件放在一起的设计，这大大提高了办公环境以及远程协作场景下的效率。日历一目了然的呈现了预约、会议、面试、休假等时间安排，而且这些安排可以对公司内部的人可见。一来，你自己可以了解自己的繁忙程度，二来，别人来约时间就变得轻松无比了。

对于个人来说，如何利用邮件和日历更好的组织自己一定程度上决定着工作效率。这里也涉及到了如何处理邮件的哲学，有兴趣可以留言，我后面发文再聊。

__产品流程 & GitHub Issues__

![GitHub Issues](/images/remote-working/ghe.jpg)

另一个在工具链方面的感受算是我们团队自己的创新吧，就是前面提到的我们把 GitHub Issues 用于产品设计讨论。所有的产品需求都在 GitHub 被跟踪，产品经理、设计师、工程师都会参与其中，少部分情况也会涉及到市场营销的人员。

产品经理负责整个需求的起草，创建一个新的 GtiHub Issue，包括但不限于前期信息的搜集，合作团队的预先沟通，制定需求的不同阶段目标，召集相关人员做 Kick Off（需求启动的一个会议），根据会议讨论，需求和目标可能会做更改。

产品经理或者设计师会根据一开始宽泛的需求给出一些原型设计，经过讨论后，确定一个方案。设计师会给出高保真设计稿和必要的标注信息，交付工程师具体实现。这过程中，设计的迭代、设计产出、工程进度都会在 GitHub Issue 上更新。任何一个项目外的人都可以很轻松的了解到项目的需求和状态，甚至期间作出的各种决定的由来。

出色的产品经理和设计师在整个过程中极为重要，也让这个方案对工程师极为友好，既有了参与前期讨论，提出建议的机会，也省去了很多不必要的麻烦。各个角色各司其职，相互配合，达到更高的效率。

本质上来说，我们只是恰好选了 GitHub Issue，因为我们团队的人比较喜欢，其实有很多工具可以承载产品和设计的讨论。

总而言之，远程协作的一个关键是选择合适团队的流程与工具链。

# 透明度

从前面讲工具链的可以看出，其实我们很多的工具都是在为了提高透明度而努力。我认为在远程团队中，透明度是重中之重，所有的流程和工具链的改进都应该为提高透明度为目的。

前面提到的邮件、日历、GitHub Issues 一方面是为了提高效率，另一方面也给了不同地区的人公平参与的机会。一般来说，一个公司有总部，远程团队亦是如此，总会有决策区域。但是大家总不希望”大脑“说什么就做什么，就相当于一个外包团队。

重要的不是团队究竟为了提高透明度做了什么，重要的是要有这个意识。不过我还是来说说我们团队都有哪些措施来体现这一点的吧。

__中心化讨论__

![Discussions](/images/remote-working/discussions.jpg)

远程团队最大的优点就是分布式，可以异步工作，但很多时候的讨论是需要中心化的。一个例子就是我上面提到的 GitHub Issues，我们用它来优化产品流程，直接剥离掉了不同地区的属性，所有人都是平等的。在工程师内部，例如我们 iOS 团队，所有工程的重大决策全部都发布到 GitHub 上并邀请所有人讨论和Review。Teams 中每个团队有个对应的 Channel，主要的事情都会在这个 Channel 中进行讨论，基本上不存在你会错过什么的情况。


__相互了解__

![Discussions](/images/remote-working/meetup.jpg)

我们认为让互相之间知道对方在做什么是非常重要的，在这方面，我们尝试了很多。例如，每周更新每个人重点工作的领域，你可以看到全球每个人正在负责什么。每年一次的 Global Meetup 给了所有人一次面对面交流的机会，全球各地的人聚在一起互相认识，把脸和名字对起来，一起头脑风暴，研究现在团队的问题，制定下一步的规划。这时你才会意识到原来这个拥有一亿多月活用户的产品是由一群分布在天南海北，甚至很多都没见过面的人培育出来的。


欢迎大家在下方留言谈谈你们团队在远程协作上的一些优秀方案。



-----------------



> Microsoft 正在火热招聘中，如果您也认同相似的理念，想和同样优秀的产品经理、设计师、工程师一起工作，不妨关注一下。
>
> 工作地点：__苏州__，__北京__，__上海__
>
> 招聘目标：__工程师__，__设计师__，__产品经理__
>
> 要求：
>
> - 出色的基础素质，例如工程师看重算法、数据结构、沟通、心态、审美等
> - 拥有对高质量产品的追求
> - 出众的对应岗位的能力
> - 基本的语言能力（英语）
> - 对工作经验不做要求，对学历不做要求
>
> 欢迎发送简历到 __leimagnet[AT]gmail.com__ 联系内推。
