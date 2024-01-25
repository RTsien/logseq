title:: );             ( (highlights)
author:: [[rust-lang.org]]
full-title:: ");             ("
category:: #articles
url:: https://doc.rust-lang.org/src/alloc/rc.rs.html#342-350
![](https://readwise-assets.s3.amazonaws.com/static/images/article4.6bc1851654a0.png)

- Highlights first synced by [[Readwise]] [[Jan 4th, 2024]]
	- impl<T> Rc<T> { /// Constructs a new `Rc<T>`. /// /// # Examples /// /// ``` /// use std::rc::Rc; /// /// let five = Rc::new(5); /// ``` #[cfg(not(no_global_oom_handling))] #[stable(feature = "rust1", since = "1.0.0")] pub fn new(value: T) -> Rc<T> { // There is an implicit weak pointer owned by all the strong // pointers, which ensures that the weak destructor never frees // the allocation while the strong destructor is running, even // if the weak pointer is stored inside the strong one. unsafe { Self::from_inner( Box::leak(Box::new(RcBox { strong: Cell::new(1), weak: Cell::new(1), value })) .into(), ) } } ([View Highlight](https://read.readwise.io/read/01hjr375jhw2atpatgxd1df36d))
		- 💡: Rc::new实际是通过Box::leak实现的 #card