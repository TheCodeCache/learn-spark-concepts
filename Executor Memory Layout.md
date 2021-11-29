**--executor-memory = shuffle memory + User memory + Reserved Memory(300MB) + Memory Buffer (Yarn Overhead)**

User memory us further subdivided into:
- Storage Memory
- Execution Memory


![image](https://user-images.githubusercontent.com/26399543/143792740-26482c61-1608-4064-9d2f-e086f2757f3a.png)

Off-heap memory:
- This is also called as Native Memory (taken directly from underlying OS)


![image](https://user-images.githubusercontent.com/26399543/143792986-d01ce8da-9acd-44e6-bfc6-0aa6328e437e.png)

**Reference:**  
1. https://www.youtube.com/watch?v=5dga0UT4RI8

