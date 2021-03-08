---
layout: single
title:  "Weekly Programming Update - 1"
date:   2021-03-04 07:49:23 +0100
categories: programming
excerpt: A weekly update of my progress to learning software development
---

The ultimate goal is for me to progress in learning software development, so I will be cataloguing weekly updates of my progress and what I have learned

Linked below is a new git repo that will eventually contain a single project per week for the next 6 months. All the code will be written in a mixture of Java and potentially some python or C. This is purely a place to log my progress as this blog has it's focus from ctf's to more programming as hackthebox took out far to much time and the lack of good windows boxes caused me to stop playing. 

[The Code Repo](https://github.com/71xn/code)

## Problem 1 - HackerRank ArrayList

### [GitHub Link](https://github.com/71xn/code/blob/main/HackerRank/HackerRank_ArrayList/src/Solution.java)

### [Prompt](https://www.hackerrank.com/challenges/java-arraylist/problem)


{% highlight java linenos %}


import java.io.*;
import java.util.*;

public class Solution {

    public static void main(String[] args) {
        /* Enter your code here. Read input from STDIN. Print output to STDOUT. Your class should be named Solution. */
        Scanner in = new Scanner(System.in);

        // Getting Numbers
        int n = in.nextInt();
        int d,q,x,y;
        ArrayList[] set = new ArrayList[n];
        for(int i=0;i<n;i++){
            d = in.nextInt();
            set[i] = new ArrayList();
            for(int j=0;j<d;j++){
                set[i].add(in.nextInt());
            }
        }
        // Getting Queries
        q=in.nextInt();
        for(int i=0;i<q;i++){
            x=in.nextInt();
            y=in.nextInt();
            try{
                System.out.println(set[x-1].get(y-1));
            } catch(Exception e){
                System.out.println("ERROR!");
            }
        }


    }
}


{% endhighlight %}