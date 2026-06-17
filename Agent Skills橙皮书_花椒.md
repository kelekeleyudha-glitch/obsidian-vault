
## 本质
即给 agent 的操作手册
实际上是命名为 SKILL.md 的文本文件，Markdown格式，用自然语言写成 

### 两层结构

Frontmatter（YAML元信息：）
包含 名称 和 描述
其中 描述 需要包含skill的 任务目标 触发条件 当然还有其他的

Markdown（正文：执行流程）
先如何 后如何 输出格式 完成标准 附加约束


### 与 System Prompt 的区别

System Prompt是全局的、永久加载的  每一轮对话AI都会看到

Skill是模块化的、按需加载的


### 与MCP的关系

MCP是能力接口，Skill是知识和流程

很多时候一个好的Skill会同时用到MCP。比如飞书文档Skill，流程里就会调用飞书MCP提供的API。Skill负责决策（先做什么后做什么）​，MCP负责执行（调接口发请求）​



## 原理

### LLM的三种特性以及skill的对应策略


#### 1指令遵循

YAML的键值对结构清晰，Markdown的标题和列表层次分明。
SKILL.md的格式恰好是LLM最擅长理解的格式

#### 2上下文窗口与Skill加载

上下文窗口是LLM的「工作记忆」​ 窗口大小有限
所以Skill不能全部加载，必须按需加载

##### skill的调用：渐进式披露-三个层级

个人理解就是三层目录

预加载:只读元数据

Agent启动时,只读取每个Skill的Frontmatter,也就是name和description。一个Skill的元数据大约只有几十个token。27个Skill加起来也不过一两千token,几乎不占空间。这个阶段的目的是让系统知道「有哪些Skill可以用、什么时候该用」。


按需加载:完整SKILL.md正文 （建议500行以内）

当用户说了触发词,或者Agent判断当前任务需要某个Skill时,才把完整的SKILL.md正文加载到上下文里。比如你说「帮我审校」,审校Skill的全部300行才会被加载。


深度加载:Bundled Resources (脚本/引用文件/素材)

有些Skill会附带外部资源:scripts/目录下的脚本、references/目录下的参考文件、assets/目录下的素材 比如我的审校Skill引用了个人素材库的路径 只有在AI真正需要用到这些资源的时候,才会去读取


#### 3条件触发：Agent怎么知道该用哪个Skill（Conditional Triggering）


第一种：用户显式调用。你直接说/weekly-report或者/huashu-proofreading，Agent就加载对应的Skill。这是最明确的方式，零歧义

第二种：关键词匹配。基于description里写的触发词。你说「帮我写个周报」​，里面有「周报」这个词，命中了weekly-report Skill的description里声明的触发条件。这种匹配速度快，准确率高

第三种：语义匹配。你没说触发词，但你的意图和某个Skill的description语义接近。比如你说「这篇文章太生硬了」​，没有触发「审校」​「降AI味」这些关键词，但Agent通过理解你的意图，判断应该加载审校Skill。这种匹配更智能，但偶尔会判断错




### skill的执行模型


#### 执行流程：

触发识别-加载SKILL.md-理解指令-规划步骤-逐步执行-输出验证

示例很重要 示例帮AI建立「什么是好的输出」的标准


#### skill并非越详细越好

skill的最佳长度区间在500-2000，过多的说明反而会掩盖关键信息

把核心流程写清楚  加上几个关键示例  


#### skill触发

建议将skill之间的边界设立的非常清晰 
倘若 agent 同时触发多个 skill，轻则浪费token 重则skill冲突 或难以执行


#### skill冲突解决

设计不重叠的触发条件。让每个Skill在明确的场景下触发 「写文章」触发创作Skill，​「帮我改改」触发审校Skill。不要让同一个关键词同时触发两个Skill

在Skill里声明优先级。你可以在SKILL.md里写一条规则：​「如果正在执行其他写作任务，审校Skill不自动加载，等用户显式调用」​AI理解自然语言，这种约束它能遵守

把流程拆成阶段。先完成创作阶段，再进入审校阶段。在Skill的触发条件里加上阶段判断




## 生态

### 7个网站

1. Anthropic官方Skills仓库（github.com/anthropics/skills）
2. skills.sh（Vercel Labs推出）
3.  AgentSkill.sh
4. SkillsMP（skillsmp.com）
5. SkillHub（skillhub.club）
6. 腾讯SkillHub中国版
7. 字节跳动生态


![[Pasted image 20260617050054.png]]


### 评判skill的标准

触发条件清晰
执行步骤可验证
上下文占用合理
有用户确认环节
维护活跃


### 推荐一套课程

如果你想系统地入门Agent Skills，推荐DeepLearning.AI在2026年1月发布的「Agent Skills with Anthropic」课程。免费，2小时，由Anthropic的Elie Schoppik主讲。


