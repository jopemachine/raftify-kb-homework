### AP (Availability and Partition Tolerance)
Sacrifices consistency. For example, in **Cassandra**, all nodes can write and read without a primary node, replicating data to adjacent nodes as many times as specified. Even if a connection is lost, nodes can **still write and read data without ensuring consistency**. Cassandra later recovers this through `Eventual Consistency`.
![](https://velog.velcdn.com/images/jollidah/post/32ff37e8-6fa9-4b43-8fa9-ad7f77800f01/image.png)
