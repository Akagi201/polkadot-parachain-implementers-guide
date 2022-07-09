<!-- markdownlint-disable MD010 -->
# 协议概览

本节旨在高水平地描述在 Polkadot 中运行平行链所涉及的角色和协议。具体来说，我们描述了不同的行为者如何相互沟通，他们单独和集体保存什么样的数据结构，以及他们为什么要做这些事情的高级目的。

我们的最高目标是将一个平行链区块从编写到安全包含，并定义一个可以重复和平行进行的过程，用于许多不同的平行链，以便随着时间的推移扩展它们。理解这里所采取的高层次的方法对于进一步提供拟议的架构的背景是很重要的。与此相关的 Polkadot 的关键部分是主要的 Polkadot 区块链，被称为中继链，以及为该区块链提供安全和输入的行为者。

首先，有必要介绍一下我们参与这个协议的主要行为者。

1. 验证人。这些节点负责验证拟议的平行链区块。他们通过检查区块的有效性证明（PoV）来做到这一点，并确保 PoV 仍然可用。他们把金融资本作为 "游戏中的皮肤"，如果他们被证明是错误的验证，可以被砍掉（销毁）。
2. 收集人。这些节点负责创建验证人知道如何检查的有效性证明。创建 PoV 通常需要熟悉交易格式和平行链的区块编写规则，以及能够访问平行链的全部状态。
3. 渔民。这些是由用户操作的、无权限的节点，其目标是抓住行为不端的验证人以换取赏金。收集人和验证人也可以作为 "渔夫" 行事。渔民对于安全来说并不是必须的，而且本文也没有深入涉及。

这意味着一个简单的管道，收集人向验证人发送平行链区块和其必要的 PoV 来检查。然后，验证人使用 PoV 验证区块，签署描述积极或消极结果的声明，如果有足够的积极声明，该区块可以在中继链上被注意到。负面的声明不是否决，但会导致争议，那些错误的一方会被惩罚。如果另一个验证人后来发现一个验证人或一组验证人错误地签署了一份声明，声称一个区块是有效的，那么这些验证人将被惩罚，而验证人将获得赏金。

然而，这种表述有一个问题。为了让另一个验证人在事后检查前一组验证人的工作，PoV 必须保持 _可用_，以便其他验证人可以获取它以检查工作。预计 PoVs 太大，无法直接包含在区块链中，所以我们需要一个替代的 _数据可用性_ 方案，要求验证人证明他们工作的输入将保持可用，所以他们的工作可以被检查。经验性测试告诉我们，许多 PoV 在重载期间可能在 1 到 10MB 之间。

下面是对包含管道的描述：一个平行链区块（或简称 parablock）从创建到包含的路径。

1. 验证人是由验证人分配程序选择并分配给平行链的。
2. 收集人产生平行链区块，它被称为平行链候选人或候选人，同时还有候选人的 PoV。
3. 收集人通过收集人协议将候选人和 PoV 转发给分配给同一平行链的验证人。
4. 在某一特定时间点分配到某一平行链的验证人参与到候选人支持子系统中，以验证被提出来验证的候选人。从验证人那里收集到足够多的签名有效性声明的候选人被认为是 "可支持的"。他们的支持是已签署的有效性声明的集合。
5. 由 BABE 选择的中继链区块的作者，可以为每个平行链记下最多一个可支持的候选者，并将其与支持一起纳入中继链区块。一个可支持的候选者一旦被纳入中继链，就被认为在中继链的那个分叉中得到了支持。
6. 一旦在中继链中得到支持，该平行链候选者就被认为是 "待定可用性"。它不被认为是作为平行链的一部分，直到它被证明是可用的。
7. 在接下来的中继链区块中，验证人将参与可用性分布子系统以确保候选者的可用性。有关候选者可用性的信息将在随后的中继链块中被指出。
8. 一旦中继链状态机有足够的信息认为候选人的 PoV 是可用的，该候选人就被认为是平行链的一部分，并逐渐成为一个完整的平行链区块，或简称为 parablock。

请注意，候选人可以在以下任何一种情况下不被列入：

* 收集人无法将该候选者传播给分配给该平行链的任何验证人。
* 候选人没有被参与候选人支持子系统的验证人支持。
* 候选人没有被中继链区块作者选择纳入中继链。
* 候选人的 PoV 在超时内不被认为是可用的，并被从中继链中丢弃。

