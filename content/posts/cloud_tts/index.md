---
title : "利用Azure TTS服务进行小说朗读" 
date : "2024-05-06T17:36:56+08:00" 
lastmod : "2024-05-06T17:36:56+08:00" 
tags : ["tts","python","azure"] 
categories : ["技术"]
draft : flase
featuredImage : /images/posts/cloud_tts/featuredImage.jpg
summary : '借助GPT封装云厂商TTS服务，手机APP实现自定义朗读功能'
---

## 需求

iOS系统自带语音效果不佳，最近接触到Azure TTS服务效果良好。想在读书软件香色闺阁中加入Azure tts音色。

## 方案

在服务端对Azure tts服务进行封装，对外提供接口。香色闺阁中调用接口，实现朗读。

### Azure tts

1. 先注册Azure账号，Azure AI services|语音服务中创建项目，获取秘钥和区域。密钥和区域写入环境变量中，方便使用。
2. 安装python SDK，pip install azure-cognitiveservices-speech
3. 参考[官方文档](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/get-started-text-to-speech?tabs=linux%2Cterminal&pivots=programming-language-python)
4. 获取音色配置文件，完成文本转语音功能。

#### 音色列表

- 获取音色列表形成音色配置文件config.ini。

```python
import requests
import json
import configparser
import os
# URL地址
url = 'https://japanwest.tts.speech.microsoft.com/cognitiveservices/voices/list'

# 定义headers
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.3',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
    'Accept-Encoding': 'gzip, deflate, sdch',
    'Accept-Language': 'en-US,en;q=0.8,zh-CN;q=0.6,zh;q=0.4',
    'Ocp-Apim-Subscription-Key':os.environ.get('SPEECH_KEY'),
    # 其他您需要的headers
}

# 发送GET请求，包含headers
response = requests.get(url, headers=headers)

# 检查请求是否成功
if response.status_code == 200:
    config = configparser.ConfigParser()
    config.read('config.ini')
    config['voice']={}
    # 输出返回的数据
    data = json.loads(response.text)
    for voice in data:
        if voice['Locale'] == 'zh-CN' or voice['Locale'] == 'zh-TW':
            config['voice'][voice['DisplayName']] =voice['ShortName']
    with open('config.ini', 'w',encoding='utf-8') as configfile:
        config.write(configfile)
else:
    # 输出错误信息
    print('Error:', response.status_code)
```

#### 文本转语音

- config.ini文件与下面代码放在同一目录下。

```python
import azure.cognitiveservices.speech as speechsdk
import os
import configparser

class tts():
    def __init__(self):
        self.config = configparser.ConfigParser()
        self.config.read('config.ini')
        
    def speak(self, voice_id,text):
        path = '/home/ubuntu/tts/output.wav'
        speech_config = speechsdk.SpeechConfig(subscription=os.environ.get('SPEECH_KEY'), region="japanwest")  
        file_config = speechsdk.audio.AudioOutputConfig(filename=path)
        speech_config.speech_synthesis_voice_name= self.config['voices'][voice_id]
        speech_synthesizer = speechsdk.SpeechSynthesizer(speech_config=speech_config, audio_config=file_config)  
        speech_synthesis_result = speech_synthesizer.speak_text_async(text).get()
        return path,speech_synthesis_result

if __name__ == "__main__":
    text_to_tts = tts()
    text_to_tts.speak("yunxia","2019/03/02上午有1/2的概率下暴雨所以有600人選擇3:30p.m.再出門,支付$500或￥600可以獲得代金券")
```

#### 对外接口

- 基于fastapi编写对外服务接口。

