---
title: "Get Most Visited"
date: 2021-12-28
description: "Given two strings representing times of entry and exit from a car parking lot, find the cost of the ticket according to the given billing rules."
menu:
  sidebar:
    name: getMostVisited
    identifier: getMostVisited
    parent : coding-test
    weight: 10
hero: images/sprint.png 
---
## Task description:

Pat is an ordinary kid who works hard to be a great runner. As part of training, Pat must run sprints of different intervals on a straight trail. The trail has numbered markers that the coach uses as goals.
Pat's coach provides a list of goals to reach in order. Each time Pat starts at, stops at, or passes a marker it is considered a visit. 
Determine the lowest numbered marker that is visited the most times during Pat's day of training. 

## Write a function:
Complete the function getMostVisited in the given editor. The function must return an integer denoting Pat's most visited position on the trail after performing all m − 1 sprints. If there are multiple such answers, return the smallest one.

## Example:
n=5 sprints = [2, 4, 1, 3], means the number of markers on the trail is 5, and assigned sprints = [2, 4, 1, 3], Pat first sprints from position 2 -> 4. The next sprint is from position 4 -> 1, and then 1 -> 3. The total number of visits to each position in the example is calculated like so: Pat has visited markers 2 and 3 a total of 3 times each. Since 2<3, the lowest numbered marker that is visited the most times during Pat's day of training is 2. 

<!--- {{< img src="images/getMostVisited.png" align="center" title="Forest">}} --->

## Thought:
Take the first line 2 -> 4 as an example, the smaller number is the starting marker, the bigger one is the ending marker. 
int[] incremental represents the number of markers, every starting marker gets +1 record, and the ending marker's next number gets -1 record. By using the global variable "score" and looping through the incremental array, the markers between start and end all get +1 record. While The -1 after the ending marker will make the global score stops counting the current line. By doing this for every line, we'll get the scores of every marker.
My solution:
```java
public static int getMostVisited(int n, List<Integer> sprints) { 
        int[] incremental = new int[n + 2]; //if n=5, index 0 and 6 is the added two
        for (int i=0; i<sprints.size()-1; i++) {
            int start = Math.min(sprints.get(i), sprints.get(i+1));
            int end = Math.max(sprints.get(i+1), sprints.get(i));
            incremental[start] ++;
            incremental[end + 1] --;
        }
        int[] scores = new int[n+1];
        int score = 0;
        for (int i=1; i<n+1; i++) {
            score += incremental[i];
            scores[i] = score;
        }
        //Traverse and compare to find the most visited.
        int maxVal = 0, maxValIndex = -1;
        for (int i=1; i<n+1; i++) {
            if (scores[i] > maxVal) {
                maxVal = scores[i];
                maxValIndex = i;
            }
        }
        return maxValIndex;
    }
```