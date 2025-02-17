# 背景与总结
TODO 截图路径



# 一、1 提效利器：RAG为传统MIS系统大幅提效，门槛并不高
## 1.提效点
* 传统录入，需要一个文本框，一个文本框的添加信息 --> 转换成 语音输入，转换成文字，文字自动拆分成对应的文本框的结构化信息，入库。
* 查询，每一个筛选器需要进行录入信息，点击查询按钮 --> 转换成聊天文本框的方式，提问的方式，给出答案。

## 2.为什么要使用 Miniconda 或 Anaconda
相当于java的maven管理jar。python使用这两个工具管理各种工具包的下载和发布。
并且可以为python创建独立的虚拟环境。每一个虚拟环境可以独立一个python版本。
避免遇到依赖冲突、全局环境混乱、升级难题等问题。

Conda：作为 Anaconda 的一部分，Conda 也可以用来创建和管理虚拟环境，但它提供了更多的功能，如包管理和跨语言支持。

## 3.初识 CondaConda
* 也有分两个版本，一个是完整版本 Anaconda，一个是精简版本 Miniconda。
* Anaconda 是一个打包的集合，里面预装好了 conda、某个版本的 python、众多包、科学计算工具等等，就是把很多常用的、不常用的库都给你装好了。
* Miniconda 是 Anaconda 的精简版。Miniconda 只包含最基本的 Conda 包管理器和 Python 解释器。
由于 Miniconda 体积较小，它适合于那些只需要 Conda 包管理功能的用户。用户可以根据自己的需要，通过 Conda 安装所需的包和库。Miniconda 可以节省磁盘空间，并且启动速度更快。它提供了更多的灵活性来选择安装哪些包。

## 4.安装 Miniconda
参考截图。

## 5.虚拟环境搭建 & 系统运行
* 打开命令行环境 Anaconda Powershell Prompt
* 创建虚拟环境
conda create -n rag1 python=3.9
这里的 rag1 是虚拟环境的名称。
* 激活虚拟环境
conda activate rag1

# 二、2 对话模式：构建RAG应用的核心密码之一
## 1.System、user、assistant的关系
System：你是一个ERP MIS系统
user：客户A的款项到账了多少？
AI(assistant)：已到账款项为57980。

## 2.外部存储提供记忆功能 + RAG(全称是“Retrieval-Augmented Generation”，即“检索增强的生成”)
为prompt提供更多的上下文信息。

## 3.对话流程
* 检索阶段
根据用户提问内容，检索RAG信息，提供更多上下文propmt。
* 编码阶段

* 融合阶段
* 生成阶段
将用户问题 + 检索结果，一同形成prompt，请求大模型。



## 4.RAG组装后的一个例子
private String 从外部知识库检索到的知识 = "客户：A 入账日期：2024-07-06T00:00:00Z 入账金额：9527 已到账款项：57980 剩余到账款项：2908 "

private String 根据外部知识组装的语言 = f"""
您已经知道以下信息：
${从外部知识库检索到的知识},
请根据以上您所知道的信息回答用户的问题: 
"""
messages=[
      {"role": "user", "content": ${根据外部知识组装的语言} + "客户A的款项到账了多少？"},
      {"role": "assistant", "content": "已到账款项为57980。"},
      {"role": "user", "content": ${根据外部知识组装的语言} + "还剩多少？"},
      {"role": "assistant", "content": "剩余到账款项为2908元。"}
  ]
“根据外部知识组装的语言” 就是从外部知识库检索的过程。
messages则是将检索到的外部知识提供给大模型的代码。

# 三、3 返回结构化数据：构建RAG应用的核心密码之二
## 1.返回布尔值
messages=[
  {"role": "user", "content": f"""
  请以布尔值格式返回答案：老婆饼和老婆是不是同一类东西？  
  """},
  ]
## 2.返回整数
messages=[
  {"role": "user", "content": f"""
  请以整数格式返回正确选项：老婆饼是什么类型？
  
1. 点心
2. 水果
3. 菜肴
  """},
  ]
  
## 3.返回浮点数
messages=[
  {"role": "user", "content": f"""
  请以浮点数格式返回按米计算的答案：姚明有多高？  
  """},
  ]

## 4.返回数组
messages=[
  {"role": "user", "content": f"""
  请以数组的格式返回答案：请列出唐宋八大家的姓名  
  """},
  ]
  
## 5.返回json
messages=[
  {"role": "user", "content": f"""
  请以json的格式返回答案：我们需要查询A公司到账款项。
  
  json格式为：
  {
    'name':'A公司',
    'type':1,
  }
  其中type对应的选项和序号是：
  1. 到账款项
  2. 剩余款项
  请返回对应的序号。
  """},
  ]


返回
{
  'name':'A公司',
  'type':1,
 }  
 
当然如果大模型不够智能的话，我们需要分布，先返回“到账款项”，然后再将“到账款项”转换成数字（问大模型或者通过映射关系反向查询）

## 6.一劳永逸，希望大模型直接返回json。实现方式是提示大模型若干个例子，但实话实说，大模型不一定能100%的返回符合预期的结果
messages=[
  {"role": "user", "content": f"""
  请根据用户的输入返回json格式结果：
  
  示例1：
  用户：客户北京极客邦有限公司的款项到账了多少？
  系统：
  {{'模块':1,'客户名称':'北京极客邦有限公司'}}

  用户：{用户输入}
  系统：
  """},
  ]

# 四、4.动手实战：UI 实现和国产免费大模型接入
## 1.API的形式调用大模型

