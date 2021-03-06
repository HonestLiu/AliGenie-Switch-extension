## 介绍

项目原名：“天猫精灵可控开关插件”  
使用arduino借助Blinker提供的物联网服务实现远程控制舵机和通过接入天猫精灵平台和小米IOT平台实现天猫精灵和小爱同学的语音控制，同时支持Blinker APP手机控制。

注：如果图片等无法显示或想查看更多我的文章，可前往[**我的博客**](http://www.liuchengblog.top/)查看“[**免布线物联网开关扩展**](http://www.liuchengblog.top/index.php/2020/08/18/66/)”文章，其中**附带更多图片而且还有视频演示**
## 实际安装图

> 这里只供演示,各位可以按需3D打印外壳,Mini D1很小的，足够你们发挥😅😅😅

> 不必一定使用Mini D1开发版，只有是Esp8266的开发版都行。

<img src="https://s1.ax1x.com/2020/07/26/apaqcd.jpg" style="zoom: 50%;" /> 

<img src="https://s1.ax1x.com/2020/07/26/apab1H.jpg" style="zoom: 50%;" /> 



## 接线图
<img src="https://s1.ax1x.com/2020/07/26/apakTK.png" alt="1" style="zoom: 67%;" /> 



## 代码如下

> 注：要记得安装“esp8266库”和"Blinker库"哦！否者会烧写失败。

```
#define BLINKER_WIFI
#define BLINKER_ALIGENIE_OUTLET
#define BLINKER_MIOT_OUTLET
 
#include <Blinker.h>
#include <Servo.h>
 
#define PIN_SERVO D4
Servo myservo;
 
char auth[] = "Blinker Key";
char ssid[] = "Wifi name";
char pswd[] = "wifi key";
 
bool oState = false;
BlinkerButton Button1("Light");
 
void button1_callback(const String & state)
{
    
   BLINKER_LOG("get button state: ", state);
 
    
    if (state == BLINKER_CMD_ON) {
        myservo.write(32);
        digitalWrite(D3, HIGH);
        Button1.text("灯已开启");
        delay(500);
        digitalWrite(D3, LOW);
        myservo.write(90);
        Button1.print("on");
    }
    else if (state == BLINKER_CMD_OFF) {
        myservo.write(134);
        digitalWrite(D3, HIGH);
        Button1.text("灯已关闭");
        delay(500);
        digitalWrite(D3, LOW);
        myservo.write(90);    
        Button1.print("off");
        
   }
   
}
 
void aligeniePowerState(const String & state)
{
    BLINKER_LOG("need set power state: ", state);
 
    if (state == BLINKER_CMD_ON) {
        myservo.write(32);
        digitalWrite(D3, HIGH);
        
        delay(500);
        digitalWrite(D3, LOW);
        myservo.write(90);
        BlinkerAliGenie.powerState("on");
        BlinkerAliGenie.print();
 
        oState = true;
    }
    else if (state == BLINKER_CMD_OFF) {
        myservo.write(134);
        digitalWrite(D3, HIGH);
        Button1.text("灯已关闭");
        delay(500);
        digitalWrite(D3, LOW);
        myservo.write(90);
        BlinkerAliGenie.powerState("off");
        BlinkerAliGenie.print();
 
        oState = false;
    }
}
 
void aligenieQuery(int32_t queryCode)
{
    BLINKER_LOG("AliGenie Query codes: ", queryCode);
 
    switch (queryCode)
    {
        case BLINKER_CMD_QUERY_ALL_NUMBER :
            BLINKER_LOG("AliGenie Query All");
            BlinkerAliGenie.powerState(oState ? "on" : "off");
            BlinkerAliGenie.print();
            break;
        case BLINKER_CMD_QUERY_POWERSTATE_NUMBER :
            BLINKER_LOG("AliGenie Query Power State");
            BlinkerAliGenie.powerState(oState ? "on" : "off");
            BlinkerAliGenie.print();
            break;
        default :
            BlinkerAliGenie.powerState(oState ? "on" : "off");
            BlinkerAliGenie.print();
            break;
    }
}

void miotPowerState(const String & state)
{
    BLINKER_LOG("need set power state: ", state);

    if (state == BLINKER_CMD_ON) {
        myservo.write(32);
        digitalWrite(D3, HIGH);
        
        delay(500);
        digitalWrite(D3, LOW);
        myservo.write(90);
        BlinkerAliGenie.powerState("on");
        BlinkerAliGenie.print();

        oState = true;
    }
    else if (state == BLINKER_CMD_OFF) {
        myservo.write(134);
        digitalWrite(D3, HIGH);
        Button1.text("灯已关闭");
        delay(500);
        digitalWrite(D3, LOW);
        myservo.write(90);
        BlinkerAliGenie.powerState("off");
        BlinkerAliGenie.print();
        
        oState = false;
    }
}

void miotQuery(int32_t queryCode)
{
    BLINKER_LOG("MIOT Query codes: ", queryCode);

    switch (queryCode)
    {
        case BLINKER_CMD_QUERY_ALL_NUMBER :
            BLINKER_LOG("MIOT Query All");
            BlinkerMIOT.powerState(oState ? "on" : "off");
            BlinkerMIOT.print();
            break;
        case BLINKER_CMD_QUERY_POWERSTATE_NUMBER :
            BLINKER_LOG("MIOT Query Power State");
            BlinkerMIOT.powerState(oState ? "on" : "off");
            BlinkerMIOT.print();
            break;
        default :
            BlinkerMIOT.powerState(oState ? "on" : "off");
            BlinkerMIOT.print();
            break;
    }
}
 
void dataRead(const String & data)
{
    BLINKER_LOG("Blinker readString: ", data);
 
    Blinker.vibrate();
    
    uint32_t BlinkerTime = millis();
    
    Blinker.print("millis", BlinkerTime);
}
 
void setup()
{
    Serial.begin(115200);
    BLINKER_DEBUG.stream(Serial);
 
    pinMode(D3, OUTPUT);
    digitalWrite(D3, LOW);
    Button1.attach(button1_callback);
    Blinker.begin(auth, ssid, pswd);
    Blinker.attachData(dataRead);
    
    BlinkerAliGenie.attachPowerState(aligeniePowerState);
    BlinkerAliGenie.attachQuery(aligenieQuery);
    BlinkerMIOT.attachPowerState(miotPowerState);
    BlinkerMIOT.attachQuery(miotQuery);
    
    myservo.attach(PIN_SERVO);
 
}
 
void loop()
{
    Blinker.run();
}
```

> 附上Blinker App 控制界面配置码，喜欢的复制粘贴后点更新配置即可。
>
> 注：自己设置的话，拖一个开关模块，按键类型改为“开关按键”数据键名改为“Light”即可

```
{¨config¨{¨headerColor¨¨transparent¨¨headerStyle¨¨light¨¨background¨{¨img¨´´}}¨dashboard¨|{¨type¨¨btn¨¨ico¨¨fad fa-lightbulb-on¨¨mode¨Ê¨t0¨¨灯光状态¨¨t1¨¨文本2¨¨bg¨Ì¨cols¨Í¨rows¨Í¨key¨¨Light¨´x´Ë´y´Í¨speech¨|¦¨act¨¨cmd¨‡¨on¨¨开灯¨¨off¨¨关灯¨—÷¨lstyle¨Ë}÷¨actions¨|¦ß
```


