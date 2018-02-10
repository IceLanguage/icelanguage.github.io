---
layout: page
title: union-find算法
category: 
    - blogs
---

这个算法主要用来判断2点是否连通
在博客记录下来，方便以后能用
```
public class WeightedQuickUnionUF {
    private int[] parent;   // parent[i] = parent of i
    private int[] size;     // size[i] = number of sites in subtree rooted at i
    private int count;      // number of components

  
    public WeightedQuickUnionUF(int n) {
        count = n;
        parent = new int[n];
        size = new int[n];
        for (int i = 0; i < n; i++) {
            parent[i] = i;
            size[i] = 1;
        }
    }

  
    public int count() {
        return count;
    }
  
    
    public int find(int p) {
        validate(p);
        while (p != parent[p])
            p = parent[p];
        return p;
    }

    // validate that p is a valid index
    private void validate(int p) {
        int n = parent.length;
        if (p < 0 || p >= n) {
            throw new IllegalArgumentException("index " + p + " is not between 0 and " + (n-1));  
        }
    }

   
    public boolean connected(int p, int q) {
        return find(p) == find(q);
    }

    
    public void union(int p, int q) {
        int rootP = find(p);
        int rootQ = find(q);
        if (rootP == rootQ) return;

        // make smaller root point to larger one
        if (size[rootP] < size[rootQ]) {
            parent[rootP] = rootQ;
            size[rootQ] += size[rootP];
        }
        else {
            parent[rootQ] = rootP;
            size[rootP] += size[rootQ];
        }
        count--;
    }


   
    public static void main(String[] args) {
        int n = StdIn.readInt();
        WeightedQuickUnionUF uf = new WeightedQuickUnionUF(n);
        while (!StdIn.isEmpty()) {
            int p = StdIn.readInt();
            int q = StdIn.readInt();
            if (uf.connected(p, q)) continue;
            uf.union(p, q);
            StdOut.println(p + " " + q);
        }
        StdOut.println(uf.count() + " components");
    }

}
```
使用实例Coursera大作业week1
Percolation .java
```


import edu.princeton.cs.algs4.StdRandom;
import edu.princeton.cs.algs4.StdStats;
import edu.princeton.cs.algs4.WeightedQuickUnionUF;

public class Percolation {
	
	private WeightedQuickUnionUF uf;  
	private WeightedQuickUnionUF uf_backwash;
    private int N;  
    private boolean[] arrayOpen; 
    private int openSiteNumber;
    
	public Percolation(int n)
	{
        if(n<0||n==-2147483648)
        {
            throw new IllegalArgumentException();  
        }
		this.N=n;
		this.uf = new WeightedQuickUnionUF((N+1)*(N+1)); 
		this.uf_backwash = new WeightedQuickUnionUF(N*N+N+1);  
		this.arrayOpen=new boolean[(N+1)*(N+1)];
		this.openSiteNumber=0;
		
		for (int i=1; i<=N; i++){  
            uf.union(0*N+1, 0*N+i);  
            uf_backwash.union(0*N+1, 0*N+i);  
            arrayOpen[0*N+i] = true;  
            uf.union((N+1)*N+1, (N+1)*N+i);  
            arrayOpen[(N+1)*N+i] = true;  
        } 
		
	}
	public void open(int row, int col)
	{
		if (row < 1 || row > N){  
            throw new IndexOutOfBoundsException("row index " + row + " out of bounds");  
        }  
        if (col < 1 || col > N){  
            throw new IndexOutOfBoundsException("row index " + col+ " out of bounds");  
        }
        if (arrayOpen[row*N+col]){  
            return;  
        }
        openSiteNumber++;
        arrayOpen[row*N+col] = true; 
        
        if (arrayOpen[(row-1)*N+col]){  
            uf.union(row*N+col, (row-1)*N+col);  
            uf_backwash.union(row*N+col, (row-1)*N+col);  
        }  
        if (arrayOpen[(row+1)*N+col]){  
            uf.union(row*N+col, (row+1)*N+col);  
            if (row!=N){  
                uf_backwash.union(row*N+col, (row+1)*N+col);  
            }  
        }  
        if (col!=1 && arrayOpen[row*N+col-1]){  
            uf.union(row*N+col, row*N+col-1);  
            uf_backwash.union(row*N+col, row*N+col-1);  
        }  
        if (col!=N && arrayOpen[row*N+col+1]){  
            uf.union(row*N+col, row*N+col+1);  
            uf_backwash.union(row*N+col, row*N+col+1);  
        } 
        
	}
	public boolean isOpen(int row, int col)
	{
		if (row < 1 || row > N){  
            throw new IndexOutOfBoundsException("row index " + row + " out of bounds");  
        }  
        if (col < 1 || col > N){  
            throw new IndexOutOfBoundsException("row index " + col+ " out of bounds");  
        }
        return arrayOpen[row*N+col];
	}
	public boolean isFull(int row, int col)
	{
		if (row < 1 || row > N){  
            throw new IndexOutOfBoundsException("row index " + row + " out of bounds");  
        }  
        if (col < 1 || col > N){  
            throw new IndexOutOfBoundsException("row index " + col+ " out of bounds");  
        } 
        return uf_backwash.connected(row*N+col, 0*N+1) && arrayOpen[row*N+col]; 
	}
	public int numberOfOpenSites()
	{
		return openSiteNumber;
	}
	public boolean percolates()
	{
		return uf.connected(0*N+1, (N+1)*N+1);
	}
	public static void main(String[] args)
	{
		
	}
}

```

