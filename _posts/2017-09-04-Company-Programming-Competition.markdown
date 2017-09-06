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


> 从深度优先搜索到贪心算法。

# 前言

最近产品线举办了一个软件编程大赛，题目非常的有趣，就是在一个9 × 9的格子里，你要和另一个敌人PK，在PK的过程中，你可以吃格子里的果实来提升攻击力。每次可以往正上、正下、正左、正右、左上、左下、右上、右下八个方向走。每次要么连续吃果实要么连续走空白区域，且不能走重复的位置。初始状态如下图所示：

![](http://o7x0ygc3f.bkt.clouddn.com/%E6%9C%80%E9%95%BF%E8%B7%AF%E5%BE%84%E9%97%AE%E9%A2%98-5.png)

为了尽可能地吃最多的果实，我们的路线可以这样规划：

![](http://o7x0ygc3f.bkt.clouddn.com/%E6%9C%80%E9%95%BF%E8%B7%AF%E5%BE%84%E9%97%AE%E9%A2%98-6.png)

所以，我们可以把这个问题归结为：已知空白区域不能走，每次可以往正上、正下、正左、正右、左上、左下、右上、右下八个方向走，走过的位置不能再走，求能吃最多果实的路线（最长路径问题）？

---

# 前置条件

首先我们将上面的地图使用布尔类型的二维数组表示，其中true表示可以行走的格子，false表示不能行走的格子：

```
boolean[][] map = new boolean[][] {
    {false, false, false, false, false, false, false, false, false},
    {false, false, false, false, false, false, true , true , false},
    {false, false, false, true , false, false, true , true , false},
    {false, false, true , true , false, false, false, false, false},
    {false, false, true , false, false, false, false, false, false},
    {false, false, true , false, false, false, false, false, false},
    {false, false, false, true , false, true , false, false, false},
    {false, false, false, false, true , true , false, false, false},
    {false, false, false, false, false, false, false, false, false}
};
```

对于地图上的每一个点，我们用一个简单类来表示：

```
public class Pos {

    private int x;  // 横坐标
    private int y;  // 纵坐标
    
    // get、set、construct方法省略
    
}
```

由于我们是使用横纵坐标而不是几行几列来表示一个点（没错，我就是这么傲娇），那么我们就需要给地图定义横纵坐标方向。方向如下图所示：

![](http://o7x0ygc3f.bkt.clouddn.com/%E6%9C%80%E9%95%BF%E8%B7%AF%E5%BE%84%E9%97%AE%E9%A2%98-8.png)

那么起点上方的果实坐标就是[3, 2]（横坐标为3，纵坐标为2），但是对应着二维数组为map[2][3]（第二行，第三列），即横坐标对应着二维数组的列，纵坐标对应着二维数组的行。

---

# 深度优先搜索算法

拿到这道题，脑袋里第一个想到的就是深度优先搜索算法，每次往八个方向递归，当不能继续走下去的时候保存路径，并回退到能继续行走的点，继续递归直到结束。接下来就是我们的深度优先搜索代码：

```
/**
 * 深度优先搜索
 * @param pos 当前节点
 * @param map 地图
 * @param path 当前路径
 * @param result 所有路径结果集
 */
public static void dfs(Pos pos, boolean[][] map, List<Pos> path, List<List<Pos>> result) {

    // 将当前节点加入当前路径
    path.add(pos);

    // 设置当前节点不能再经过
    map[pos.getY()][pos.getX()] = false;

    // 记录当前八个方向的节点是否经过
    boolean[] through = new boolean[8];

    // 向左行走
    Pos left = new Pos(pos.getX() - 1, pos.getY());
    if (inMap(map, left) && !visited(map, left)) {
        through[0] = true;
        dfs(left, map, path, result);
        map[left.getY()][left.getX()] = true;
    }

    // 向左上行走
    Pos leftUp = new Pos(pos.getX() - 1, pos.getY() - 1);
    if (inMap(map, leftUp) && !visited(map, leftUp)) {
        through[1] = true;
        dfs(leftUp, map, path, result);
        map[leftUp.getY()][leftUp.getX()] = true;
    }

    // 向上行走
    Pos up = new Pos(pos.getX(), pos.getY() - 1);
    if (inMap(map, up) && !visited(map, up)) {
        through[2] = true;
        dfs(up, map, path, result);
        map[up.getY()][up.getX()] = true;
    }

    // 向右上行走
    Pos rightUp = new Pos(pos.getX() + 1, pos.getY() - 1);
    if (inMap(map, rightUp) && !visited(map, rightUp)) {
        through[3] = true;
        dfs(rightUp, map, path, result);
        map[rightUp.getY()][rightUp.getX()] = true;
    }

    // 向右行走
    Pos right = new Pos(pos.getX() + 1, pos.getY());
    if (inMap(map, right) && !visited(map, right)) {
        through[4] = true;
        dfs(right, map, path, result);
        map[right.getY()][right.getX()] = true;
    }

    // 向右下行走
    Pos rightDown = new Pos(pos.getX() + 1, pos.getY() + 1);
    if (inMap(map, rightDown) && !visited(map, rightDown)) {
        through[5] = true;
        dfs(rightDown, map, path, result);
        map[rightDown.getY()][rightDown.getX()] = true;
    }

    // 向下行走
    Pos down = new Pos(pos.getX(), pos.getY() + 1);
    if (inMap(map, down) && !visited(map, down)) {
        through[6] = true;
        dfs(down, map, path, result);
        map[down.getY()][down.getX()] = true;
    }

    // 向左下行走
    Pos leftDown = new Pos(pos.getX() - 1, pos.getY() + 1);
    if (inMap(map, leftDown) && !visited(map, leftDown)) {
        through[7] = true;
        dfs(leftDown, map, path, result);
        map[leftDown.getY()][leftDown.getX()] = true;
    }



    // 检查是否已无路可走
    if ((visited(map, left) || through[0]) && (visited(map, leftUp) || through[1]) &&
            (visited(map, up) || through[2]) && (visited(map, rightUp) || through[3]) &&
            (visited(map, right) || through[4]) && (visited(map, rightDown) || through[5]) &&
            (visited(map, down) || through[6]) && (visited(map, leftDown) || through[7])) {

        // 周围能走的节点都经过时表示无路可走，保存路径
        if (!through[0] && !through[1] && !through[2] && !through[3] && !through[4] && !through[5] && !through[6] && !through[7]) {
            result.add(new ArrayList<>(path));
        }
            
        // 当前正在回退，需要将当前节点移除当前路径
        path.remove(pos);
    }


}
```

上述算法还需要下面两个辅助函数，一个检查当前节点是否已经经过，一个检查当前节点是否在地图内：

```

/**
 * 检查当前节点是否已经经过
 */
private static boolean visited(boolean[][] map, Pos pos) {

    // 不在图的点默认走过
    if (!inMap(map, pos)) {
        return true;
    }

    // true表示没有走过
    if (map[pos.getY()][pos.getX()]) {
        return false;
    } else {
        return true;
    }

}

/**
 * 检查当前节点是否在地图内
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
