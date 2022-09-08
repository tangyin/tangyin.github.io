# UE4插件使用
 ### 功能端口

| 功能 | 端口号 | 
| - | - | 
| GRPC调用 | **50051** |
| 动捕数据接收 | **5558** | 
| 音频数据接收 | **6558** | 
| 从UE4接收音视频流 | **8124** | 

***
### GRPC调用
**当前的grpc的协议**
```C++
syntax = "proto3";

package ue4_interactive;

service UE4Interactive {
  //用于执行各种UE事件调用
  rpc UE4Event (UE4EventRequest) returns (UE4EventReply) {}
  //用于心跳检测UE主线程是否卡死
  rpc UE4HeartBeat (UE4HeartBeatRequest) returns (UE4HeartBeatReply) {}
}

message UE4EventRequest {
    //用于标识响应事件的人，与UE中配置的人物ID相对应，暂时仅action_jugong与action_shenshou有效
  string model_name = 1;
  //标识事件的类型
  string event_type = 2;
  string data = 3;
}
message UE4EventReply {
  float event_duration_in_seconds = 1;
}

message UE4HeartBeatRequest {
}
message UE4HeartBeatReply {
//返回UE主线程上次更新的本地时间（从1970年1月1日到现在的秒数）
  int64 timestamp = 1;
}

```
#### UE4EventRequest
UE4EventRequest 的 event_type现在支持的类型有：

| 类型 | 功能 |返回值 | data 数据示例|
| - | - | - | - |
| input_key | 按键控制 |  **返回 0** | {"keys":["Num 0"，"K"]} |
| console_command | 执行UE的控制台命令 |  **返回 0** | {"command":["r.SetRes 960x1080w",  "t.MaxFPS 60"]} |
| widget_text | 显示文本控件，执行人物动作 |  **返回 0** | {"color":"R=1,G=0,B=0","fontsize":"50","x-location":"0.5","y-location":"0.5","width":"0.9","height":"0.9","text": "hello world", "action": "jump"} |
| widget_pic  |显示图片控件，执行人物动作 | **返回 0** | {"x-location":"0.5","y-location":"0.5","width":"0.9","height":"0.9","action":"dead","pic":"http://1.com/1.jpg"} |
| widget_pic_and_text  |显示文本控件，执行人物动作 | **返回 0** | {"x-location":"0.5","y-location":"0.5","width":"0.9","height":"0.9","action":"dead","text": "hello world", "pic":"http://1.com/1.jpg"} |
| widget_video_and_text  |显示文本控件，执行人物动作 | **返回 0** | {"text": "hello world", "action":"idle","rate":2.0,"videos":[{"thumbnail":"http://1.com/1.jpg", "video_resource":"https://1.com/1.mp4"},{"thumbnail":"http://2.com/2.jpg", "video_resource":"https://2.com/2.mp4"}]} |
| widget_button |显示文本控件，执行人物动作 |  **返回 0** | {"text":"hello world","action":"idle3","user_id":"aaaa","session_id":"bbbb","buttons":[{"button":"buttonText1", "callback_info":"123"},{"button":"buttonText2", "callback_info":"1123"}]} |
| widget_reset  | 隐藏widget事件的控件 | **返回 0** |{}
| reset  | 隐藏widget事件的控件，人物返回默认待机动作 | **返回 0** |{}
| action | 执行人物动作|  **返回 0** | {"action":"attack", "rate":"1.0", "follow":"false"} ,切换姿态：{"pose":"stand"}, 刷新脸部表情循环 {"refresh_face":"true"} |
| expression | 执行人物微表情|  **返回 0** | {"expression":"expression1", "rate":"1.0"} |
| action_and_expression | 执行人物带表情动作|  **返回 0** | {"action_and_expression":"action_and_expression1", "rate":"1.0"} |
| scene_command |执行场景内的事件调用 |  **返回 0** |{"type":"ChangeBackground","color":{R:1.0,G:0.5,B:1.0} }|
| action_and_camera | 执行人物动作与切换相机显示 |  **返回 0** | {"action":"action1","delay":0.1, "camera":"camera1", "reset":"0"} |
| action_jugong | 执行人物鞠躬 | **返回确切人物鞠躬的动画时间** | {} |
| action_shenshou | 执行人物伸手 | **返回确切人物请的动画时间** | {} |



**scene_command** 现在支持的事件有两种：

