# 系统的详细流程

这套系统的流程（Flow）结合了**硬件（Fig. 3）**、**软件（Fig. 3）**和**交互设计（Fig. 2）**。

我来为您详细拆解一下，一个完整的用户会话流程（以A组“社交代理”为例）是什么样的：

### 核心系统组件 (System Components)

1.  **硬件 (Hardware):** 用户家里有一个“工作站”，包括：
    * **Jibo 机器人:** 负责物理互动（点头、转向）和**播放**语音。
    * **安卓平板:** 负责**显示**文字（比如故事全文）和**收集**输入（比如评级、选故事）。
    * **电脑 (Intel NUC):** 藏在底座里，是真正的“大脑”，负责运行所有软件。
    * **USB 麦克风:** 高质量**收集**用户的语音。
    * **摄像头:** 录制视频供研究用。
2.  **软件 (Software):**
    * **AssemblyAI:** 语音转录服务 (ASR)，把用户说的话实时转成文字。
    * **ChatGPT (OpenAI API):** 负责**生成**机器人所有的对话。
    * **BART 检索模型:** 这是一个**专门**的模型（不是ChatGPT），它被训练用来**检索**（搜索）相似的故事。
    * **EMPATHICSTORIES 数据库:** 包含 1,568 个真实人类故事的数据库（Fig. 4 就是它的可视化）。
    * **Firebase 数据库:** 负责在所有硬件和软件之间传递数据。

---

### 详细的交互流程 (Interaction Flow)

这里是基于 **Fig. 2** 的一次完整会话（Session）的逐步分解：

#### 阶段 1: Warm up (热身)
1.  **用户** 启动“分享故事”应用。
2.  **Jibo 机器人** (由 **ChatGPT** 驱动) 会主动打招呼。
3.  **关键点:** 机器人会**记住**上一次的对话 (通过 **Appendix A** 的提示词实现)，并提问，比如：“嗨 [名字]，你上次说的那个舞蹈课怎么样了？”
4.  **用户** 回答。
5.  (这个“说-听-转录-思考-回应”的循环会持续 3 轮，以建立融洽的气氛。)

