# Code-Robot-tranh-vat-can
Tự làm xe Robot Tránh Vật Cản Bằng Cảm Biến Siêu Âm

-Code mô hình xe robot tránh vật cản
- #include <Servo.h>
- #define trig 3 //chân trig của HC-SR04
- #define echo 4//chân echo của HC-SR04

- Servo srf05;  // create servo object to control a servo
- #define inA1 6 //Định nghĩa chân in1 của động cơ A
- #define inA2 7 //Định nghĩa chân in2 của động cơ A
- #define inB1 8 //Định nghĩa chân in1 của động cơ B
- #define inB2 9 //Định nghĩa chân in2 của động cơ B

- void setup()
{
  pinMode(inA1, OUTPUT);//Set chân in1 của dc A là output
  pinMode(inA2, OUTPUT);//Set chân in2 của dc A là output
  pinMode(inB1, OUTPUT);//Set chân in1 của dc B là output
  pinMode(inB2, OUTPUT);//Set chân in2 của dc B là output
  pinMode(trig,OUTPUT);//chân trig sẽ phát tín hiệu
  pinMode(echo,INPUT);//chân echo sẽ nhận tín hiệu
  Serial.begin(9600);
  srf05.attach(5);  // attaches the servo on pin 5 to the servo object
}

void loop() 
{
  objectAvoider (inA1, inA2, inB1, inB2,50, 1000);
}

int objectDistance_cm (byte angle)
{
  srf05.write(angle);
  delay(500);
  unsigned long duration;//biến đo thời gian
  int distance;//biến lưu khoảng cách

  /* phát xung từ chân trig */
  digitalWrite(trig,0);//tắt chân trig
  delayMicroseconds(2);
  digitalWrite(trig,1);// phát xung từ chân trig
  delayMicroseconds(5);// xung có độ dài 5 microSeconds
  digitalWrite(trig,0);//tắt chân trig
  
  /* tính toán thời gian */
  duration = pulseIn(echo,HIGH);//đo độ rộng xung HIGH ở chân echo. ( http://arduino.vn/reference/pulsein )
  distance = int(duration/2/29.412);//tính khoảng cách đến vật.

  /* in kết quả ra Serial monitor */
  //Serial.print(distance);
  //Serial.println("cm");
  //delay(200);
  return distance;
}

void robotMover (byte inR1, byte inR2, byte inL1, byte inL2, byte action)
{
  /*
  inR1 inR2 là 2 chân tín hiệu động cơ bên phải
  inL1 inL2 là 2 chân tín hiệu động cơ bên trái
  action= 0 đứng yên
  action =1 đi thẳng
  action =2 lùi lại
  action =3 quay trái
  action =4 quay phải
  action =5 rẽ trái
  action =6 rẽ phải
  action =7 rẽ lùi trái
  action =8 rẽ lùi phải

  */
  switch (action)
  {
    case 0:// không di chuyển
      motorControlNoSpeed(inR1, inR2, 0);
      motorControlNoSpeed(inL1, inL2, 0);
      break;
    case 1://đi thẳng
      motorControlNoSpeed(inR1, inR2, 1);
      motorControlNoSpeed(inL1, inL2, 1);
      break;
    case 2:// lùi lại
      motorControlNoSpeed(inR1, inR2, 2);
      motorControlNoSpeed(inL1, inL2, 2);
      break;
    case 3:// quay trái
      motorControlNoSpeed(inR1, inR2, 1);
      motorControlNoSpeed(inL1, inL2, 2);
      break;
    case 4:// quay phải
      motorControlNoSpeed(inR1, inR2, 2);
      motorControlNoSpeed(inL1, inL2, 1);
      break;
    case 5:// rẽ trái
      motorControlNoSpeed(inR1, inR2, 1);
      motorControlNoSpeed(inL1, inL2, 0);
      break;
    case 6:// rẽ phải
      motorControlNoSpeed(inR1, inR2, 0);
      motorControlNoSpeed(inL1, inL2, 1);
      break;
    case 7:// rẽ lùi trái
      motorControlNoSpeed(inR1, inR2, 2);
      motorControlNoSpeed(inL1, inL2, 0);
      break;
    case 8:// rẽ lùi phải
      motorControlNoSpeed(inR1, inR2, 0);
      motorControlNoSpeed(inL1, inL2, 2);
      break;
    default:
      action = 0;
      
  }
}


