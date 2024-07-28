#### CP (Consistency and Partition Tolerance)
Sacrifices availability. For example, **MongoDB** nodes communicate to check statuses. If a node doesn't respond for a few seconds, it's considered "**unavailable**." If the primary node is unavailable, a secondary node is promoted to primary with the election. All persistence requests are **unavailable until the election finishes**.

![](https://velog.velcdn.com/images/jollidah/post/308b77e6-96f9-43f8-9dd8-3ac15d34fbf9/image.png)