```python
from fastapi import FastAPI,Query
from fastapi.responses import FileResponse
from pydantic import BaseModel
from speech_synthesis import tts
import os
import uvicorn

# class TTSRequest(BaseModel):
#     voice_id: str  # 假设voice_id是一个整数
#     text: str      # text是一个字符串

app = FastAPI()

# @app.get("/tts/")
# async def handle_tts(request: TTSRequest):
#     text_to_speech = tts()
#     audio_file_path,speech_synthesis_result = text_to_speech.speak(request.voice_id, request.text)

#     if os.path.isfile(audio_file_path):
#         # 使用 FileResponse 返回文件内容
#         return FileResponse(audio_file_path, filename="output.wav", media_type="audio/wav")
#     else:
#         print("audio_file_path: " + audio_file_path)
#         print(speech_synthesis_result)
#         return {"error": "File not found"}, 404

@app.get("/tts/")
async def handle_tts(voice_id: str = Query(None, description="The ID of the voice to use"),
                     text: str = Query(None, description="The text to be synthesized")):
    text_to_speech = tts()
    audio_file_path,speech_synthesis_result = text_to_speech.speak(voice_id, text)

    if os.path.isfile(audio_file_path):
        # 使用 FileResponse 返回文件内容
        return FileResponse(audio_file_path, filename="output.wav", media_type="audio/wav")
    else:
        print("audio_file_path: " + audio_file_path)
        print(speech_synthesis_result)
        return {"error": "File not found"}, 404

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

#### 部署

1. 服务器用的ubuntu，将上面python代码和config.ini放到/home/ubuntu/tts中
2. python执行代码提供服务
3. 配置域名，将域名指向服务器

### 编写香色tts源

- 可以从香色中导出一个语音源，使用下列代码转换成json格式。修改json中内容，然后再转换成xbs在香色中导入即可。

```python
import xxtea
import struct
import os
import json


xxtea_key = bytes([0xe5, 0x87, 0xbc, 0xe8, 0xa4, 0x86, 0xe6, 0xbb, 0xbf, 0xe9, 0x87, 0x91, 0xe6, 0xba, 0xa1, 0xe5])

def xbs2json(buffer):

    try:
        out = xxtea.decrypt(buffer, xxtea_key,False, 0)
    except Exception as e:
        return None, str(e)
    
    n = len(buffer)
    n -= 4
    m = struct.unpack('<I', out[n:])[0]
    
    if m < n - 3 or m > n:
        return None, "decode error"
    
    n = m
    return out[:n], None

def json2xbs(buffer):
    buffer_len = len(buffer)
    n = buffer_len // 4
    if buffer_len % 4 != 0:
        n += 1
    
    # Pad the buffer to the next multiple of 4
    padding_length = n * 4 - buffer_len
    buffer_enc_len = b'\x00' * padding_length
    
    # Append the original buffer length as a little-endian uint32
    buffer_enc_len += struct.pack('<I', buffer_len)
    
    # Combine the original buffer with the padding and length
    buffer = buffer + buffer_enc_len
    
    try:
        # Encrypt the buffer using XXTEA
        out = xxtea.encrypt(buffer, xxtea_key,False, 0)
        return out, None
    except Exception as e:
        return None, str(e)

def load_file(filepath):
    if os.path.exists(filepath):
        with open(filepath, 'rb') as file:
            return file.read(),None
    else:
        return None, f"File {filepath} does not exist."

def xbs_to_json(input,output):
    data, error = load_file(input)
    if error:
        print(error)
        exit(1)
    encode_data,error = xbs2json(data)
    if error:
        print(error)
        exit(1)
    # 打开文件以二进制模式写入
    with open(output, 'w', encoding='utf-8') as f:
        json_data = json.loads(encode_data.decode('utf-8'))
        json.dump(json_data, f, ensure_ascii=False, indent=4)
def json_to_xbs(input,output):
    data, error = load_file(input)
    if error:
        print(error)
        exit(1)
    decode_data,error = json2xbs(data)
    if error:
        print(error)
        exit(1)
    with open(output, 'wb') as f:
        f.write(decode_data)

if __name__ == '__main__':
    xbs_to_json('D:\\workspace\\script\\xs\\test\\azure_tts.xbs' , 'D:\\workspace\\script\\xs\\test\\azure_tts1.json')
    json_to_xbs('D:\\workspace\\script\\xs\\test\\azure_tts1.json' , 'D:\\workspace\\script\\xs\\test\\azure_tts1.xbs')
