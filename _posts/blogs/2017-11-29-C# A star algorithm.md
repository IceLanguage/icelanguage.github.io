---
layout: page
title: C# A星寻路算法
category: 
    - blogs
---

```cs
using System;
using System.Collections.Generic;

public struct _mapPoint
{
    public int x;
    public int y;
}
class FindWay
{
    class Point
    {
        public Point() { }
        public Point(_mapPoint m) { x = m.x; y = m.y; }
        public int G;
        public int H;
        public int x;
        public int y;
        public Point father;
    }

    //开启列表
    static List<Point> open_List = new List<Point>();
    //关闭列表
    static List<Point> close_List = new List<Point>();

    /// <summary>
    /// A*寻路算法
    /// </summary>
    /// <param name="startPoint"></param>
    /// <param name="endPoint"></param>
    /// <param name="movePower"></param>
    /// <param name="consumption"></param>
    /// <returns></returns>
    public static List<_mapPoint> Astar(_mapPoint startPoint, _mapPoint endPoint, int movePower, ref byte[,] consumption)
    {
        //定义出发位置
        Point startingPoint = new Point(startPoint);

        //定义目的地
        Point destination = new Point(endPoint);

        open_List.Add(startingPoint);
        while (!(IsInOpenList(destination.x, destination.y))) //当终点存在开启列表中就意味着寻路结束了
        {
            Point nowPoint = GetMinFFromOpenList();
            open_List.Remove(nowPoint);
            close_List.Add(nowPoint);
            CheckGrid(nowPoint, destination, ref consumption);
        }

        List<_mapPoint> way = new List<_mapPoint>();

        Point p = GetPointFromOpenList(destination.x, destination.y);
        List<Point> temp = new List<Point>();
        while (p.father != null)
        {
            temp.Add(p);
            p = p.father;
        }
        temp.Reverse();
        foreach (Point pt in temp)
        {
            if (pt.G > movePower) break;
            way.Add(new _mapPoint() { x = pt.x, y = pt.y });
        }
        open_List.Clear();
        close_List.Clear();
        return way;
    }

    //从开启列表查找F值最小的节点
    private static Point GetMinFFromOpenList()
    {
        Point Pmin = null;
        foreach (Point p in open_List) if (Pmin == null || Pmin.G + Pmin.H > p.G + p.H) Pmin = p;
        return Pmin;
    }
    //判断关闭列表是否包含一个坐标的点
    private static bool IsInCloseList(int x, int y)
    {
        Point p = close_List.Find(item => item.x == x && item.y == y);
        if (p!=null) return true;
        
        return false;
    }
    //判断开启列表是否包含一个坐标的点
    private static bool IsInOpenList(int x, int y)
    {
        Point p = open_List.Find(item => item.x == x && item.y == y);
        if (p!=null) return true;
       
        return false;
    }
    //从开启列表返回对应坐标的点
    private static Point GetPointFromOpenList(int x, int y)
    {
        foreach (Point p in open_List) if (p.x == x && p.y == y) return p;
        return null;
    }
    //检查当前节点附近的节点
    private static void CheckGrid(Point nowPoint, Point destination, ref byte[,] consumption)
    {
        for (int xt = nowPoint.x - 1; xt <= nowPoint.x + 1; xt++)
        {
            for (int yt = nowPoint.y - 1; yt <= nowPoint.y + 1; yt++)
            {
                //排除超过边界、不相干和关闭列表中的点
                if (!(xt >= 0 && xt < consumption.GetLength(0) && yt >= 0 && yt < consumption.GetLength(1))) continue;
                if (IsInCloseList(xt, yt)) continue;
                if (!(xt == nowPoint.x && yt - 1 == nowPoint.y) && !(xt - 1 == nowPoint.x && yt == nowPoint.y) && !(xt + 1 == nowPoint.x && yt == nowPoint.y) && !(xt == nowPoint.x && yt + 1 == nowPoint.y))
                    continue;

                if (IsInOpenList(xt, yt))
                {
                    Point pt = GetPointFromOpenList(xt, yt);
                    int cost = CalculateConsumption(ref consumption, pt, nowPoint);
                    int G_new = nowPoint.G + cost;
                    if (G_new < pt.G)
                    {
                        open_List.Remove(pt);
                        pt.father = nowPoint;
                        pt.G = G_new;
                        open_List.Add(pt);
                    }
                }
                else //不在开启列表中
                {
                    Point pt = new Point();
                    pt.x = xt;
                    pt.y = yt;
                    pt.father = nowPoint;
                    int cost = CalculateConsumption(ref consumption, pt, nowPoint);
                    pt.G = pt.father.G + cost;
                    pt.H = Math.Abs(pt.x - destination.x) + Math.Abs(pt.y - destination.y);
                    open_List.Add(pt);
                }
            }
        }
    }

    /// <summary>
    /// 计算移动消耗
    /// </summary>
    /// <param name="consumption"></param>
    /// <param name="pt"></param>
    /// <param name="nowPoint"></param>
    /// <returns></returns>
    private static int CalculateConsumption(ref byte[,] consumption, Point pt,Point nowPoint)
    {
        int xt = nowPoint.x;
        int yt = nowPoint.y;
        int cost = consumption[pt.x, pt.y] - consumption[xt, yt];
        if (cost <= 1) cost = 1;
        else if (cost > 1 || cost < 2) cost *= Road.jumpCost;
        else cost *= Road.climbCost;
        return cost;
    }
}
```