| 场景事件类型 | 功能介绍| data数据示例 |
| - | - | - |
| ChangeBackGround |切换背景 | {"type":"ChangeBackground","map":"mapname"} |
| ChangeCamera | 切换相机|{"type":"ChangeCamera","time":"1.0","name":"CameraName" } |
| SetCharacter | 设置人物衣服与蓝图。如人物不存在，将创建，此时同时需要cloth和bp。如人物存在，则可以仅设置cloth和bp中的一项|{"type":"SetCharacter", "name":"zhangyibei", "cloth":"clothName", "bp":"MotionMocap", "hair":"bun", "makeup":"Ada_heavy", "shoe":"Ada_black_leather"} |
| RemoveCharacter | 移除人物|{"type":"RemoveCharacter", "name":"zhangyibei"} |
| SetCharacterVisible | 设置人物可见性，须先创建人物|{"type":"SetCharacterVisible", "name":"zhangyibei", "visible":"false"} |
| SetOnlyShowCharacter | 仅显示指定人物，须先创建人物|{"type":"SetOnlyShowCharacter", "name":"zhangyibei"} |
| SetOriginTransform | 设置所有人物的位置偏移|{"type":"SetOriginTransform", "x":"1.0", "y":"2.0", "z":"3.0", "roll":"90", "pitch":"0", "yaw":"-90"} |
| ResetClothPhysics |重置指定人物的衣服物理效果，须先创建人物|{"type":"ResetClothPhysics", "name":"zhangyibei"}|
| SetMasterVolume | 设置总音量| {"type":"SetMasterVolume","volume":"100"}|
| SetBackgroundSound | 设置背景音| {"type":"SetBackgroundSound",stop:"false","volume":"100","sound":"http://downsc.chinaz.net/Files/DownLoad/sound1/201702/8335.wav"}|
| ChangeBackgroundImage |设置场景内的背景图|{"type":"ChangeBackgroundImage",reset:"false","video":"https://file-examples-com.github.io/uploads/2017/04/file_example_MP4_480_1_5MG.mp4","image":"http://www.dngswin10.com/uploads/allimg/522019/1432563317-3.jpg","effect_type":"wave_drop","recache":"true", "id":"1"}|
| SetDesk|设置场景内的桌子 |{"type":"SetDesk","deskname":"desk_default","visible":"true"}|
| SetItem|设置场景内的物品 |{"type":"SetItem", "iteminfo":{"itemtype":"desk", "visible":"true", "deskname":"MGJ_desk_default1"}}|
| SetLogoVisible |设置logo可见性|{"type":"SetLogoVisible","visible":"false"}|


**widget** 现在支持的事件有两种：