这个过程还可以进一步划分。步骤 2 和 3 与收集人的工作有关，即整理并通过整理分发子系统将候选人分发至验证人。步骤 3 和 4 涉及到候选人支持子系统中的验证人和区块作者（本身就是一个验证人）的工作，将区块纳入中继链。步骤 6、7 和 8 对应于中继链状态机的逻辑（也称为运行时），用于将区块完全纳入链中。第 7 步需要验证人的进一步工作，以参与可用性分布子系统，并将该信息纳入中继链，以便第 8 步完全实现。

这给我们带来了该过程的第二部分。一旦一个 parablock 被认为是可用的和平行链的一部分，它仍然是 "等待批准"。在流水线的这一阶段，该 parablock 已经得到了分配给该 parablock 的小组中大多数验证人的支持，其数据已经被整个验证人集合保证为可用。一旦它被认为是可用的，主机甚至会开始接受该块的子块。在这一点上，我们可以认为该区块已经被暂时包含在平行链中，尽管还需要更多的确认。然而，平行链组中的验证人（被称为该平行链的 "平行链验证人"）是从一个验证人集合中抽出的，这个验证人集合中含有一定比例的拜占庭，或者任意的恶意成员。这就意味着，某些区块的平行链验证人可能是多数人不诚实的，这意味着在区块被认为是被批准之前，必须对其进行（二次）批准检查。这一点是必要的，因为一个给定的 parachain 的验证人是从整个验证人集合中抽取的，而这个验证人集合被假定为最多<1/3 不诚实--这意味着有机会随机抽取 parachain 的验证人，这些验证人多数或完全不诚实，可能错误地支持一个候选人。审批程序允许我们在事后发现这种错误行为，而不需要分配更多的 Parachain 验证人，并减少系统的吞吐量。一个 parablock 未能通过审批过程，将使该区块以及其所有的子代无效。然而，只有支持有问题的区块的验证人会被惩罚，而不是支持后代的验证人。

审批程序，一目了然，看起来像这样：

1. 被纳入流水线的 Parablocks 正在等待批准，其时间窗口被称为二次检查窗口。
2. 在二次检查窗口期间，验证人随机地自我选择对 parablock 进行二次检查。
3. 这些验证人在这里被称为二次检查者，他们获得了 parablock 和其 PoV，并重新运行验证功能。
4. 二级检查员对他们的检查结果进行 gossip。相互矛盾的结果会导致升级，所有验证人都需要检查该块。争端中失败一方的验证人被惩罚。
5. 在审批过程结束时，该区块要么被批准，要么被拒绝。稍后会有更多关于拒绝过程的内容。

关于审批程序的更多信息，可在 "审批" 专门章节中找到。关于争议的更多信息，可在 "争议" 的专门章节中找到。

这两条流水线总结了扩展和获得一个 Parablock 的全部安全所需的事件顺序。请注意，在接受一个新的区块之前，必须结束对一个特定的 parachain 的包含流水线。纳入后，审批程序开始启动，可以同时为许多平行链区块运行。

重申候选人的生命周期：

1. 候选：由一个收集人向一个验证人提出。
2. 附议：由一个验证人向其他验证人提出。
3. 可支持：由大多数指定的验证人证明的有效性。
4. 被支持：可支持的并在中继链的分叉中指出。
5. 待定的可用性：有支持，但还没有被认为是可用的。
6. 包含：已支持并被认为是可用的。
7. 接受：已支持，可使用，且无争议。

