# Course Schedule
## https://leetcode.com/problems/course-schedule

There are a total of numCourses courses you have to take, labeled from 0 to numCourses-1.

Some courses may have prerequisites, for example to take course 0 you have to first take course 1, which is expressed as a pair: [0,1]

Given the total number of courses and a list of prerequisite pairs, is it possible for you to finish all courses?

```
Example 1:

Input: numCourses = 2, prerequisites = [[1,0]]
Output: true
Explanation: There are a total of 2 courses to take. 
             To take course 1 you should have finished course 0. So it is possible.
Example 2:

Input: numCourses = 2, prerequisites = [[1,0],[0,1]]
Output: false
Explanation: There are a total of 2 courses to take. 
             To take course 1 you should have finished course 0, and to take course 0 you should
             also have finished course 1. So it is impossible.
``` 

**Constraints:**

1. The input prerequisites is a graph represented by a list of edges, not adjacency matrices. Read more about how a graph is represented.
2. You may assume that there are no duplicate edges in the input prerequisites.
3. 1 <= numCourses <= 10^5

## Implementation :

```java
class Solution {
    public boolean canFinish(int numCourses, int[][] prerequisites) {
        int[] inDegrees = new int[numCourses];
        int count = 0;
        for(int i = 0; i < prerequisites.length; i++) {
            inDegrees[prerequisites[i][0]]++;
        }
        
        Stack<Integer> stack = new Stack<>();
        for(int i = 0; i < inDegrees.length; i++) {
            if(inDegrees[i] == 0) {
                stack.push(i);
            }
        }
        
        while(!stack.isEmpty()) {
            int course = stack.pop();
            count++;
            for(int i = 0; i < prerequisites.length; i++) {
                if(prerequisites[i][1] == course) {
                    inDegrees[prerequisites[i][0]]--;
                    if(inDegrees[prerequisites[i][0]] == 0) {
                        stack.push(prerequisites[i][0]);
                    }
                }
            }
        }
        return numCourses == count;
    }
}
```

# References :
1. https://www.youtube.com/watch?v=0LjVxtLnNOk
2. https://www.youtube.com/watch?v=z-mB8ZL6mjo (Topological Sort)