|widget_sub_type| 功能介绍| data数据示例 |
| - | - | - |
| slideshow |切换背景 |{"x-location":"0.0", "y-location":"0.0", "width":"1.0", "height":"1.0", "widget_sub_type":"slideshow",//子类型 "play_order":"positive",//播放顺序，支持positive,reverse,random "show_time":"5.0",//单张图片展示时间 "transition_time":"0.1",//图片切换过渡时间 "effect_type":"type1",//图片切换类型 "images":["https://di.gameres.com/attachment/forum/201807/05/093735i3ay0fdrivdbwaia.jpg", "https://n.sinaimg.cn/tech/transform/345/w141h204/20220325/9e74-3534ee32dd5d6b6b767bb95c68f1b5a6.gif"]//展示的图片信息 }|

**input_key** 支持的按键：

| 支持的按键参数名 | 按键名称 |
|-|-|
|Tab|Tab|
|Enter|Enter|
|Pause|Pause|
|CapsLock|CapsLock|
|Escape|Escape|
|SpaceBar|SpaceBar|
|PageUp|PageUp|
|PageDown|PageDown|
|End|End|
|Home|Home|
|Left|Left|
|Up|Up|
|Right|Right|
|Down|Down|
|Insert|Insert|
|BackSpace|BackSpace|
|Delete|Delete|
|0|Zero|
|1|One|
|2|Two|
|3|Three|
|4|Four|
|5|Five|
|6|Six|
|7|Seven|
|8|Eight|
|9|Nine|
|A|A|
|B|B|
|C|C|
|D|D|
|E|E|
|F|F|
|G|G|
|H|H|
|I|I|
|J|J|
|K|K|
|L|L|
|M|M|
|N|N|
|O|O|
|P|P|
|Q|Q|
|R|R|
|S|S|
|T|T|
|U|U|
|V|V|
|W|W|
|X|X|
|Y|Y|
|Z|Z|
|Num 0|NumPadZero|
|Num 1|NumPadOne|
|Num 2|NumPadTwo|
|Num 3|NumPadThree|
|Num 4|NumPadFour|
|Num 5|NumPadFive|
|Num 6|NumPadSix|
|Num 7|NumPadSeven|
|Num 8|NumPadEight|
|Num 9|NumPadNine|
|Num *|Multiply|
|Num +|Add|
|Num -|Subtract|
|Num .|Decimal|
|Num /|Divide|
|F1|F1|
|F2|F2|
|F3|F3|
|F4|F4|
|F5|F5|
|F6|F6|
|F7|F7|
|F8|F8|
|F9|F9|
|F10|F10|
|F11|F11|
|F12|F12|
|NumLock|NumLock|
|ScrollLock|ScrollLock|
|LeftShift|LeftShift|
|RightShift|RightShift|
|LeftControl|LeftControl|
|RightControl|RightControl|
|LeftAlt|LeftAlt|
|RightAlt|RightAlt|
|LeftCommand|LeftCommand|
|RightCommand|RightCommand|
|;|Semicolon|
|=|Equals|
|,|Comma|
|-|Hyphen|
|_|Underscore|
|.|Period|
|/|Slash|
|`|Tilde|
|[|LeftBracket|
|\\|Backslash|
|]|RightBracket|
|'|Apostrophe|
|\"|Quote|
|(|LeftParantheses|
|)|RightParantheses|
|&|Ampersand|
|*|Asterix|
|^|Caret|
|$|Dollar|
|!|Exclamation|
|:|Colon|


#### UE4HeartBeatRequest









***
### 动捕数据格式
**动捕数据为经过MessagePack处理的Json数据**

动捕数据分为：
* **人的脸部数据**
* **人的身体数据**
* **虚拟相机的位移与转动数据**

所有数据格式如下：
```JavaScript
{
	//数据类型，现在有camera, face, body三种，分别表示相机数据，面部数据和人体数据
	"data_type":"face",
	 //相机或人物唯一标识。如果data_type是camera，则为相机标识，临时只有一个，固定标识为camera。如果data_type为face或body，则标识与UE工程中配置的人物数据表的标识相对应。
	"data_id":"zhangyibei",
	"data":{
		//data_Type 为camera时存在，存储相机的位移与旋转，T表示位移数据，R表示旋转数据
		"Camera":{
			"T":[0.0, 0.0, 0.0],
			"R":[0,0, 0,0, 0.0]
		},
		//data_Type 为face或body时存在，存储每个骨骼的位移与旋转
		"skeleton_frame":{
			"Bone_Name1":{
				"T":[0.0, 0.0, 0.0],
				"R":[0,0, 0,0, 0.0]
			},
			"Bone_Name2":{
				"T":[0.0, 0.0, 0.0],
				"R":[0,0, 0,0, 0.0]
			}
		},
		 //data_Type为face时存在，存储BS数据
		"bs_frame":{
			"BS_Name1":1.0,
			"BS_Name2":2.5,
		}
	}
	}
}
```
***
### 音频数据格式
**动捕数据为经过MessagePack处理的Json数据**

UE4中，音频的默认接收格式为：
采样频率：48000
通道数：1
采样位数：16bit

UE从PixelStreaming的输出为PCM，音频信息固定：
采样频率：48000
通道数：2
采样位数：16bit



[音频 属性详解(涉及采样率、通道数、位数、比特率、帧等)](https://blog.csdn.net/aoshilang2249/article/details/38469051)

音频数据格式如下：
```JavaScript
{
	//音频数据总长度，单次发送音频数据长度不得超过11520000
	"length":5760000‬,
	 //PCM编码的音频数据，以字符串形式存储
	"audio":"............"
}
```

### 从UE4接收音视频流

UE4 可以作为一个 Socket Server, 对外输出渲染后的音视频流。输出的音频流为双通道，采样率为 48000；输出的视频流有两种格式，一种是 H264 流，一种是图片帧流。图片帧流包含 RGB 图片帧和 Alpha 图片帧，两种图片帧流都是经过 JPEG 编码后的数据流。

#### 音视频流控制消息

支持向 UE4 发送消息来控制音视频推流的开始和终止、请求关键帧、选择视频流格式。格式如下：

| 功能 | 消息内容 | 消息 Byte Size |
| - | - | - |
| 请求关键帧 | 0 | 1 |
| 开始推流 | 4 | 1 |
| 终止推流 | 5 | 1 |
| 视频流格式 | 6+格式 | 1+1 |

视频流格式：JPEG: 0, H264: 1，默认输出为 H264 流。

#### 音视频流 Protocol

音视频流 Protocol 由 header 和 payload 组成， header 的格式如下：

时间戳（uint64） + 流种类（uint8） + 帧ID（uint64） + payload Size（uint32）

流种类如下：

| 数值 | 流种类 |
| - | - |
| 0 | 音频流 |
| 1 | RGB 图片帧流 |
| 2 | Alpha 图片帧流 |
| 10 | H264 关键帧视频流 |
| 11 | H264 视频流 |

帧ID在视频流格式是图片帧流时，数值才有效。

#### Client 端的主要流程

* 建立 Socket 连接
* 发送要接收的视频流格式
* 发送开始推流消息
* 接收音视频流
* 发送终止推流消息
* 断开 Socket 连接