void motorControlNoSpeed (byte in1,byte in2, byte direct)
{
// in1 and in2 are 2 signal pins to control the motor
// en is the enable pin
// the defauspeed is the highest
// direct includes:
//    0:Stop
//    1:Move on 1 direct
//    2:Move on another direct
switch (direct) 
  {
    case 0:// Dừng không quay
      digitalWrite(in1,LOW);
      digitalWrite(in2,LOW);
      break;
    case 1:// Quay chiều thứ 1
      digitalWrite(in1,HIGH);
      digitalWrite(in2,LOW);
      break;    
    case 2:// Quay chiều thứ 2
      digitalWrite(in1,LOW);
      digitalWrite(in2,HIGH);
      break;
    //default: 
  }
}

void objectAvoider (byte inR1, byte inR2, byte inL1, byte inL2, byte allow_distance, int turn_back_time)
{
  
  robotMover(inR1,inR2,inL1,inL2,1);
  Serial.println("Tiến");
  //delay(10);
  int front_distance=objectDistance_cm (90);
  int left_distance;
  int right_distance;
  int max_distance;
  if (front_distance > allow_distance)
    {
      robotMover(inR1,inR2,inL1,inL2,1);
      Serial.println("Tiến");
      delay(10);
    }
    if (front_distance <= allow_distance)
    {    
      robotMover(inR1,inR2,inL1,inL2,2);
      Serial.println("Lùi");
      delay(300);
      robotMover(inR1,inR2,inL1,inL2,0);
      left_distance = objectDistance_cm (180); //đo khoảng cách bên trái
      Serial.println("left: ");    
      Serial.println(left_distance);
      //delay (3000);
      right_distance = objectDistance_cm (0); //đo khoảng cách bên phải
      Serial.println("right: ");
      Serial.println(right_distance);
      Serial.println("front: ");
      Serial.println(front_distance);
      //delay (3000);
      max_distance = max(left_distance,right_distance);// so sánh giá trị lớn nhất với khoảng cách bên phải (gán bằng cái lớn nhất)
      if  (max_distance==left_distance)
      {
        robotMover(inR1,inR2,inL1,inL2,3);
        Serial.println("Rẽ trái");
        delay (turn_back_time/2);// Nếu bên trái là khoảng cách lớn nhất thì rẽ trái
      }
      else
      {
        if  (max_distance==right_distance)
        {
          robotMover(inR1,inR2,inL1,inL2,4);// Nếu bên phải có khoảng cách lớn nhất thì rẽ phải
          Serial.println("Rẽ phải");
          delay (turn_back_time/2);
        }
      }  
    }
}


Các sản phẩm sử dụng cho mô hình xe robot dò đường 
<a href="https://nshopvn.com/product/cam-bien-sieu-am-ioe-sr05-ho-tro-ngo-ra-ttl/"><strong>Cảm biến siêu âm IOE-SRF05</strong></a>
<a href="https://nshopvn.com/product/cam-bien-sieu-am-hc-sr04/"><strong>Cảm biến siêu âm HCSR04</strong></a
<a href="https://nshopvn.com/product/mach-dieu-khien-dong-co-dc-l298n/"><strong>Mạch điều khiển động cơ DC L298</strong></a>
<a href="https://nshopvn.com/product/dong-co-servo-mg996-360-do/"><strong>Động cơ RC Servo 360 độ</strong></a>
