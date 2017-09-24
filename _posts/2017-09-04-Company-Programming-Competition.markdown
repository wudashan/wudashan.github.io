---
layout:     post
title:      "公司编程竞赛之最长路径问题"
subtitle:   "教练，我想学算法。"
date:       2017-09-04 23:00:00
author:     "Wudashan"
header-img: "img/post-bg-company-programming-competition.jpg"
catalog: true
tags:
    - ACM
    - 算法
    - 路径问题
---


> 从深度优先搜索到贪心算法，再到模拟退火算法。

# 前言

最近产品线举办了一个软件编程大赛，题目非常的有趣，就是在一个9 × 9的格子里，你要和另一个敌人PK，在PK的过程中，你可以吃格子里的果实来提升攻击力。每次可以往正上、正下、正左、正右、左上、左下、右上、右下八个方向走。每次要么连续吃果实要么连续走空白区域，且不能走重复的位置。初始状态如下图所示：

![](http://o7x0ygc3f.bkt.clouddn.com/%E6%9C%80%E9%95%BF%E8%B7%AF%E5%BE%84%E9%97%AE%E9%A2%98-5.png)

为了尽可能地吃最多的果实，我们的路线可以这样规划：

![](http://o7x0ygc3f.bkt.clouddn.com/%E6%9C%80%E9%95%BF%E8%B7%AF%E5%BE%84%E9%97%AE%E9%A2%98-9.png)

所以，我们可以把这个问题归结为：已知空白区域不能走，每次可以往正上、正下、正左、正右、左上、左下、右上、右下八个方向走，走过的位置不能再走，求能吃最多果实的路线（最长路径问题）？

---

# 前置条件

## 地图表示

<span id="simpleMap"></span>

首先我们将上面的地图使用布尔类型的二维数组表示，其中true表示可以行走的格子，false表示不能行走的格子：


```
boolean[][] simpleMap = new boolean[][] {
    {false, false, false, false, false, false, false, false, false},
    {false, false, false, false, false, false, true , true , false},
    {false, false, false, true , false, false, true , true , false},
    {false, false, true , false, false, false, false, false, false},
    {false, false, true , false, false, false, false, false, false},
    {false, false, true , false, false, false, false, false, false},
    {false, false, false, true , false, true , false, false, false},
    {false, false, false, false, true , true , false, false, false},
    {false, false, false, false, false, false, false, false, false}
};
```


## 节点表示

对于地图上的每一个点，我们用一个简单类来表示：

```
public class Pos {

    private int x;  // 横坐标
    private int y;  // 纵坐标
    
    // get、set、construct方法省略

    @Override
    public String toString() {
        final StringBuffer sb = new StringBuffer("Pos{");
        sb.append("x=").append(x);
        sb.append(", y=").append(y);
        sb.append('}');
        return sb.toString();
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Pos pos = (Pos) o;

        if (x != pos.x) return false;
        return y == pos.y;
    }

    @Override
    public int hashCode() {
        int result = x;
        result = 31 * result + y;
        return result;
    }

    
}
```

由于我们是使用横纵坐标而不是几行几列来表示一个点（没错，我就是这么傲娇），那么我们就需要给地图定义横纵坐标方向。方向如下图所示：

![](http://o7x0ygc3f.bkt.clouddn.com/%E6%9C%80%E9%95%BF%E8%B7%AF%E5%BE%84%E9%97%AE%E9%A2%98-8.png)

那么**起点上方的果实**坐标就是[3, 2]（横坐标为3，纵坐标为2），但是对应着二维数组为map[2][3]（第二行，第三列），即横坐标对应着二维数组的列，纵坐标对应着二维数组的行。

## 方向表示

为了程序简洁，我们给节点的八个方向定义偏移量，这样每次行走只要对偏移量数组进行for循环就可以了。

```
Pos[] moveOffset = new Pos[] {
    new Pos(-1,  0),    // 向左移动
    new Pos(-1, -1),    // 向左上移动
    new Pos( 0, -1),    // 向上移动
    new Pos( 1, -1),    // 向右上移动
    new Pos( 1,  0),    // 向右移动
    new Pos( 1,  1),    // 向右下移动
    new Pos( 0,  1),    // 向下移动
    new Pos(-1,  1)     // 向左下移动
};
```


---

# 深度优先搜索算法

## 算法思想

拿到这道题，脑袋里第一个想到的就是深度优先搜索算法，其思想为每次往八个方向递归，当不能继续走下去的时候保存最长路径，并回退到能继续行走的点，继续递归直到结束。

## 代码示例

接下来就是我们的深度优先搜索代码：

```
/**
 * 通过深度优先搜索获取最长路径
 * @param map 地图
 * @param start 起点
 * @param moveOffset 方向偏移量
 * @return 最长路径
 */
public static List<Pos> getLongestPathByDFS(boolean[][] map, Pos start, Pos[] moveOffset) {

    List<Pos> longestPath = new ArrayList<>();
    dfs(start, map, new ArrayList<>(), longestPath, moveOffset);
    return longestPath;

}

/**
 * 递归实现深度优先搜索
 * @param pos 当前节点
 * @param map 地图
 * @param path 当前路径
 * @param result 最终结果
 * @param moveOffset 方向偏移量
 */
private static void dfs(Pos pos, boolean[][] map, List<Pos> path, List<Pos> result, Pos[] moveOffset) {

    // 记录当前节点的周围是否经过
    List<Pos> visited = new ArrayList<>();

    // 保存当前节点的周围节点
    Pos[] neighbours = new Pos[moveOffset.length];

    // 依次向周围行走
    for (int i = 0; i < moveOffset.length; i++) {
        Pos next = new Pos(pos.getX() + moveOffset[i].getX(), pos.getY() + moveOffset[i].getY());
        neighbours[i] = next;
        if (inMap(map, next) && !path.contains(next) && map[next.getY()][next.getX()]) {
            path.add(next);
            visited.add(next);
            dfs(next, map, path, result, moveOffset);
        }
    }

    // 当前无路可走时保存最长路径
    if (visited.isEmpty()) {
        if (path.size() > result.size()) {
            result.clear();
            result.addAll(path);
        }
    }

    // 当周围节点都不能行走时回退到上一步
    for (Pos neighbour : neighbours) {
        if (canPath(map, path, neighbour, visited)) {
            return;
        }
    }
    path.remove(pos);

}

/**
 * 判断当前节点是否可以行走
 */
private static boolean canPath(boolean[][] map, List<Pos> path, Pos pos, List<Pos> visited) {

    // 不在地图里，不能行走
    if (!inMap(map, pos)) {
        return false;
    }

    // 空白格子，不能行走
    if (!map[pos.getY()][pos.getX()]) {
        return false;
    }

    // 已经在路径中或经过，不能行走
    if (path.contains(pos) || visited.contains(pos)) {
        return false;
    }

    return true;
}

/**
 * 判断当前节点是否在地图内
 */
private static boolean inMap(boolean[][] map, Pos pos) {

    if (pos.getY() < 0 || pos.getY() >= map.length) {
        return false;
    }

    if (pos.getX() < 0 || pos.getX() >= map[0].length) {
        return false;
    }

    return true;

}
```

接下来，就让我们在主函数里验证一下结果吧！

```
public static void main(String[] args) {

    // 初始化参数
    boolean[][] simpleMap = new boolean[][] {
        {false, false, false, false, false, false, false, false, false},
        {false, false, false, false, false, false, true , true , false},
        {false, false, false, true , false, false, true , true , false},
        {false, false, true , false, false, false, false, false, false},
        {false, false, true , false, false, false, false, false, false},
        {false, false, true , false, false, false, false, false, false},
        {false, false, false, true , false, true , false, false, false},
        {false, false, false, false, true , true , false, false, false},
        {false, false, false, false, false, false, false, false, false}
    };
    Pos[] moveOffset = new Pos[] {
        new Pos(-1,  0),    // 向左移动
        new Pos(-1, -1),    // 向左上移动
        new Pos( 0, -1),    // 向上移动
        new Pos( 1, -1),    // 向右上移动
        new Pos( 1,  0),    // 向右移动
        new Pos( 1,  1),    // 向右下移动
        new Pos( 0,  1),    // 向下移动
        new Pos(-1,  1)     // 向左下移动
    };
    Pos start = new Pos(3, 3);

    // 执行深度优先算法
    List<Pos> longestPath = getLongestPathByDFS(simpleMap, start, moveOffset);

    // 打印路径
    System.out.print(longestPath);

}
```

执行Main函数之后，控制台将输出`[Pos{x=3, y=2}, Pos{x=2, y=3}, Pos{x=2, y=4}, Pos{x=2, y=5}, Pos{x=3, y=6}, Pos{x=4, y=7}, Pos{x=5, y=6}, Pos{x=5, y=7}]`，即行走的最长路径。

<span id="complexMap"></span>

虽然深度优先搜索算法可以计算出最长路径，但是它的时间复杂度却高得惊人！已知每次可以走8步，最多可以走m × n步（地图的长和宽），那么时间复杂度就是 O（8<sup>mn</sup>）。由于我们的地图可以走的选择比较单一，所以在我的电脑上`1ms`就可以算出结果。感兴趣的童鞋可以试试这个地图在你们的电脑上需要多久出结果：



```
boolean[][] complexMap = new boolean[][] {
    {false, true,  true,  false, false, true,  true,  false, true},
    {true,  false, false, false, true,  false, false, false, true},
    {true,  true,  false, false, true,  true,  false, false, false},
    {false, true,  true,  false,  false, true,  true,  true,  false},
    {false, true,  true,  false, false, true,  true,  false, true},
    {true,  false, false, false, true,  false, false, false, true},
    {true,  true,  false, false, true,  true,  false, false, false},
    {false, true,  true,  true,  false, true,  true,  true,  false},
    {false, true,  true,  false, false, true,  true,  false, true}
};
```


至少，在我的电脑上需要`5471ms`才能得出结果，非常的夸张！由于产品线比赛要求每次计算时间不能超过`1000ms`，所以使用该算法基本不可行。那么是否有时间更快的算法呢？别走开，答案就在下面。

--- 

# 贪心算法

## 算法思想

贪心算法采用的是这样一种思想：每次都走出路最少的格子，这样，后面的步骤可选择的余地就大，最优解的概率也就大的多。

## 代码示例

下面便是贪心算法的代码：

```

/**
 * 通过贪心算法获取最长路径
 * @param map 地图
 * @param start 起点
 * @param moveOffset 方向偏移量
 * @return 最长路径
 */
public static List<Pos> getLongestPathByChain(boolean[][] map, Pos start, Pos[] moveOffset) {

    List<Pos> longestPath = new ArrayList<>();
    chain(start, map, new ArrayList<>(), longestPath, moveOffset);
    return longestPath;

}


/**
 * 递归实现贪心算法
 * @param pos 当前节点
 * @param map 地图
 * @param path 当前路径
 * @param result 最终结果
 * @param moveOffset 方向偏移量
 */
private static void chain(Pos pos, boolean[][] map, List<Pos> path, List<Pos> result, Pos[] moveOffset) {

    // 获取出路最小的节点
    Pos minWayPos = getMinWayPos(pos, map, moveOffset);

    if (minWayPos != null) {
        // 递归搜寻路径
        path.add(minWayPos);
        map[minWayPos.getY()][minWayPos.getX()] = false;
        chain(minWayPos, map, path, result, moveOffset);
    } else {
        // 当前无路可走时保存最长路径
        if (path.size() > result.size()) {
            result.clear();
            result.addAll(path);
        }
    }

}

/**
 * 获取当前节点周围最小出路的节点
 */
private static Pos getMinWayPos(Pos pos, boolean[][] map, Pos[] moveOffset) {

    int minWayCost = Integer.MAX_VALUE;
    List<Pos> minWayPoss = new ArrayList<>();

    for (int i = 0; i < moveOffset.length; i++) {
        Pos next = new Pos(pos.getX() + moveOffset[i].getX(), pos.getY() + moveOffset[i].getY());
        if (inMap(map, next) && map[next.getY()][next.getX()]) {
            int w = -1;
            for (int j = 0; j < moveOffset.length; j++) {
                Pos nextNext = new Pos(next.getX() + moveOffset[j].getX(), next.getY() + moveOffset[j].getY());
                if (inMap(map, nextNext) && map[nextNext.getY()][nextNext.getX()]) {
                    w++;
                }
            }
            if (minWayCost > w) {
                minWayCost = w;
                minWayPoss.clear();
                minWayPoss.add(next);
            } else if (minWayCost == w) {
                minWayPoss.add(next);
            }
        }
    }

    if (minWayPoss.size() != 0) {
        // 随机返回一个最小出路的节点
        return minWayPoss.get((int) (Math.random() * minWayPoss.size()));
    } else {
        return null;
    }

}
```

写好算法之后，再验证一下结果！

```
public static void main(String[] args) {

    // 初始化参数
    boolean[][] simpleMap = new boolean[][] {
        {false, false, false, false, false, false, false, false, false},
        {false, false, false, false, false, false, true , true , false},
        {false, false, false, true , false, false, true , true , false},
        {false, false, true , false, false, false, false, false, false},
        {false, false, true , false, false, false, false, false, false},
        {false, false, true , false, false, false, false, false, false},
        {false, false, false, true , false, true , false, false, false},
        {false, false, false, false, true , true , false, false, false},
        {false, false, false, false, false, false, false, false, false}
    };
    Pos[] moveOffset = new Pos[] {
        new Pos(-1,  0),    // 向左移动
        new Pos(-1, -1),    // 向左上移动
        new Pos( 0, -1),    // 向上移动
        new Pos( 1, -1),    // 向右上移动
        new Pos( 1,  0),    // 向右移动
        new Pos( 1,  1),    // 向右下移动
        new Pos( 0,  1),    // 向下移动
        new Pos(-1,  1)     // 向左下移动
    };
    Pos start = new Pos(3, 3);

    // 执行贪心算法
    List<Pos> longestPath = getLongestPathByChain(simpleMap, start, moveOffset);

    // 打印路径
    System.out.print(longestPath);

}
```

执行Main函数之后，控制台将输出`[Pos{x=3, y=2}, Pos{x=2, y=3}, Pos{x=2, y=4}, Pos{x=2, y=5}, Pos{x=3, y=6}, Pos{x=4, y=7}, Pos{x=5, y=7}, Pos{x=5, y=6}]`，路径长度与深度优先搜索算法一致，即也能找到最长路径。

那么在复杂一点的地图上，与深度优先搜索相比，贪心算法的结果怎么样呢？在我的机器上，计算结果如下：

\ | [simpleMap](#simpleMap) | [complexMap](#complexMap) 
----|------|---- 
深度优先搜索算法 | 最长路径为8步，计算时间为1ms  | 最长路径为33步，计算时间为5254ms 
贪心算法 |  最长路径为8步，计算时间为1ms  | 最长路径为4/9/20步，计算时间为1ms 

从结果上可以发现，由于贪心算法并没有遍历所有路径，而是每次都往出路最少的格子走，所以时间上快很多，但是其结果却非常地不稳定，这是因为贪心算法容易陷入局部最优解的情况！

显然如果用贪心算法来寻路吃果实，那么能不能打败敌人就要靠运气了。怎么办？是否有折中一点的算法，既耗费可接受的时间，又可以计算出较好的结果呢？答案还是肯定的，接下来就介绍更高端的算法——模拟退火算法。

---

# 模拟退火算法

## 算法思想



## 代码示例

---

# 总结

其实求最长路径问题可以看成是`哈密尔顿路径问题`，但由于寻找哈密尔顿路径是一个典型的NPC问题，所以不能在多项式时间内得到最优解。我们可以通过深度优先搜索得到最优解，也可以通过贪心算法得到一个局部最优解，还可以通过模拟退火算法得到一个近似解。

---

# 参考阅读

[[1] 哈密顿图 - 维基百科](https://zh.wikipedia.org/wiki/%E5%93%88%E5%AF%86%E9%A1%BF%E5%9B%BE)

[[2] 贪婪算法求解哈密尔顿路径问题 - 51CTO博客](http://mengliao.blog.51cto.com/876134/539522/)

[[3] 什么是P问题、NP问题和NPC问题](http://www.matrix67.com/blog/archives/105)

[[4] 最长路径问题 - 维基百科](https://zh.wikipedia.org/wiki/%E6%9C%80%E9%95%BF%E8%B7%AF%E5%BE%84%E9%97%AE%E9%A2%98)