PercolationStats.java
```


import edu.princeton.cs.algs4.StdIn;
import edu.princeton.cs.algs4.StdOut;
import edu.princeton.cs.algs4.StdRandom;
import edu.princeton.cs.algs4.StdStats;
import edu.princeton.cs.algs4.WeightedQuickUnionUF;

public class PercolationStats {
	
	private double recT[];  
    private double res_mean;  
    private double res_stddev;  
    private int N; 
    
	public PercolationStats(int n, int trials) 
	{
		this.recT = new double[trials];  
        this.N = n;  
         
        int times = 0; 
       if (n <= 0){  
            throw new IllegalArgumentException();  
        }  
        if (trials <= 0){  
            throw new IllegalArgumentException();  
        }   
        this.recT = new double[trials];  
        while (times < trials){  
            Percolation percolation = new Percolation(N);  
            boolean[][] arrOpen = new boolean[N+1][N+1];  
            int count = 0;  
            while(true){  
                count++;  
                while(true){  
                    int x = StdRandom.uniform(N) + 1;  
                    int y = StdRandom.uniform(N) + 1;  
                    if (arrOpen[x][y]){  
                        continue;  
                    }else{  
                        percolation.open(x, y);  
                        arrOpen[x][y] = true;  
                        break;  
                    }  
                }  
                if (percolation.percolates()){  
                    recT[times] = (double)count / ((double)N * (double)N);  
                    break;  
                }  
            }  
            times++;  
        }  
        this.res_mean = StdStats.mean(recT);  
        this.res_stddev = StdStats.stddev(recT);
	}
	public double mean()                        
	{
		return res_mean;
	}
	public double stddev()
	{
		return res_stddev;
	}
	public double confidenceLo()
	{
		return this.res_mean - 1.96*this.res_stddev / Math.sqrt(N);
	}
	public double confidenceHi() 
	{
		return this.res_mean + 1.96*this.res_stddev / Math.sqrt(N);
	}

	public static void main(String[] args)
	{
		int N = StdIn.readInt();  
        int T = StdIn.readInt();  
        PercolationStats percolationStats = new PercolationStats(N, T);  
        StdOut.println("mean = " + percolationStats.mean());  
        StdOut.println("stddev = " + percolationStats.stddev());  
        StdOut.println("95% confidence interval " + percolationStats.confidenceLo() + ", " + percolationStats.confidenceHi()); 
	}
}

```