```mermaid
digraph {
	subgraph cluster_vg {
		label=<
			Parachain Validators
			<br/>
			(subset of all)
		>
		labeljust=l
		style=filled
		color=lightgrey
		node [style=filled color=white]

		v1 [label="Validator 1"]
		v2 [label="Validator 2"]
		v3 [label="Validator 3"]

		b [label="(3) Backable", shape=box]

		v1 -> v2 [label="(2) Seconded"]
		v1 -> v3 [label="(2) Seconded"]

		v2 -> b [style=dashed arrowhead=none]
		v3 -> b [style=dashed arrowhead=none]
		v1 -> b [style=dashed arrowhead=none]
	}

	v4 [label=<
		<b>Validator 4</b> (relay chain)
		<br/>
		<font point-size="10">
			(selected by BABE)
		</font>
	>]

	col [label="Collator"]
	pa [label="(5) Relay Block (Pending Availability)", shape=box]
	pb [label="Parablock", shape=box]
	rc [label="Relay Chain Validators"]

	subgraph cluster_approval {
		label=<
			Secondary Checkers
			<br/>
			(subset of all)
		>
		labeljust=l
		style=filled
		color=lightgrey
		node [style=filled color=white]

		a5 [label="Validator 5"]
		a6 [label="Validator 6"]
		a7 [label="Validator 7"]
	}

	b -> v4 [label="(4) Backed"]
	col -> v1 [label="(1) Candidate"]
	v4 -> pa
	pa -> pb [label="(6) a few blocks later..." arrowhead=none]
	pb -> a5
	pb -> a6
	pb -> a7

	a5 -> rc [label="(7) Approved"]
	a6 -> rc [label="(7) Approved"]
	a7 -> rc [label="(7) Approved"]
}
```

上图显示了一个区块从（1）候选状态到（7）批准状态的乐观路径。

还需要注意的是，中继链是由 BABE 扩展的，它是一种分叉算法。这意味着不同的区块作者可以同时被选择，而且可能不是建立在同一个区块父本上。此外，验证人的集合并不固定，平行链的集合也不固定。而且，即使有相同的验证人和平行链集，验证人对平行链集的分配也是灵活的。这意味着，在接下来的章节中提出的架构必须处理网络状态的可变性和多重性。

```mermaid
digraph {
	rca [label="Relay Block A" shape=box]
	rcb [label="Relay Block B" shape=box]
	rcc [label="Relay Block C" shape=box]

	vg1 [label=<
		<b>Validator Group 1</b>
		<br/>
		<br/>
		<font point-size="10">
			(Validator 4)
			<br/>
			(Validator 1) (Validator 2)
			<br/>
			(Validator 5)
		</font>
	>]
	vg2 [label=<
		<b>Validator Group 2</b>
		<br/>
		<br/>
		<font point-size="10">
			(Validator 7)
			<br/>
			(Validator 3) (Validator 6)
		</font>
	>]

	rcb -> rca
	rcc -> rcb

	vg1 -> rcc [label="Building on C" style=dashed arrowhead=none]
	vg2 -> rcb [label="Building on B" style=dashed arrowhead=none]
}
```

在这个例子中，第 1 组已经收到了 C 块，而其他组由于网络不同步而没有收到。现在，第 2 组的验证人可以在 B 的基础上建立另一个块，称为 C'。假设之后，一些验证人同时知道了 C 和 C'，而其他验证人仍然只知道一个。

```meramid
digraph {
	rca [label="Relay Block A" shape=box]
	rcb [label="Relay Block B" shape=box]
	rcc [label="Relay Block C" shape=box]
	rcc_prime [label="Relay Block C'" shape=box]

	vg1 [label=<
		<b>Validator Group 1</b>
		<br/>
		<br/>
		<font point-size="10">
			(Validator 4) (Validator 1)
		</font>
	>]
	vg2 [label=<
		<b>Validator Group 2</b>
		<br/>
		<br/>
		<font point-size="10">
			(Validator 7) (Validator 6)
		</font>
	>]
	vg3 [label=<
		<b>Validator Group 3</b>
		<br/>
		<br/>
		<font point-size="10">
			(Validator 2) (Validator 3)
			<br/>
			(Validator 5)
		</font>
	>]

	rcb -> rca
	rcc -> rcb
	rcc_prime -> rcb

	vg1 -> rcc       [style=dashed arrowhead=none]
	vg2 -> rcc_prime [style=dashed arrowhead=none]
	vg3 -> rcc_prime [style=dashed arrowhead=none]
	vg3 -> rcc       [style=dashed arrowhead=none]
}
```

那些意识到许多竞争头的验证人必须意识到在每个头上发生的工作。他们可能对两个头都有一些或全部的贡献。由于网络的不同步性，两个分叉有可能在一段时间内平行增长，尽管在没有对抗性网络的情况下，在有验证人知道两个链头的情况下这是不可能的。
