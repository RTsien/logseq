title:: 火影忍者手游实时pvp技术揭密（服务器篇） (highlights)
author:: [[woa.com]]
full-title:: "火影忍者手游实时pvp技术揭密（服务器篇）"
category:: #articles
url:: https://km.woa.com/articles/show/292113?kmref=search&from_page=1&no=6
summary:: The text discusses the technical details of real-time PvP in the mobile game Naruto. It explains the use of reliable UDP-based frame synchronization for PvP gameplay and the anti-cheating measures implemented by reporting player actions and statuses. Additionally, the text touches on the use of UDP over TCP for better performance in real-time gameplay.
![](https://readwise-assets.s3.amazonaws.com/static/images/article3.5c705a01b476.png)

- Highlights first synced by [[Readwise]] [[Mar 30th, 2024]]
	- 对于可靠性的保证，可以采用请求重传，而火影使用的是冗余重传。使用冗余重传的一个好处是，简化了麻烦的时序问题，并且收到的每个包都是完整的顺序的。对于网络拥塞情况下的带宽利用优于TCP，它不足之处是流量略微增加了些。下图是冗余重传的过程： ([View Highlight](https://read.readwise.io/read/01ht5wn236b9xzrq06c9jkp5sq))
	- •   Client发Action1过来，记seq=1，服务器未收到。
	  •   Client又新增了Action2，此时新包将同时包含Action1,Action2，并且seq=2。
	  •   Server确认了上一步骤的包，发给Client的包Ack=2表示确认。
	  •   Client由于某些原因（可能延迟等）尚未收到Server的Ack=2的确认，此时新增Action3，并发包seq=3。
	  •   Client再次发Action4时，发现之前已经Ack=2了，故新包将只带Action3，Action4并且seq=4。 ([View Highlight](https://read.readwise.io/read/01ht5wntd2mycmmxq29eejey9h))
	- 这里演示了冗余传输的过程，服务器对于收到的包，可以根据seq/ack情况动态去除冗余或者丢弃过期包。可能你会觉得全冗余是否不太合适并且有明显优化空间？在实际现网长期运行中，全冗余的冗余率是100%左右，相比于一些可靠传输的重发最近三帧等方式，这种为可靠性付出的代价是合适的并且也提高了更多实时性。 ([View Highlight](https://read.readwise.io/read/01ht5wq2mdg4ynhy53vd72srpg))