title:: 面向开发过程的天刀服务器引擎之：多类型匹配服务框架 (highlights)
author:: [[woa.com]]
full-title:: "面向开发过程的天刀服务器引擎之：多类型匹配服务框架"
category:: #articles
url:: https://km.woa.com/articles/show/423657?kmref=search&from_page=1&no=2
summary:: The text discusses the challenges and solutions in developing a diverse matchmaking system for a mobile game, with multiple match types and pools to consider. It outlines the architecture, including processes like proxy, matchsvr, and judge, to handle the complexities of player data and matching across different pools effectively. The framework allows for standardized development of new matchmaking types by extending key interfaces, streamlining the process and improving efficiency in ongoing game content production.
![](https://readwise-assets.s3.amazonaws.com/static/images/article0.00998d930354.png)

- Highlights first synced by [[Readwise]] [[Apr 15th, 2024]]
	- 天刀手游匹配系统的特殊性在于，有多种匹配类型（match type），截止到目前有9种（还有大量需求正在酝酿中），分别是：战场队伍匹配，战场对手匹配，活动匹配，快速游戏，论剑匹配，40v40战场匹配，吃鸡匹配，锦标赛匹配（世界杯），话本匹配。 ([View Highlight](https://read.readwise.io/read/01hvg94c11yf7rn4ht8b9vcx2n))
	- 对于每种匹配类型（match type），还会有不同的匹配池（match id），例如，对话本匹配来说，每个话本需要是个独立的匹配池，对活动匹配来说，每个活动也需要是个独立的匹配池。目前天刀手游的规划中，话本匹配大约有10个，活动匹配和快速游戏大约各有30个，并且还在增长中。 ([View Highlight](https://read.readwise.io/read/01hvg94naq054c7ztqrcf3vhah))
	- mmorpg的匹配系统设计，要充分考虑到玩家的各种数据的平衡，例如，除了战力值，等级，段位，elo等常见的匹配指标外，还要考虑玩家的职业配比，职责搭配（T，N，DPS），男女搭配，装备情况，帮派情况，社区情况，group情况等 ([View Highlight](https://read.readwise.io/read/01hvg981c0ts5hf4qdkem33ft7))
	- 如果玩家同一时间只能加入一个匹配池，则整体的负载压力不大，也不需要做分布式状态协同，这是端游时代的情况。到了手游时代，由于屏幕变小，玩法变多，社交元素增加，玩家变得非常依赖于匹配系统，所以策划参考了业界同类游戏的做法后，希望玩家能同时加入所有类型的匹配系统的所有匹配池，然后匹配上哪个玩家就直接玩哪个，这样体验最好（这是项目的【真实需求】，所以即使比较复杂，我们肯定也要实现，相对的，后面讨论中会遇到一些【伪需求】）。 ([View Highlight](https://read.readwise.io/read/01hvg9ahgmf7y7fas2qw7hdkbr))
		- 💡: red会有这种场景吗？❓
	- 比较特殊的是judge进程，它的作用是，协调同一玩家在不同匹配池的匹配状态，保证结果的唯一性。因为如前文所说，同一个玩家，可以同时加入多个匹配类型和匹配池，但是，最终匹配成功的只能有一个，否则，就会出现玩家同时加入了两个匹配队伍的情况，此时玩家只能二选一，那就会导致有一个队伍少了一个人（如下图所示）。 ([View Highlight](https://read.readwise.io/read/01hvg9qcs0vyerm32rvvv2v40t))
		- 💡: 这个比red考虑的更复杂。red是通过statesvr来做状态互斥，确保不会同时在两个类型里匹配成功❓
	- 实际的匹配是一个分布式的状态机 ([View Highlight](https://read.readwise.io/read/01hvgafwbfhb939csffmpq8tzj))
	- ![](https://km.woa.com/asset/d57660349fe94d63a37d26eb430fd490?height=578&width=918) ([View Highlight](https://read.readwise.io/read/01hvgafy36py2rap530387ax19))
	- ![](https://km.woa.com/asset/634a6a4be5b345a885ec174b3fbf71c8?height=493&width=883) ([View Highlight](https://read.readwise.io/read/01hvga9gz34jaw3yb255ey73gx))
		- 💡: 乐观实现
	- 考虑到我们使用分布式匹配的方式来承载，那么匹配算法的选择，重点就不是考虑单进程的性能，而是考虑分布式的简单程度。这里的算法，不是指具体的匹配规则，而是匹配的对象选择遍历策略。
	  
	    像匈牙利算法，km算法这种找出集合中全局最佳匹配子集的算法，需要递归和数据的反复访问，所以分布式实现的难度很高，而且对mmorpg游戏业务来说，项目的【真实需求】并不是找到这种【最优的匹配子集】，而是【在较短时间内为玩家找到满足条件的匹配对象】。所以，针对我们的架构，我们选择了线性遍历的方式，并且，我们不追求最佳匹配对象，只找出合格匹配对象，所以在高负载的情况下可以把大部分的匹配开销控制在单个匹配池之内，降低整体负载。流程如下： ([View Highlight](https://read.readwise.io/read/01hvgawk63f65s5j1gvmra15zv))
		- 💡: 匹配算法的具体实现
	- ![](https://km.woa.com/asset/ea2d83364e1b4755822a144d732be312?height=375&width=611)
	  
	    如图中所示，由于match_obj数据是存在matchsvr本地内存中的（因为匹配是个对性能要求较高的强状态模块），所以如果本匹配池未找到全部匹配对象，需要转移到另外一个匹配池继续寻找。由于我们采用的是线性遍历，所以前一个匹配池的中间结果（例如需要匹配40人的团队，第一个匹配池凑够了10个人）可以发给第二个匹配池接着匹配。需要注意的是match obj的数据，如果我们直接在协议中发送这个数据，会导致通讯数据量很大，然后match proxy的负载也会变得很高，所以我们把数据存在tcaplus里，其它匹配池接到远程匹配请求之后从tcaplus里拉取数据进行匹配（对应前面状态图中的sync状态）。 ([View Highlight](https://read.readwise.io/read/01hvgb83kzxvw8rvf8d7xvxzzv))
	- ![](https://km.woa.com/asset/6c377d16f9ad41d99b3358da332d6d47?height=937&width=1071)
	  
	    大部分新增匹配类型，只需要继承并实现图中的match_pool，match_obj，match_result的接口即可 ([View Highlight](https://read.readwise.io/read/01hvgbbd4gg7qnxke6w5s64c4c))
	- class MATCH_POOL : public MX_OBJECT { public: virtual MATCH_RESULT* rlt() = 0; virtual int match_target_num() = 0; virtual int match_type() = 0; //<0表示不需要interval match virtual int match_interval_sec() { return DEFAULT_MATCH_INTERVAL_SEC; } virtual void init(); virtual int add_obj(MATCH_OBJ &obj); virtual int del_obj(MATCH_OBJ &obj, int reason); virtual int num(); virtual void clear_timeout_obj(); virtual void timing_match_routine(); virtual MATCH_RESULT* obj_match(MATCH_OBJ &obj); virtual bool can_obj_match(MATCH_OBJ &obj_1, MATCH_OBJ &obj_2); virtual int match_result_proc(); public: void set_obj_timeout_sec(int obj_timeout_sec) { _obj_timeout_sec = obj_timeout_sec;} mx_list& list() { return _list; } int check_and_del_exist(MATCH_OBJ &obj); private: int _match_result_succ_proc(); int _match_result_fail_proc(); int _match_result_error_proc(); protected: mx_list _list; int _obj_timeout_sec; }; ([View Highlight](https://read.readwise.io/read/01hvgc2vg2b08nzbvr705xhdcr))
		- 💡: 核心匹配算法是在can_obj_match实现的
	- class MATCH_RESULT { public: //这些接口和add_match_obj的区别在于，某些匹配类型，result需要把 //obj中的多个rid（例如bf team匹配）merge到一个result里 virtual int add_rid(u64 obj_rid) = 0; virtual int add_rid(MATCH_OBJ &obj) = 0; virtual u64 rid(int idx) const = 0; virtual int rid_num() const = 0; virtual int match_type() const = 0; virtual void clear(); ([View Highlight](https://read.readwise.io/read/01hvgbza1b4tfe9eg8h61fed0c))
		- 💡: 这里的rid是指什么❓