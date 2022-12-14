# 一、电路接线
## RFID-RC522
|RC522引脚号| Arduino板引脚号 |
|--|--|
| 3V3 | 3.3V |
| GND | GND |
| MISO | 12 |
| MOSI | 13 |
| SCK | 13 |
| SDA | 10 |
## 蜂鸣器
|蜂鸣器引脚号| Arduino板引脚号 |
|--|--|
| VCC | 5V |
| GND | GND |
| I/O | 8 |
## LED
|LED引脚号| Arduino板引脚号 |
|--|--|
| LedRed| 5 |
| LedGreen| 6 |

# 二、代码
源代码来源：[DIY打造智能门禁锁RFIDRC522原理与应用](https://www.bilibili.com/video/BV1gR4y147RX/)
功能：假如卡可以进入，则绿灯闪烁，蜂鸣器叫两声
假如卡不可以进入，则红灯闪烁，蜂鸣器长鸣

记得安装RFID的库
```c
#include <SPI.h> 
#include <RFID.h>

RFID rfid(10, 9);       
unsigned char status; 
unsigned char str[MAX_LEN]; 

String accessGranted [2] = {"143043708", "123035296"};  //序列号位置
int accessGrantedSize = 2;                                //几个卡号
       
boolean locked = true;

int redLEDPin = 5;
int greenLEDPin = 6;
int pinBeep= 8;

void setup() 
{ 
  Serial.begin(9600);     //串口开启波特率9600
  SPI.begin();            //通过SPI与读卡器连接
  rfid.init();            //RFID定义

  pinMode(redLEDPin, OUTPUT);     //LED闪烁表示开启
  pinMode(greenLEDPin, OUTPUT);
  pinMode(pinBeep, OUTPUT);
  digitalWrite(pinBeep, HIGH);
    
  digitalWrite(redLEDPin, HIGH);
  delay(200);
  digitalWrite(greenLEDPin, HIGH);
  delay(200);

  digitalWrite(redLEDPin, LOW);
  delay(200);
  digitalWrite(greenLEDPin, LOW);

  Serial.println("Please put the card to the reader");
} 

void loop() 
{ 
  if (rfid.findCard(PICC_REQIDL, str) == MI_OK)   //等待卡被放到读卡器上
    Serial.println("Card found"); 
    String temp = "";                             //储存一个值到temp中
    if (rfid.anticoll(str) == MI_OK)              //读取卡的串口号 
    { 
      Serial.print("The card's ID number is : "); 
      for (int i = 0; i < 4; i++)                 //记录下temp值
      { 
        temp = temp + (0x0F & (str[i] >> 4)); //保留前四位
        temp = temp + (0x0F & str[i]); //保留后四位
      } 
      Serial.println (temp);
      checkAccess (temp);     //检查此卡能否接入
    } 
    rfid.selectTag(str); //锁卡防止冗余读卡
  
  rfid.halt();
}

void checkAccess (String temp)    //检查卡能否被读写的函数
{
  boolean granted = false;
  for (int i=0; i <= (accessGrantedSize-1); i++)    //在已储存的卡号中检测此卡能否接入
  {
    if(accessGranted[i] == temp)            //如果卡号允许被接入则开锁
    {
      Serial.println ("Access Granted");
      granted = true;

      //蜂鸣器叫两下
      for(int i=0;i<2;i++)
      {
      digitalWrite(pinBeep,LOW);
      delay(100);
      digitalWrite(pinBeep,HIGH);
      delay(100);
      }
      //绿灯闪烁
      digitalWrite(greenLEDPin, HIGH);   
      delay(200);
      digitalWrite(greenLEDPin, LOW);
      delay(200);
      digitalWrite(greenLEDPin, HIGH);
      delay(200);
      digitalWrite(greenLEDPin, LOW);
      delay(200);

    }
  }
  if (granted == false)     
  {
    Serial.println ("Access Denied");
    //蜂鸣器长鸣
    digitalWrite(pinBeep,LOW);
    delay(1000);
    digitalWrite(pinBeep,HIGH);
    delay(100);
    //红灯闪烁
    digitalWrite(redLEDPin, HIGH);      
    delay(200);
    digitalWrite(redLEDPin, LOW);
    delay(200);
    digitalWrite(redLEDPin, HIGH);
    delay(200);
    digitalWrite(redLEDPin, LOW);
    delay(200);
  }
}
```
# 三、效果
我有两张卡，一张可以接入，一张不可以接入
![请添加图片描述](https://img-blog.csdnimg.cn/07c4569d15f342ad9c1eceb2ec9469ed.png)