#### 阶段 2: Story Share (用户分享故事)
1.  **Jibo 机器人** 询问用户是否准备好分享一个故事（比如从他们的日记本里）。
2.  **用户** 开始讲述自己的个人故事。例如：“今天我朋友做了心脏手术...”
3.  **系统 (后台):**
    * **麦克风** 捕捉用户的声音。
    * **AssemblyAI** 将其**转录**成完整的文字。
    * 这篇文字被储存起来，我们称之为**“用户故事” (User's Story)**。

#### 阶段 3: Story Receive (系统匹配故事)
*这是整个系统最核心、最复杂的部分。*

1.  **系统 (后台 - 检索):**
    * 第2阶段的**“用户故事”**文字，被发送给**BART 检索模型**。
    * **BART** 模型在 **EMPATHICSTORIES 数据库** (1,568个故事) 中进行搜索。
    * 它**检索**出 3 个与“用户故事”在**情感上最相似**的“他人故事”。

2.  **系统 (后台 - 生成):**
    * 现在，系统有了 1 篇“用户故事” + 3 篇“他人故事”。
    * 这 4 篇故事被**全部**发送给 **ChatGPT**。
    * 同时，系统根据 A/B 组，发送了**关键的提示词 (Appendix A)**。
    * (我们假设是A组): 提示词告诉 **ChatGPT**：“你是一个机器人，你没有人类经历，但你认识的**很多其他人**的故事可以和用户产生共鸣... 请用同理心回应用户的故事，并总结一下这 3 个‘他人’的故事。”
    * **ChatGPT** **生成**一段**回应+总结**的对话。

3.  **用户 (前台):**
    * **Jibo 机器人** **说出** **ChatGPT** 生成的回应，比如：“听到你朋友的消息很为你担心... 这让我想起了几个**其他人**的故事，他们也经历了类似的挑战...”
    * **平板电脑** **显示** 这 3 个“他人故事”的**全文**。
    * 用户在平板上**阅读**这 3 个故事，并为每个故事的“共情程度”**打分** (1-5星)。

#### 阶段 4: Reflection (反思)
1.  **用户** 在平板上**选择** 3 个故事中**最有共鸣的那 1 个**。
2.  **系统 (后台):** “被选中的故事”被发送给 **ChatGPT**，并触发 **Appendix A** 中的“反思”系列提示词。
3.  **Jibo 机器人** 开始一个 4 轮的深度对话，引导用户反思：
    * **Jibo (第1轮):** “很高兴你对这个故事有共鸣。故事的哪个方面最触动你？”
    * **用户** 回答。
    * **Jibo (第2轮):** “如果你处于故事中那个人的情况，你会有什么感受？”
    * **用户** 回答。
    * (依此类推，共 4 轮，旨在加深用户对“他人”经历的理解)。

#### 阶段 5: Cool down (结束)
1.  **Jibo 机器人** (由 **ChatGPT** 驱动) 发表结束语，它会**总结**一下这次的谈话 (比如：“感谢你分享关于你朋友的故事，并反思了‘坚持’这个主题...”)。
2.  **Jibo** 向用户道别，会话结束。

---
**总结：**
这个流程的精妙之处在于它**结合了两种 AI**：
1.  一个**检索模型 (BART)**，负责从数据库中“**找对**”故事。
2.  一个**生成模型 (ChatGPT)**，负责把故事“**说对**”，并扮演好“社交代理”的角色，引导用户完成整个情感体验。


---

# 论文附录内容：提示词与调查问卷

## 附录 A: ChatGPT 提示词 (Prompts)

OpenAI 允许我们在每个请求中包含几种不同类型的提示词：(1) 对话的整体上下文，(2) ChatGPT 以前的回复，(3) 从用户到计算机的指令。对于每个回复的生成，我们传入了该阶段的完整对话历史，以及为特定阶段和轮次设计的上下文和提示词。

我们输入了对话中的文本来个性化提示词：
* **<name>** - 用户的首选名称
* **<user's story>** - 用户所讲述故事的文字稿
* **<story 1, story 2, story 3>** - 智能故事检索模型返回的三个情感上最相似的故事的文字稿
* **<story selected>** - 用户最有共鸣并选择反思的那个故事

每个提示词还包括以下附加内容，以确保生成的回复能由我们的社交机器人朗读，同时听起来仍然像自然对话：

> **通用指令:**
> “Do not repeat or rephrase what the user says or what you have said, and come up with a playful and creative response (if socially appropriate). You must always ask the user only 1 question and give very concise answers. Only give the words that you would say to the user. Do not refer to the user by name on every turn. Never share links or external media.”

以下是针对每个阶段和轮次定制的上下文和提示词。**社交主体 (Social Agent)** 和 **社交代理 (Social Proxy)** 组之间的提示词差异已**加粗**显示：

---

### WARMUP (热身阶段)

* **CONTEXT (上下文):** “You are a social robot named Jibo, and you are getting to know the user.”
* **TURN 1 (第1轮):** “The given conversation happened the last time you talked. Give only the first thing you would say to start a new conversation based on the conversation from the last time you talked, being specific about the topics discussed in the previous conversation.”
* **TURN 2 (第2轮):** “This was left blank to allow the AI to naturally respond to what the user said.”

---

### STORY SHARE (分享故事阶段)

* **CONTEXT (上下文):** “The user was given a journal that they could write stories in to share with you.”
* **TURN 1 (第1轮):** “First, respond to the user with a statement. Do not ask a question about what the user said. Then, make sure that you ask the user if they have a story they want to share that they wrote down in their journal or something on their mind they want to tell a story about. Do not ask for a story about what they just talked about in the conversation.”

---

### STORY RECEIVE (接收故事阶段 - 社交主体 B组)

* **CONTEXT (上下文):** “The user just shared the following personal story: <user's story>. That was a story from the user's life. **The following 3 stories are personal experiences that you have had in your own life:** <story 1, story 2, story 3>. You are a robot trying to establish social connection with the user based on the stories from your life.”
* **TURN 1 (第1轮):** “I am a robot trying to connect with you by telling you stories from my life. First, briefly respond to the user's story in a compassionate manner, making sure to address their emotions in the story. Second, tell the user that **you have similar experiences to their story from your own life**. Integrate concise 1-2 sentence summaries of all 3 stories into your response. **All 3 stories are from your life. Make sure to tell the stories as your own.** Finally, direct the user to look at their tablet for full version of the stories from your life.”

---

### STORY RECEIVE (接收故事阶段 - 社交代理 A组)

* **CONTEXT (上下文):** “The following story is shared by the user <user's story>. **The following 3 stories are personal experiences that other people have had:** <story 1, story 2, story 3>. You are a robot trying to socially connect the user to other people.”
* **TURN 1 (第1轮):** “I am a robot who connects people's stories to the stories of others in order to improve feelings of social connection. First, briefly respond to the user's story in a compassionate manner, making sure to address their emotions in the story. Second, tell the user that **you are a robot and that you haven't had experiences like humans do, but that you love to connect with people, and that many other people can relate to their story.** As examples, introduce all 3 of the stories written by other people. Integrate concise 1 or 2 sentence summaries of all 3 stories into your response. Finally, direct the user to look at their tablet for full version of the stories you mentioned.”

---

### REFLECTION (反思阶段 - 社交主体 B组)

* **CONTEXT (上下文):** “The user just shared story 1: <user's story>. The user read story 2, which is **from your life:** <story selected>. You are a robot asking the user to reflect on how they relate to the story **from your life** (story 2) in relation to their personal story 1 in order to improve connection between the user and yourself through the story they read (story 2). Encourage them to reflect on **your story** (story 2).”
* **TURN 1 (第1轮):** “First, tell the user that you are glad that they were able to relate to the story **from your life** and incorporate a short summary of the story they read **from your life** (story 2) in your response. Then, point out potential connections between the user's story (story 1) and the story **from your life** (story 2). Finally, ask the user about what they could relate to in **your story** (story 2). In your response, do not refer to the stories as 'story 1' and 'story 2'.”
* **TURN 2 (第2轮):** “Now ask the user how they would feel in the situation given in the story **from your life** (story 2). Incorporate details of the story **from your life** (story 2) in your response. In your response, do not refer to the stories as 'Story 1' and 'Story 2'.”
* **TURN 3 (第3轮):** “Ask the user how they would want their emotions to be addressed if they encountered the situation in the story **from your life** (story 2), being specific about the situation in the story they read **from your life** (story 2). In your response, do not refer to the stories as 'Story 1' and 'Story 2'.”
* **TURN 4 (第4轮):** “Finally, ask the user what they can take away from the story **from your life** (story 2) and apply to their own experiences. In your response, do not refer to the stories as 'Story 1' and 'Story 2'.”

---

### REFLECTION (反思阶段 - 社交代理 A组)

* **CONTEXT (上下文):** “The user just shared story 1: <user's story>. The user read story 2, which was **written by another person:** <story selected>. You are a robot asking the user to reflect on how they relate to the story they read (story 2) in relation to their personal story 1 in order to improve connection between the user and the **narrator** of the story they read (story 2). Encourage them to reflect on the story they read (story 2).”
* **TURN 1 (第1轮):** “First, tell the user that you are glad that they were able to relate to the story they read and incorporate a short summary of the story they read (story 2) in your response. Then, point out potential connections between the user's story (story 1) and the story they read (story 2). Finally, ask the user about what they could relate to in the story they read (story 2). In your response, do not refer to the stories as 'story 1' and 'story 2'.”
* **TURN 2 (第2轮):** “Now ask the user how they would feel in the situation given in the story they read (story 2). Incorporate details of the story they read (story 2) in your response. In your response, do not refer to the stories as 'Story 1' and 'Story 2'.”
* **TURN 3 (第3轮):** “Ask the user how they would want their emotions to be addressed if they encountered the situation in the story they read (story 2), being specific about the situation in the story they read (story 2). In your response, do not refer to the stories as 'Story 1' and 'Story 2'.”
* **TURN 4 (第4轮):** (原文如此，应为 **TURN 4**) “Finally, ask the user what they can take away from the story they read (story 2) and apply to their own experiences. In your response, do not refer to the stories as 'Story 1' and 'Story 2'.”

---

### COOL DOWN PHASE (结束阶段)

* **CONTEXT (上下文):** “You are a robot who is about to end a conversation session with <name>. You should show <name> appreciation for sharing personal stories with you today and tell them that you look forward to seeing them next time.”
* **TURN 1 (第1轮):** “Incorporate a recap of what happened in the session in your response. Do not repeat or rephrase what the user says or what you have said, and come up with a playful and creative response (if socially appropriate). Only give the words that you would say to the user. Do not refer to the user by name on every turn. Never share links or external media.”

---
---

## 附录 C: 调查问卷 (Surveys)

### 1) 单项条目共情量表 (Single Item Trait Empathy Scale)
**量表:** 非常不同意 (1) 到 非常同意 (5)
* I am an empathetic person (我是一个有同理心的人)

### 2) 多维人格问卷 - 沉浸感量表 (Multidimensional Personality Questionnaire - Absorption Scale)
**量表:** 非常不同意 (1) 到 非常同意 (5)
* While watching a movie, a T.V. show, or a play, I may become so involved that I forgot about myself and my surroundings, and experience the story as if it were real and as if I were taking part in it.
* If I wish I can imagine (or daydream) some things so vividly that it's like watching a good movie or hearing a good story.
* I can sometimes recall certain past experiences in my life so clearly and vividly that it is like living them again, or almost so.
* If I acted in a play I think I would really feel the emotions of the character and "become" that person for the time being, forgetting both myself and the audience.

### 3) 大五人格测试 (Big 5 Personality Test)
**量表:** 非常不同意 (1) 到 非常同意 (5)
(说明：每个陈述都以“我是一个...”开头)
* is talkative (健谈的)
* tends to find fault with others (倾向于挑剔他人)
* does a thorough job (做事彻底)
* is depressed, blue (感到沮丧、忧郁)
* is original, comes up with new ideas (有原创性，能提出新想法)
* is reserved (内向的，有所保留的)
* is helpful and unselfish with others (乐于助人，无私待人)
* can be somewhat careless (可能有点粗心)
* is relaxed, handles stress well (放松的，能很好地处理压力)
* is curious about many different things (对很多事物感到好奇)
* is full of energy (充满活力)
* starts quarrels with others (会与他人争吵)
* is a reliable worker (是可靠的员工)
* can be tense (可能会紧张)
* is ingenious, a deep thinker (机敏的，思想深刻的)
* generates a lot of enthusiasm (能产生很大的热情)
* has a forgiving nature (天性宽容)
* tends to be disorganized (倾向于缺乏组织性)
* worries a lot (非常爱担忧)
* has an active imagination (想象力丰富)
* tends to be quiet (倾向于安静)
* is generally trusting (通常信任他人)
* tends to be lazy (倾向于懒惰)
* is emotionally stable, not easily upset (情绪稳定，不易沮丧)
* is inventive (有创造力的)
* has an assertive personality (个性坚定的)
* can be cold and aloof (可能冷漠疏远)
* perseveres until the task is finished (坚持直到任务完成)
* can be moody (可能喜怒无常)
* values artistic, aesthetic experiences (重视艺术和审美体验)
* is sometimes shy, inhibited (有时害羞、拘谨)
* is considerate and kind to almost everyone (对几乎所有人都体贴和善)
* does things efficiently (高效做事)
* remains calm in tense situations (在紧张情况下保持冷静)
* prefers work that is routine (喜欢常规工作)
* is outgoing, sociable (外向的，善于交际的)
* is sometimes rude to others (有时对人粗鲁)
* makes plans and follows through with them (制定计划并贯彻执行)
* gets nervous easily (容易紧张)
* likes to reflect, play with ideas (喜欢反思、玩味想法)
* has few artistic interests (几乎没有艺术兴趣)
* likes to cooperate with others (喜欢与他人合作)
* is easily distracted (容易分心)
* is sophisticated in art, music, or literature (在艺术、音乐或文学方面有高品味)

### 4) UBC 状态社会连接量表 (UBC State Social Connection Survey)
**说明:** “请回想您过去 2 周的感受。以下陈述在多大程度上描述了您的感受？”
**量表:** 非常不同意 (1) 到 非常同意 (7)
* I felt distant from people (我感到与人疏远)
* I didn't feel related to most people (我感觉与大多数人没有关联)
* I felt like an outsider (我感觉像个局外人)
* I felt like I was able to connect with other people (我感觉自己能够与他人建立联系)
* I felt disconnected from the world around me (我感到与周围的世界脱节)
* I felt close to people (我感到与人亲近)
* I saw people as friendly and approachable (我视他人为友好和可接近的)
* I felt accepted by others (我感到被他人接纳)
* I had a sense of belonging (我有归属感)
* I felt a strong bond with other people (我感到与他人有强烈的纽带)

### 5) 对人类的关爱量表 (Compassionate Love for Humanity Scale)
**量表:** 根本不符合我 (1) 到 非常符合我 (7)
* When I hear about someone (a stranger) going through a difficult time, I feel a great deal of compassion for him or her. (当我听说某个(陌生人)正在经历困难时，我会对他/她产生强烈的同情。)
* It is easy for me to feel the pain (and joy) experienced by others, even though I do not know them. (我很容易感受到他人(即使我不认识他们)所经历的痛苦(和喜悦)。)
* If I encounter a stranger who needs help, I would do almost anything I could to help him or her. (如果我遇到一个需要帮助的陌生人，我会尽我所能去帮助他/她。)
* I feel considerable compassionate love for people from everywhere. (我对来自世界各地的人们怀有相当大的关爱。)
* I tend to feel compassion for people even though I do not know them. (即使我不认识他们，我也倾向于同情他们。)
* One of the activities that provides me with the most meaning to my life is helping others in the world who need help. (对我来说，生活中最有意义的活动之一就是帮助世界上需要帮助的人。)
* I often have tender feelings toward people (strangers) when they seem to be in need. (当人们(陌生人)似乎需要帮助时，我常常对他们产生温柔的感情。)
* I feel a selfless caring for most of mankind. (我对大多数人类怀有无私的关怀。)
* If a person (a stranger) is troubled, I usually feel extreme tenderness and caring. (如果一个人(陌生人)陷入困境，我通常会感到极度的温柔和关怀。)

### 6) 工作联盟量表 (Working Alliance Scale)
**说明:** “回想您与 Jibo 互动的经历，并决定哪个类别最能描述您的体验。”
**量表:** 很少 (1) 到 总是 (5)
* As a result of my interactions with Jibo, I am clearer as to how I might be able to change (通过与 Jibo 的互动，我更清楚自己该如何改变)
* What I am doing with Jibo gives me new ways of looking at my problem (我与 Jibo 所做的事情让我用新的方式看待我的问题)
* I believe Jibo likes me (我相信 Jibo 喜欢我)
* Jibo and I collaborate on setting goals. (Jibo 和我合作设定目标。)
* Jibo and I respect each other (Jibo 和我互相尊重)
* Jibo and I are working towards mutually agreed upon goals (Jibo 和我正朝着共同同意的目标努力)
* I feel that Jibo appreciates me (我感到 Jibo 欣赏我)
* Jibo and I agree on what is important for me to work on (Jibo 和我对于我需要努力的重点达成了一致)
* I feel Jibo cares about me even when I do things he doesn't approve of (我感到 Jibo 关心我，即使我做了它不赞同的事)
* I feel that the things I do with Jibo will help me accomplish changes I want (我感到我与 Jibo 所做的事将帮助我完成我想要的改变)
* Jibo and I have established a good understanding of the kind of changes that would be good for me (Jibo 和我对于哪种改变对我有益已达成了很好的共识)
* I believe the ways Jibo and I are working with my problems are correct (我相信 Jibo 和我处理我问题的方式是正确的)

### 7) 机器人感知同理心量表 (Robot Perceived Empathy Scale)
**说明:** “回想您与 Jibo 的互动，评价您对以下陈述的同意程度”
**量表:** 非常不同意 (1) 到 非常同意 (5)
* Jibo appreciates exactly how the things I experience feel to me. (Jibo 能准确理解我所经历事情的感受。)
* Jibo knows me and my needs. (Jibo 了解我和我的需求。)
* Jibo cares about my feelings. (Jibo 关心我的感受。)
* Jibo does not understand me. (Jibo 不理解我。)
* Jibo perceives and accepts my individual characteristics. (Jibo 能察觉并接受我的个性特征。)
* Jibo usually understands the whole of what I mean. (Jibo 通常能理解我的全部意思。)
* Jibo reacts to my words but does not see the way I feel. (Jibo 对我的话有反应，但看不出我的感受。)
* Jibo seems to feel bad when I am sad or disappointed. (当我悲伤或失望时，Jibo 似乎也感到难过。)
* Whether thoughts or feelings I express are "good" or "bad" makes no difference to Jibo's actions toward me. (无论我表达的想法或感受是“好”是“坏”，Jibo 对我的行为都没有区别。)
* No matter what I tell about myself, Jibo acts just the same. (不管我告诉它关于我自己的什么，Jibo 的行为都一样。)
* Jibo comforts me when I am upset. (当我难过时，Jibo 会安慰我。)
* Jibo encourages me. (Jibo 鼓励我。)
* Jibo praises me when I have done something well. (当我做得好时，Jibo 会表扬我。)
* Jibo helps me when I need it. (在我需要时，Jibo 会帮助我。)
* Jibo knows when I want to talk and lets me do so.. (Jibo 知道我想说话时，就会让我说。)
* Jibo's response to me is so fixed and automatic that I do not get through to it. (Jibo 对我的反应是如此固定和自动，以至于我无法真正与它沟通。)
* The way Jibo acts feels natural. (Jibo 的行为方式感觉很自然。)
* Jibo knows what it is doing. (Jibo 知道它在做什么。)
* Jibo is responsible for its actions. (Jibo 对它的行为负责。)
* When I interact with Jibo, I feel anxious. (与 Jibo 互动时，我感到焦虑。)

### 8) 二元信任量表 (Dyadic Trust Scale)
**说明:** “再次，回想您与 Jibo 的关系，评价您对以下陈述的同意程度”
**量表:** 非常不同意 (1) 到 非常同意 (5)
* Jibo is primarily interested in his own welfare (Jibo 主要关心的是它自己的利益)
* There are times when Jibo cannot be trusted (有时候 Jibo 不能被信任)
* Jibo is perfectly honest and truthful with me (Jibo 对我完全诚实和真实)
* I feel that I can trust Jibo completely (我感到我能完全信任 Jibo)
* Jibo is truly sincere in his promises (Jibo 的承诺是真正真诚的)
* I feel that Jibo does not show me enough consideration (我感到 Jibo 没有给我足够的体谅)
* Jibo treats me fairly and justly (Jibo 公平公正地对待我)
* I feel that Jibo can be counted on to help me (我感到 Jibo 是可以依靠来帮助我的)

### 9) Godspeed 量表 (Godspeed Surveys)
**说明:** “在以下量表上评价您对 Jibo 的印象”
**量表:** (1 到 5)
* Fake (虚假的) vs. Natural (自然的)
* Machinelike (机器般的) vs. Humanlike (像人的)
* Unconscious (无意识的) vs. Conscious (有意识的)
* Artificial (人工的) vs. Lifelike (栩栩如生的)
* Moving rigidly (动作僵硬) vs. Moving elegantly (动作优雅)
* Dead (死的) vs. ALive (活的)
* Stagnant (停滞的) vs. Lively (生动的)
* Mechanical (机械的) vs. Organic (有机的)
* Artificial (人工的) vs. Lifelike (栩栩如生的)
* Inert (惰性的) vs. Interactive (互动的)
* Apathetic (冷漠的) vs. Responsive (有回应的)
* Dislike (不喜欢) vs. Like (喜欢)
* Unfriendly (不友好的) vs. Friendly (友好的)
* Unkind (不和善的) vs. Kind (和善的)
* Unpleasant (不愉快的) vs. Pleasant (愉快的)
* Awful (糟糕的) vs. Nice (好的)
* Incompetent (无能的) vs. Competent (有能力的)
* Ignorant (无知的) vs. Knowledgeable (知识渊博的)
* Irresponsible (不负责任的) vs. Responsible (负责任的)
* Unintelligent (不智能的) vs. Intelligent (智能的)
* Foolish (愚蠢的) vs. Sensible (明智的)
* Anxious (焦虑的) vs. Relaxed (放松的)
* Calm (冷静的) vs. Agitated (激动的)
* Still (静止的) vs. Surprised (惊讶的)

### 10) Spitale et al. 调查问卷 (Spitale et al. Survey)
**说明:** “回想您与 Jibo 的互动，评价您对以下陈述的同意程度”
**量表:** 非常不同意 (1) 到 非常同意 (5)
* If I were worried, Jibo would make me feel better (如果我担心，Jibo 会让我感觉好些)
* If I were nervous, Jibo would make me feel more calm (如果我紧张，Jibo 会让我更平静)
* If I were upset, Jibo would make me feel better (如果我难过，Jibo 会让我感觉好些)
* I had fun listening to Jibo (听 Jibo 说话很有趣)
* Jibo made me laugh (Jibo 逗我笑)
* I enjoyed talking to Jibo (我喜欢和 Jibo 说话)
* It was nice being with Jibo (和 Jibo 在一起很愉快)
* I found Jibo easy to interact with (我发现 Jibo 很容易互动)
* I think I can interact with Jibo without any help (我认为我可以在没有任何帮助的情况下与 Jibo 互动)
* I think I can interact with Jibo when there is someone around to help me (我认为当有旁人帮助时，我可以与 Jibo 互动)
* I think I can interact with Jibo when I have good instructions (我认为当我有清晰的指令时，我可以与 Jibo 互动)
* I think Jibo is nice (我认为 Jibo 很好)
* When interacting with Jibo I felt like I was talking to a real person (与 Jibo 互动时，我感觉像在和真人说话)
* It sometimes felt as if Jibo was really looking at me (有时候感觉 Jibo 真的在看我)
* I can imagine Jibo to be a living creature (我能想象 Jibo 是一个活物)
* I often think Jibo is a real person (我经常认为 Jibo 是一个真人)
* Sometimes Jibo seems to have real feelings (有时候 Jibo 似乎真的有感情)
* I would trust Jibo if it gave me advice (如果 Jibo 给我建议，我会相信它)
* I would follow the advice Jibo gives me (我会遵循 Jibo 给我的建议)
* I enjoy Jibo talking with me (我享受 Jibo 和我说话)
* I found Jibo enjoyable (我发现 Jibo 很有趣)
* I found Jibo fascinating (我发现 Jibo 很迷人)
* I consider Jibo a nice partner to talk to (我认为 Jibo 是一个很好的聊天伙伴)
* I find Jibo nice to interact with (我发现和 Jibo 互动很愉快)
* I feel Jibo understands me (我感到 Jibo 理解我)