```

- 修改点为requestInfo中url调整为自己服务的域名，body只保留voice_id参数。
- requestFilters调整为Azure TTS音色信息，与上面config.ini中id对应。

```plaintext
{
    "语音设置-azure": {
        "chapterContent": {
            "actionID": "chapterContent",
            "parserID": "DOM"
        },
        "enable": 1,
        "shupingList": {
            "actionID": "shupingList",
            "parserID": "DOM"
        },
        "authorId": "",
        "bookDetail": {
            "actionID": "bookDetail",
            "parserID": "DOM"
        },
        "bookWorld": {
            "azure": {
                "actionID": "bookWorld",
                "validConfig": "",
                "requestInfo": "@js:\n\nlet url=\"https://tts.xiaoshame1.pp.ua/tts/\";\nlet body={\n    voice_id:params.filters.type,\n};\nlet _conf={\n      key:'text',\n      body:body,\n      headers:{},\n      post:false,\n      url:url\n };\n\nparams.nativeTool.setCache(\"xsreader_voice_conf\",JSON.stringify(_conf));\nreturn config.host",
                "bookName": "@js:\nlet regt=`\\n\\s*(.*?)::${params.filters.type}\\n`\nlet reg=new RegExp(regt)\nlet name=config.moreKeys.requestFilters.match(reg)[1]\nreturn \"当前语音：\"+name",
                "detailUrl": "@js:\nreturn config.host",
                "host": "http://captive.apple.com",
                "_sIndex": 7,
                "list": "//title",
                "responseFormatType": "html",
                "parserID": "DOM",
                "moreKeys": {
                    "pageSize": 30,
                    "requestFilters": "type\n晓晓::Xiaoxiao\n云希::Yunxi\n云健::Yunjian\n晓伊::Xiaoyi\n云扬::Yunyang\n晓辰::Xiaochen\n晓涵::Xiaohan\n晓梦::Xiaomeng\n晓墨::Xiaomo\n晓秋::Xiaoqiu\n晓睿::Xiaorui\n晓双::Xiaoshuang\n晓晓 方言::Xiaoxiao Dialects\n晓晓 多语言::Xiaoxiao Multilingual\n晓颜::Xiaoyan\n晓悠::Xiaoyou\n晓甄::Xiaozhen\n云枫::Yunfeng\n云皓::Yunhao\n云夏::Yunxia\n云野::Yunye\n云泽::Yunze\n晓萱::Xiaoxuan\n曉臻::HsiaoChen\n雲哲::YunJhe\n曉雨::HsiaoYu"
                }
            }
        },
        "shudanList": {},
        "sourceType": "text",
        "relatedWord": {
            "actionID": "relatedWord",
            "parserID": "DOM"
        },
        "weight": "9999",
        "sourceName": "语音设置-azure",
        "sourceUrl": "http://captive.apple.com",
        "miniAppVersion": "2.53.2",
        "shudanDetail": {
            "actionID": "shudanDetail",
            "parserID": "DOM"
        },
        "lastModifyTime": "1695541859.065246",
        "shupingHome": {
            "actionID": "shupingHome",
            "parserID": "DOM"
        },
        "searchShudan": {
            "actionID": "searchShudan",
            "parserID": "DOM"
        },
        "searchBook": {
            "actionID": "searchBook",
            "parserID": "DOM"
        },
        "chapterList": {
            "actionID": "chapterList",
            "parserID": "DOM"
        }
    }
}
```

### 使用

1. 发现中切换到导入的音色源，选择音色
2. 文章点击朗读，发音中选择语音测试

## 总结

1. 此方案同样可以封装阿里云和百度云的TTS服务，只需要增加相应云服务API调用代码。
2. 代码比较简单，云API调用按照官方文档的步骤来，fastapi相关代码借助GPT完成。
3. 查看他人香色语音源可以比较快熟悉的编写规则，转换成json后编写效率更高。
4. 相关代码可以在[Github](https://github.com/xiaoshame/script/tree/main/xs)上查看。
