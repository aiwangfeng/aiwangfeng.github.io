```markdown
# 高德纳的《Claude的循环》中文翻译

## 克劳德的循环 (Claude's Cycles)

唐·克努斯 (Don Knuth)，斯坦福大学计算机科学系
(2026年2月28日；修订于2026年3月2日)

震惊！震惊！昨天我得知，我花了几个星期研究的一个未解问题，刚刚被三周前发布的、Anthropic公司的混合推理模型 Claude Opus 4.6 解决了！看来，不久的某一天，我将不得不修正我对“生成式人工智能”的看法了。得知我的猜想不仅有了一个漂亮的解，而且还能见证自动推理和创造性问题解决方面的这一巨大进步，这是多么令人高兴的事啊！在这篇短文中，我将简要讲述这个故事。

问题如下，它源于我正在为《计算机程序设计艺术》未来卷撰写关于有向哈密顿循环的内容时：

考虑一个有向图，它有 \(m^{3}\) 个顶点 \(ijk\)，其中 \(0\leq i,j,k<m\)。从每个顶点出发有三条弧，分别指向 \(i^{+}jk\)，\(ij^{+}k\) 和 \(ijk^{+}\)，这里 \(i^{+}=(i+1)\bmod m\)。目标是对于所有 \(m>2\)，找到将这些弧分解为三个有向 \(m^{3}\)-循环的通用方法。

我已经解出了 \(m=3\) 的情况，并在一道习题的解答中将其作为推广问题提出 [3]。我的朋友 Filip Stappers 接受了挑战，并通过实验发现了 \(4\leq m\leq 16\) 的解；因此，除了 \(m\leq 2\) 的情况外，所需的分解极有可能存在。

事实上，正是 Filip 有勇气用上面确切的措辞向 Claude 提出这个问题。他还提供了指导/辅导，指示 Claude 总结其 ongoing 进展：

**在每次运行 exploreXX.py 之后，在进行任何其他操作之前，立即更新此文件 [plan.md]。** 没有例外。在未在此处记录之前的探索之前，不要开始下一次探索。

Claude 的进攻计划相当令人赞叹。首先，它重新表述了问题：“需要 \(\sigma: Z_{m}^{3}\to S_{3}\)，在每个顶点分配 \(\{0,1,2\}\) 的一个排列；顶点 \(v\) 处的循环 \(c\) 沿着 \(\sigma(v)[c]\) 方向前进。每个最终生成的函数图必须是一个单一的哈密顿循环。”然后，它尝试寻找一个简单的答案：“如果 \(\sigma\) 是循环的，即对于某个 \(g:\ Z_{m}^{3}\to Z_{3}\)，有 \(f_{c}(v)=(c+g(v))\bmod 3\)，那么我们只需要一个函数 \(g\)。测试了线性 \(g=(a\cdot i+b\cdot j+c\cdot k)\bmod 3\) 和二次变体。*没有线性或简单的二次 \(g\) 可行。*”

Claude 的第二次探索是它所谓的 DFS 搜索尝试（深度优先搜索）：“尝试对 \(m=3\) 的 \(\sigma\) 赋值进行暴力 DFS（搜索空间为 \(6^{27}\)）。如果没有良好的剪枝，速度太慢。”

第三次，它尝试了所谓的二维蛇形分析：“*关键的二维结果：* 对于 \(m=3\)，模式 \([0^{m-1}1]^{m}\) 在 Cay\((Z_{m}^{3},\{e_{0},e_{1}\})\) 中给出了一个哈密顿循环。其补也有效，给出了一个有效的二维分解。”这确实令人印象深刻！首先，Claude 认识到这个有向图是一个所谓的带有两个生成元 \(e_{0}\) 和 \(e_{1}\) 的凯莱有向图；数学家会理解这些弧对 \(m^{2}\) 个顶点起到了群的作用，其中 \(e_{0}\) 增加第一个分量，而 \(e_{1}\) 增加第二个分量。然后，它用符号表述了针对一般 \(m\) 的循环，尽管由于某些原因它仍然说它只是在看 \(m=3\) 的情况。并且它决定称之为“蛇形模式”！

第四次探索构建了“三维蛇形模式” \([0^{m-1}1]^{m-1}[0^{m-1}2]^{m}\)。（这是个排版错误，实际应为 \([[0^{m-1}1]^{m-1}0^{m-1}2]^{m}\)。这是 [2] 第 299 页上一个称为“模 \(m\) 进制格雷码”的经典序列。）从图中删除这个循环后，Claude 观察剩余的二正则有向图，发现它具有“刚性结构，使得分解困难。”

尽管如此，第五次尝试分析了那个剩余有向图，找到了一些循环。不幸的是，它们没有成功。

回到绘图板。又尝试了其他几种方法，例如，考察位于满足 \(i+j+k\) 同余于 \(-1\) 的超平面上的顶点处的选择。直到第15次探索，才出现了有希望的东西，Claude 称之为**纤维分解 (fiber decomposition)**：“商映射 \(\gamma(i,j,k)=i+j+k\bmod m\) 将所有弧从纤维 \(F_{s}\) 映射到 \(F_{s+1}\)。该有向图是*分层 (layered)* 的。在纤维坐标 \((i,j)\) 下，有 \(k=(s-i-j)\bmod m\)。”

使用这些纤维坐标，问题现在可以重新表述为：为每个 \((i,j,s)\) 选择一个由三个算子组成的排列，这些算子将 \((i,j)\) 移动 \((1,0)\) 或 \((0,1)\) 或 \((0,0)\)。“尝试了 \(\sigma\) 仅依赖于纤维索引 \(s\)（按 \(s\) 轮转）：给出了 \((i+j+k)\bmod 3\) 模式，但无效。随机局部搜索为 \(m=3\) 找到了解，但没有出现通用模式。”

探索 18 “将完整的纤维框架实现为可执行代码。对于 \(m=3\)，*穷举回溯* 在 0.1 秒内找到了一个有效分解（10932 个节点）。”在这个解中，排列 sigma 对于纤维 \(s=0\) 中的所有 \((i,j)\) 都是相同的。作为第 20 次探索的一部分，通过模拟退火为 \(m=4\) 找到了一个类似的“均匀”纤维。

在第 21 次探索中，Claude 意识到二维蛇形函数 “\(Q(i,j)=(i+1,j)\) 如果 \(i+j\neq m-1\bmod m\)，否则 \((i,j+1)\)” 具有良好的性质。如果在纤维 0 处进行均匀选择，并在所有其他纤维处使用蛇形函数的某个版本，当且仅当 \(m\) 为奇数时，这看起来很有希望。

这引发了一系列“大规模”模拟退火实验，但都徒劳无功。第 25 次探索后的结论是“SA 可以找到解，但不能给出通用构造。需要纯数学。”

所以，Claude 现在换上了新的思考帽。有一次它自言自语道：“也许正确的框架是：不要从纤维角度思考，直接思考什么构成哈密顿循环。”

在第 27 次探索时有一次接近成功。“将三维蛇形作为循环 0，应用循环坐标旋转得到循环 1 和 2：\(d_{1}(i,j,k)=(d_{0}\left(j,k,i\right)+1)\bmod 3\)；\(d_{2}(i,j,k)=(d_{0}\left(k,i,j\right)+2)\bmod 3\)；*对于每个测试的 \(m=3 \ldots 9\)，这三个循环各自都是哈密顿的！* 只有 \(3(m-1)\) 个顶点（共 \(m^{3}\) 个）存在冲突，*所有这些冲突顶点* 都位于超平面 \(i+j+k=m-1\bmod m\) 上。”

不幸的是，这些冲突无法解决。第 29 次探索证明了许多看似合理的场景实际上是不可能的。“这完全否定了‘单一超平面+旋转’的方法。……我们必须允许方向函数在一个旋转轨道内使用*不同的*值。”

但是，啊哈。第 30 次探索回顾了第 20 次探索中由 SA 找到的解，并注意到每个纤维上的选择仅依赖于单个坐标：\(s=0\) 仅依赖于 \(j\)；\(s=1\) 和 \(s=2\) 仅依赖于 \(i\)。这导致了一个具体的构造（第 31 次探索），以 Python 程序的形式呈现，该程序为 \(m=3, 5, 7, 9,\) 和 \(11\) 产生了有效结果——万岁！“所有三个循环都是哈密顿的，所有弧都被使用，完美的分解！”

以下是那个 Python 程序的简化版本，转换为 C 语言形式：

```c
int c,i,j,k,m,s,t; char*d;
for(c=0;c<3;c++){
    for(t=i=j=k=0;;t++){
        printf("%x%x%x\n",i,j,k);
        if(t==m*m*m) break;
        s=(i+j+k)%m;
        if(s==0) d=(j==m-1? "012":"210");
        else if(s==m-1) d=(i>0? "120":"210");
        else d=(i==m-1? "201":"102");
        switch(d[c]){
            case '0': i=(i+1)%m; break;
            case '1': j=(j+1)%m; break;
            case '2': k=(k+1)%m; break;
        }
    }
    printf("\n");
}
```

Filip Stappers 测试了 Claude 的 Python 程序，覆盖了 3 到 101 之间的所有奇数 m，每次都发现了完美的分解。因此，他相当合理地得出结论，对于奇数 m 的情况，问题确实已经解决了。并且他迅速将这个令人震惊的消息告诉了我。

当然，严格的证明仍然需要。而这个证明的构造过程原来相当有趣。让我们看看，例如，打印出的第一个循环；我们必须证明它的长度是 m^{3}。

该循环的规则虽然不平凡，但相当简单：令 s=(i+j+k)\bmod m。

当 s=0 时，如果 j=m-1，则 bump i，否则 bump k。

当 0<s<m-1 时，如果 i=m-1，则 bump k，否则 bump j。

当 s=m-1 时，如果 i>0，则 bump j，否则 bump k。

（“Bump”意思是“加 1，模 m”。）因此，在 m=3 的特殊情况下，该循环为：

022\longrightarrow 002\longrightarrow\,000\longrightarrow\,001 \longrightarrow\,011\longrightarrow\,012\longrightarrow\,010\longrightarrow\,020 \longrightarrow\,021\longrightarrow



121\longrightarrow 101\longrightarrow 111\longrightarrow 112 \longrightarrow 122\longrightarrow 102\longrightarrow 100\longrightarrow 110 \longrightarrow 120\longrightarrow



220\longrightarrow 221\longrightarrow 201\longrightarrow 202 \longrightarrow 200\longrightarrow 210\longrightarrow 211\longrightarrow 212 \longrightarrow 222\longrightarrow 022.

而在 m=5 的特殊情况下，它是：

042\longrightarrow\,002\longrightarrow\,012\longrightarrow\,022 \longrightarrow\,023\longrightarrow\,024\longrightarrow\,034\longrightarrow\,044 \longrightarrow\,004\longrightarrow\,000\longrightarrow\,001\longrightarrow\,011 \longrightarrow\,021\longrightarrow



031\longrightarrow\,032\longrightarrow\,033\longrightarrow\,043 \longrightarrow\,003\longrightarrow\,013\longrightarrow\,014\longrightarrow\,0 10\longrightarrow\,020\longrightarrow\,030\longrightarrow\,040\longrightarrow\,041 \longrightarrow



141\longrightarrow\,101\longrightarrow\,111\longrightarrow\,121 \longrightarrow\,131\longrightarrow\,132\longrightarrow\,142\longrightarrow\,102 \longrightarrow\,112\longrightarrow\,122\longrightarrow\,123\longrightarrow\,133 \longrightarrow\,143\longrightarrow



103\longrightarrow\,113\longrightarrow\,114\longrightarrow\,124 \longrightarrow\,134\longrightarrow\,144\longrightarrow\,104\longrightarrow\,100 \longrightarrow\,110\longrightarrow\,120\longrightarrow\,130\longrightarrow\,140 \longrightarrow



240\longrightarrow\,200\longrightarrow\,210\longrightarrow\,220 \longrightarrow\,230\longrightarrow\,231\longrightarrow\,241\longrightarrow\,201 \longrightarrow\,211\longrightarrow\,221\longrightarrow\,222\longrightarrow\,232 \longrightarrow\,242\longrightarrow



202\longrightarrow\,212\longrightarrow\,213\longrightarrow\,223 \longrightarrow\,233\longrightarrow\,243\longrightarrow\,203\longrightarrow\,204 \longrightarrow\,214\longrightarrow\,224\longrightarrow\,234\longrightarrow\,244 \longrightarrow



344\longrightarrow\,304\longrightarrow\,314\longrightarrow\,324 \longrightarrow\,334\longrightarrow\,330\longrightarrow\,340\longrightarrow\,300 \longrightarrow\,310\longrightarrow\,320\longrightarrow\,321\longrightarrow\,331 \longrightarrow\,341\longrightarrow



301\longrightarrow\,311\longrightarrow\,312\longrightarrow\,322 \longrightarrow\,332\longrightarrow\,342\longrightarrow\,302\longrightarrow\,303 \longrightarrow\,313\longrightarrow\,323\longrightarrow\,333\longrightarrow\,343 \longrightarrow



443\longrightarrow\,444\longrightarrow\,440\longrightarrow\,441 \longrightarrow\,401\longrightarrow\,402\longrightarrow\,403\longrightarrow\,404 \longrightarrow\,400\longrightarrow\,410\longrightarrow\,411\longrightarrow\,412 \longrightarrow\,413\longrightarrow



414\longrightarrow\,424\longrightarrow\,420\longrightarrow\,421 \longrightarrow\,422\longrightarrow\,423\longrightarrow\,433\longrightarrow\,434 \longrightarrow\,430\longrightarrow\,431\longrightarrow\,432\longrightarrow\,442 \longrightarrow\,042.

具有相同 s 值的三元组恰好间隔 m 步。为了证明所有 m^{3} 个三元组都出现了，我们需要证明对于任何给定的 s 值，所有 m^{2} 个三元组都存在。

注意，第一坐标 i 仅在 s=0 且 j=m-1 时改变。因此，具有任何给定第一坐标 i 值的 m^{2} 个三元组都是连续出现的。（我们的示例循环通过从顶点开始来表明这一点，该顶点处 i 刚刚变为 0，而不是从顶点 000 开始。）

注意，i=0 的循环元素通常必须从顶点 0(m-1)2 开始，因为前一个顶点必须具有 i=j=m-1 且 s=0。而 0(m-1)2（其 s=1）之后通常是 002, 012, ..., 0(m-3)2, 0(m-3)3，将我们带回 s=0。

通常，当 1<k<m 时，0(m-k)k 之后紧跟着 0(m-k)(k\!+\!1)；然后 j 增加，直到我们到达 0(m-k\!-\!2)(k\!+\!1)，它的后继是 0(m-k\!-\!2)(k\!+\!2)。因此，每次我们到达一个 s=0 的顶点时，k 增加 2（模 m）。由于 m 是奇数，我们最终会到达 0(m\!-\!1)1——此时我们将已经看到所有形式为 0jk 的 m^{2} 个顶点。而 0(m\!-\!1)1 的后继是 1(m\!-\!1)1。

到目前为止一切顺利！一般来说，第一分量 i 满足 0<i<m-1 的循环元素将从 i(m\!-\!1)(2\!-\!i) 开始，其中 2-i 当然是以模 m 计算。如果一切顺利，这些元素应该包括所有以 i 开头的 m^{2} 个顶点，并以 i(m\!-\!i)(1\!-\!i) 结束——它的后继是 (i\!+\!1)(m\!-\!1)(1\!-\!i)。而且一切确实顺利：我们反复推进第二分量，除非在 s=0 时；因此当 s=0 时我们看到的顶点是 i(m\!-\!2)(2\!-\!i),\,i(m\!-\!3)(3\!-\!i),\,\ldots,\,i0(m\!-\!i),\,i(m\!-\!1 )(1\!-\!i)。

最后我们到达 (m\!-\!1, m\!-\!1, 0)，它的后继是…等等

```markdown
```