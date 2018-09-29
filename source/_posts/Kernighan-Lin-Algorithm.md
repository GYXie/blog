title: Kernighan-Lin Algorithm
categories: Algorithm
tags: Algorithm
date: 2015-09-17 01:16:35
---

## Kernighan-Lin
> 这是一个启发式的(heuristic)的解决图分割问题的算法.
**Kernighan-Lin**是一个O(n<sup>2</sup>log(n))的解决图分割问题的启发式算法(heuristic algorithm)
**G(V,E)**表示一个图,**V**表示节点集合,**E**表示边的集合.Kernighan-Lin算法试图找到一个分割(partition)将V分成不相连(disjoint)的两个子集**A**和**B**.且A和B大小相同,满足A和B之间的边的权重之和**T**最小.**I<sub>a</sub>**表示**a**的内部消耗.也就是a和A中的其它节点之间边的权重的和.**E<sub>a</sub>**表示a的外部消耗,也就是a和B中节点间的边的权重的和.更进一步
	**D<sub>a</sub>=E<sub>a</sub>-I<sub>a</sub>**
表示外部消耗和内部消耗的差.如果a和b是可交换的,那么消耗的减少就是
	**T<sub>old</sub>-T<sub>new</sub>=D<sub>a</sub>+D<sub>b</sub>-2c<sub>a,b</sub>**
其中**c<sub>a,b</sub>**是a和b之间所有可能边的消耗总和.
这个算法试图找到一个A和B元素之间的最优的可交换操作序列,使得T<sub>old</sub>-T<sub>new</sub>最大,且产生一个A和B之间的分割.
```
   function Kernighan-Lin(G(V,E)):
       determine a balanced initial partition of the nodes into sets A and B
       do
          compute D values for all a in A and b in B
          let gv, av, and bv be empty lists
          for (n := 1 to |V|/2)
             find a from A and b from B, such that g = D[a] + D[b] - 2*E(a, b) is maximal
             remove a and b from further consideration in this pass
            add g to gv, a to av, and b to bv
            update D values for the elements of A = A \ a and B = B \ b
         end for
         find k which maximizes g_max, the sum of g[1],...,g[k]
         if (g_max > 0) then
            Exchange av[1],av[2],...,av[k] with bv[1],bv[2],...,bv[k]
      until (g_max <= 0)
   return G(V,E)
```