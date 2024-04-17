title:: Kubernetes Network Plugins (highlights)
author:: [[Yanick.Xia]]
full-title:: "Kubernetes Network Plugins"
category:: #articles
url:: https://blog.yanick.site/2022/03/01/kubernetes/cni/
![](https://readwise-assets.s3.amazonaws.com/media/uploaded_book_covers/profile_1160846/20220301172159.png)

- Highlights first synced by [[Readwise]] [[Apr 13th, 2024]]
	- 对于 `k8s` 本身来说，我们只是需要调用 插件 的 `addToNetwork`，而这层的封装是单独放置于 [containernetworking-cni](https://github.com/containernetworking/cni/blob/master/libcni/api.go) 之中的。在分析之前，我们在最初的 `k8s` 的文档中有涉及到，我们使用 `CNI` 至少需要
	  
	  •   bin: CNI 的二进制文件
	  •   conf: CNI 的配置文件
	  
	  而这里的 `plugin` 在逻辑上也只是 `bin` 的一层封装。
	  
	  addNetwork[github](https://github.com/containernetworking/cni/blob/96a18838186d36fe2efef4cc943902c1e8f06879/libcni/api.go#L393-L415)
	  
	  1  
	  2  
	  3  
	  4  
	  5  
	  6  
	  7  
	  8  
	  
	  func (c *CNIConfig) addNetwork(ctx context.Context, name, cniVersion string, net *NetworkConfig, prevResult types.Result, rt *RuntimeConf) (types.Result, error) {  
	  c.ensureExec()  
	  pluginPath, err := c.exec.FindInPath(net.Network.Type, c.Path)  
	  if err != nil {  
	  return nil, err  
	  }  
	  return invoke.ExecPluginWithResult(ctx, pluginPath, newConf.Bytes, c.args("ADD", rt), c.exec)  
	  }  
	  
	  而本质上，当系统创建了一个新 `POD` 的时候，`k8s` 就会通过 `Exec` 去执行这个 `Bin` 文件，将上下文对象作为参数传入 ([View Highlight](https://read.readwise.io/read/01hv98nrygb6c2d5btrc14p9yf))