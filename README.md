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

# Approach :
Push a course to stack only when all its prerequisites are completed, which means If indegree for this course is 0.

While stack is not empty, pop a course from the stack , increment the course completed counter. And also update the indegree of the course for which this current course was a prerequisite. Now if the indegree for any course becomes zero as we update the indegrees, add that course to stack for which indegree reached to zero.

Finally when the stack is empty, the value of the course completed counter should be equal to number of courses If we were able to complete all the courses. But If the value of the course completed counter is not equal to number of courses, it means its not possible to complete all the courses. 

## Implementation 1 : DFS (Stack)

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

## Implementation 2 : BFS (Queue) , Runtime : O(n * m)
```java
class Solution {
    public boolean canFinish(int numCourses, int[][] prerequisites) {
        int[] inDegrees = new int[numCourses];
    
        int count = 0;
        for(int i = 0; i < prerequisites.length; i++) {
            inDegrees[prerequisites[i][0]]++;
        }
        
        Queue<Integer> queue = new LinkedList<>();
        for(int i = 0; i < inDegrees.length; i++) {
            if(inDegrees[i] == 0) {
                queue.offer(i);
            }
        }
        
        while(!queue.isEmpty()) {
            int course = queue.poll();
            count++;
            for(int i = 0; i < prerequisites.length; i++) {
                if(prerequisites[i][1] == course) {
                    inDegrees[prerequisites[i][0]]--;
                    if(inDegrees[prerequisites[i][0]] == 0) {
                        queue.offer(prerequisites[i][0]);
                    }
                }
            }
        }
        return numCourses == count;
    }
}
```

## Implementation 2a (using Adjacency list) : BFS , Runtime = O(n + m)

```java
class Solution {
    public boolean canFinish(int numCourses, int[][] prerequisites) {
        List<List<Integer>> adjacencyList = new ArrayList<>();
        int[] indegree = new int[numCourses];
        Queue<Integer> queue = new ArrayDeque<>();
        int courseCompleted = 0;

        for(int i = 0; i < numCourses; i++)
            adjacencyList.add(new ArrayList<>());

        for(int[] prerequisite : prerequisites) {
            int a = prerequisite[0];
            int b = prerequisite[1];
            indegree[a]++;
            adjacencyList.get(b).add(a);
        }
        for(int i = 0; i < numCourses; i++){
            if(indegree[i] == 0)
              queue.offer(i);
        }

        while(!queue.isEmpty()) {
            int course = queue.remove();
            courseCompleted++;
            for(int neighbor : adjacencyList.get(course)){
                indegree[neighbor]--;
                if(indegree[neighbor] == 0)
                  queue.offer(neighbor);
            }
        }
        return courseCompleted == numCourses;
    }
}
```

# Implementation 3 : Kosaraju's Algorithm (SCC size)
Idea is to use Kosaraju'a algorithm to find the size of Strongly Connected components (SCC), if we find a SCC whose size is greater than 1, it means it have cycle, in this case return false. And if we don't find any SCC with size greater than 1, it means all the constraints can be fulfilled, and we return true.
```java
class Solution {
    public boolean canFinish(int numCourses, int[][] prerequisites) {
        
        Map<Integer,List<Integer>> graph = new HashMap<>();
        for(int[] prerequisite : prerequisites) {
            int u = prerequisite[1];
            int v = prerequisite[0];
            if(u == v)
                return false;
            graph.putIfAbsent(u, new ArrayList<Integer>());
            graph.get(u).add(v);
        }
        
        // Kosaraju's algorithm
        Stack<Integer> stack = new Stack<>();
        boolean[] visited = new boolean[numCourses];
        for(int v = 0; v < numCourses; v++) {
            if(!visited[v]) {
              dfs1(v,graph,visited,stack);   
            }
        }
        // Transpose the edges
        Map<Integer,List<Integer>> reversedGraph = new HashMap<>();
        for(int u : graph.keySet()) {
            for(int v : graph.get(u)) {
                reversedGraph.putIfAbsent(v, new ArrayList<Integer>());
                reversedGraph.get(v).add(u);
            }
        }
        
        visited = new boolean[numCourses];
        while(!stack.isEmpty()) {
            int v = stack.pop();
            if(!visited[v]) {
                int size = dfs2(v, reversedGraph, visited);
                if(size > 1)
                    return false;
            }
        }
       return true; 
    }
    
    private void dfs1(int vertex, Map<Integer,List<Integer>> graph, 
                      boolean[] visited, Stack<Integer> stack) {
        if(!visited[vertex]) {
            visited[vertex] = true;
            List<Integer> neighbors = graph.get(vertex);
            if(neighbors != null) {
              for(int neighbor : neighbors) {
                  dfs1(neighbor,graph,visited,stack);
              }   
            }
            stack.push(vertex);
        }
    }
    
    private int dfs2(int vertex, Map<Integer,List<Integer>> graph, boolean[] visited) {
        int size = 0;
        if(!visited[vertex]) {
            size++;
            visited[vertex] = true;
            List<Integer> neighbors = graph.get(vertex);
            if(neighbors != null) {
                for(int neighbor : neighbors) {
                   size += dfs2(neighbor, graph, visited);
                }
            }
        }
        return size;
    }
}
```

# References :
1. https://www.youtube.com/watch?v=0LjVxtLnNOk
2. https://www.youtube.com/watch?v=z-mB8ZL6mjo (Topological Sort)
3. https://leetcode.com/problems/course-schedule/discuss/249688/Different-O(V%2BE)-solution-using-Kosaraju's-algorithm
4. https://github.com/eMahtab/kosaraju-algorithm
