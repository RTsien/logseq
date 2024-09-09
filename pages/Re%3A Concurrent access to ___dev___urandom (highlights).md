title:: Re: Concurrent access to /dev/urandom (highlights)
author:: [[iu.edu]]
full-title:: "Re: Concurrent access to /dev/urandom"
category:: #articles
url:: https://lkml.iu.edu//hypermail/linux/kernel/0412.1/0181.html
summary:: The discussion addresses a bug in the Linux kernel related to concurrent access to /dev/urandom that could cause duplicate UUIDs. A patch is proposed to improve locking mechanisms to prevent multiple processors from generating the same random value simultaneously. Users are encouraged to test the patch and provide feedback on its effectiveness.
![](https://readwise-assets.s3.amazonaws.com/static/images/article3.5c705a01b476.png)

- Highlights first synced by [[Readwise]] [[Sep 10th, 2024]]
	- Concurrent access to /dev/urandom ([View Highlight](https://read.readwise.io/read/01j774ba21b5wdtzetz5jee6em))
	- SMP locking was added before 2.6.0 shipped (between 2.6.0-test7 and  
	  \-test8). But I see what happened; the problem is that the locking was  
	  added around add_entropy_words(), and not in the extract_entropy loop.  
	  Yes, extract_entropy() does call add_entropy_words() (which makes the  
	  fix more than just a two-line patch), but if two processors enter  
	  extract_entropy() at the same time, the locking turns out not to be  
	  sufficient. ([View Highlight](https://read.readwise.io/read/01j774b0kt5gc4ba0dejm5d8t1))
	- This patch solves a problem where simultaneous reads to /dev/urandom  
	  can cause two processes on different processors to get the same value.  
	  We're not using a spinlock around the random generation loop because  
	  this will be a huge hit to preempt latency. So instead we just use a  
	  mutex around random_read and urandom_read. Yeah, it's not as  
	  efficient in the case of contention, if an application is calling  
	  /dev/urandom a huge amount, it's there's something really misdesigned  
	  with it, and we don't want to optimize for stupid applications. ([View Highlight](https://read.readwise.io/read/01j774asdt16b92rdvjkj9w2s1))
	- @@ -1677,7 +1683,11 @@ urandom_read(struct file * file, char __  
	  flags |= EXTRACT_ENTROPY_SECONDARY;  
	  spin_unlock_irqrestore(&random_state->lock, cpuflags);  
	  
	  \- return extract_entropy(urandom_state, buf, nbytes, flags);  
	  + down(&random_read_sem);  
	  + n = extract_entropy(urandom_state, buf, nbytes, flags);  
	  + up(&random_read_sem);  
	  + return (n); ([View Highlight](https://read.readwise.io/read/01j774h6t593vsheg4p7hvwsg8))
		- 💡: down就是mutex.Lock，up就是mutex.Unlock