```
import json
import requests

def get_access_token():
  ernie_client_id = xxxx # 替换为你的client_id
  ernie_client_secret = xxxx # 替换为你的client_secret
  url = f"https://aip.baidubce.com/oauth/2.0/token?grant_type=client_credentials&client_id={ernie_client_id}&client_secret={ernie_client_secret}"
  
  playload= json.dumps("")
  headers = {
      'Content-Type': 'application/json',
      'Accept': 'application/json'
  }
  response = requests.request("POST", url, headers=headers, data=playload)
  return response.json().get("access_token")
```

```
def 对话模式(messages):
  url = "https://aip.baidubce.com/rpc/2.0/ai_custom/v1/wenxinworkshop/chat/ernie-lite-8k?access_token=" + get_access_token() //URL请求API，包含token信息
  
  json_obj = {
      "messages": messages,//待发送的prompt
  }

  playload= json.dumps(json_obj)
  headers = {
      'Content-Type': 'application/json'
  }
  
  response = requests.request("POST", url, headers=headers, data=playload) //调用大模型
  json_result = json.loads(response.text) //获取大模型返回值
  return json_result["result"]
```

# 五、5.动手实战：根据用户发问查询数据
## 1.将用户输入的信息转换成格式化的信息，方便方法调用时候获取 --- 即把用户的问题，转换成结构化的数据，方便后去根据结构化的数据去检索数据库
```
def 构造解析用户输入并返回结构化数据用的messages(用户输入):
if 之前的用户输入 is not None and len(之前的用户输入.strip()) > 0: 用户输入 = 之前的用户输入 + 用户输入

  messages=[
  {"role": "user", "content": f"""
  请根据用户的输入返回json格式结果，除此之外不要返回其他内容。注意，模块部分请按以下选项返回对应序号：
   1. 销售对账
   2. 报价单
   3. 销售订单
   4. 送货单
   5. 退货单
   6. 其他

  示例1：
  用户：客户北京极客邦有限公司的款项到账了多少？
  系统：
  {{'模块':1,'客户名称':'北京极客邦有限公司'}}

  示例2：
  用户：你好
  系统：
  {{'模块':6,'其他数据',None}}

  示例3：
  用户：最近一年你过得如何？
  系统：
  {{'模块':6,'其他数据',None}}

  用户：{用户输入}
  系统：
  """},
  ]
  return messages
```
注意事项：
* 提供非常多的示例，是让大模型更知道我的预期返回结果是什么。（如果调用结果还是不符合预期，可能是示例不够多）。
* 预期返回结果是json：比如 {{'模块':1,'客户名称':'北京极客邦有限公司'}}
* “除此之外不要返回其他内容”要求大模型返回的结果就是json，别自由发挥，添加额外信息。
* 第一行表示扩展用户输入，让上下文信息，填充到输入里，让大模型知道上下文。

## 2.如果返回结果不符合预期，就重试几次，因为每次结果都是很随机，会有成功的可能。

## 3.有了json返回结果，就可以做进一步的信息检索参数化提取了。
```
from .models import 销售入账记录

def 查询(查询参数):
    if '模块' in 查询参数:
        if 查询参数['模块'] == 1: #'销售对账'
            if '客户名称' in 查询参数:
                客户 = 查询参数['客户名称'].strip()
                return 销售入账记录.objects.filter(客户__icontains=客户)
```


# 六、6.动手实战：回答用户问题 -- 检索与拼接信息，再次向大模型发送请求
## 1.根据用户内容，返回查询结果，这里面涉及很多细节，在接下来会讲解

```
def index(request):
    if request.method == 'POST':
        用户输入 = request.POST['question'] //用户真实输入的内容

        查询参数 = 获取结构化数据查询参数(用户输入) //通过上节课讲解的内容，将用户输入，转换成json结构化内容
        
        查询结果 = None
        if 查询参数 is not None:
            查询结果 = 查询(查询参数) //说明查询参数是有内容的，所以根据查询参数去检索数据，返回检索后的数据（此时数据是json）

        if 查询结果 is None:
            从数据库查不到相关数据时的操作() //查询无结果被检索出来，所以输出默认值，找不到数据即可。
        else:
            根据查询结果回答用户输入(查询结果,用户输入) //根据返回的检索结果，和用户原始问题，进行大模型查询

    conversation_list = 对话记录.objects.filter(已结束=False).order_by('created_time')
    return render(request, "home/index.html",{"object_list":conversation_list})
```

## 2.根据查询结果回答用户输入(查询结果,用户输入) -- //根据返回的检索结果，和用户原始问题，进行大模型查询
```
def 根据查询结果回答用户输入(查询结果,用户输入):
  当前messages = 构造查询结果用的messages(查询结果,用户输入)  //本次的prompt
  之前的messages = 对话记录.objects.filter(已结束=False).order_by('created_time') //历史的上下文信息
  全部messages = 构造全部messages(之前的messages,当前messages)
  对话模式(全部messages)
```

## 3.构造查询结果用的messages(查询结果,用户输入) --- 构造本次prompt核心方法
```
def 构造查询结果用的messages(查询结果,用户输入):
  return [{"role": "user", "content": f"""
  您已经知道以下信息：

  {将查询结果转为字符串(查询结果)}

  请根据以上您所知道的信息回答用户的问题，注意，请简单和直接的回答，不要返回其他内容，不要提“根据您所提供的信息”之类的话。
：{用户输入}
  """}]
```

## 4.将查询结果转为字符串
比如json返回的数据库结果，转换成字符串，让大模型可以识别自然语言：

比如json变成字符串后：

```
客户：广州神机妙算有限公司
入账日期：2024-07-06T00:00:00Z
入账金额：9527 
已到账款项：57980 
剩余到账款项：2908
```

# 七、7.如何调试程序
## 1.记录好每一个步骤的prompt、以及返回结果、结构化参数信息，方便后期通过日志分析。






 
 
