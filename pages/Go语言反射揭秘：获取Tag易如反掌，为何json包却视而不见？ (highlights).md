title:: Go语言反射揭秘：获取Tag易如反掌，为何json包却视而不见？ (highlights)
author:: [[磊丰]]
full-title:: "Go语言反射揭秘：获取Tag易如反掌，为何json包却视而不见？"
category:: #articles
url:: https://mp.weixin.qq.com/s/pCx8d33i1cMVsJaCH1TbJA
summary:: The article discusses why the Go JSON package ignores private fields in structs, despite the ability to access them through reflection. It highlights three key design principles of the JSON package: safety, encapsulation, and predictable behavior. To export private fields when serializing, it suggests custom solutions like defining a custom `MarshalJSON` method or using third-party libraries.
![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/HvRgjyfjzzZR8WBrqCb0PaU8MMXS4ibBmUcnhcPf6R9ZkCU3HvPnwTt3ALFwAjORMHhOk5icsCicvpicFwgk0Nic1KA/0?wx_fmt=jpeg)

- Highlights first synced by [[Readwise]] [[Jun 19th, 2025]]
	- 反射三大能力：
	  
	  1.  **穿透私有屏障**：`reflect.Type`可访问非导出字段
	  2.  **Tag自由读取**：`Field.Tag.Get()`无视可见性
	  3.  **元数据解析**：支持任意自定义Tag（如`db:"user_id"`）
	  
	  > 💡 **反射本质**：Go的`reflect`包是编译器级别的"上帝视角"，不受导出规则限制 ([View Highlight](https://read.readwise.io/read/01jy1yvde3jy9a0sa0t7njrqd4))
	- import"github.com/mitchellh/mapstructure"  
	  
	  funcmain(){  
	    u := User{1,"Tom","tom@example.com"}  
	  
	  // 将结构体转为map（包含私有字段）  
	  var result map[string]interface{}  
	    mapstructure.Decode(u,&result)  
	  
	    data,_:= json.Marshal(result)  
	    fmt.Println(string(data))// {"email":"tom@example.com", "id":1, "name":"Tom"}  
	  } ([View Highlight](https://read.readwise.io/read/01jy1ywmn0bnz1zx4nwz2zkvmj))
		- 💡: mapstructure可以导出struct的私有字段