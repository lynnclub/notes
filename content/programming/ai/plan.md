---
title: "付费计划"
type: "docs"
weight: 4
---

## 按量付费

1. 之前一直用比较贵的Claude Sonnet 4.5 thinking，cursor额外账单$200/月，输入$3/M、输出$15/M，编码38.6、智能体51.7，吞吐适中  
Sonnet 4.5是编程能力好用的入门门槛

2. 为了控制cursor成本，改用Gemini 3 Flash，输入$0.5/M、输出$3/M，价格便宜5倍，编码42.6、智能体49.7稍弱，吞吐高，可以当编程主力  
GLM-5位置比较尴尬，比Flash强些、价格也贵些，没有必要，除非有套餐

3. Kimi K2.5性价比更高，输入$0.45/M、输出$2.2/M，编码39.5、智能体58.9，吞吐高，待体验

4. MiniMax M2.5使用排名第一、吞吐最高，输入$0.27/M、输出$0.95/M，编码37.4、智能体55.6，待体验

5. DeepSeek V3.2能力尚可但便宜，输入$0.26/M、输出$0.38/M，编码36.7、智能体52.9，缺点是吞吐低
Step 3.5 Flash价格比V3.2还便宜（目前免费），但是编码能力不足，可以当智能体

## 正版套餐

阿里云百炼coding plan  
最便宜，￥40/月（已停售），500次/5h，只支持国产模型，GLM-5用到爽，IDE支持lingma、vscode

copilot  
国外最便宜，免费套餐限制严  
付费$10/月，无限制使用GPT-5 mini、GPT-4o、GPT-4.1，IDE支持copilot、vscode，而且唯一凑齐openai、google、claude御三家，待体验  
https://github.com/features/copilot/plans#footnote-1

kiro  
新注册送500点，新老用户还送50点/月（包括Sonnet 4.5），付费套餐在高级模型上比copilot便宜些  
https://kiro.dev

cursor  
免费套餐模型比较差，付费套餐性价比低，主力改用kiro

gemini cli  
送免费额度，但需要科学上网，不能使用别家模型

codex  
openai推出的IDE（看不到代码，可以使用VSCode扩展替代）  
ChatGPT plus套餐$20/月，额度非常高，很难用完，如需使用非openai模型要单独付费并设置
