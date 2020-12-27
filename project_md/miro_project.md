[**[이력서 페이지로][go_resume]**]

-------------
# 유니티를 이용한 VR 3D 미로 찾기 게임

### **서론**  
-------
 송파 공업 고등학교에 있는 후배가 학교 동아리 축제 때 사용할 프로그램 작품이 필요했다. 마침 교육부에서 교육 명목으로 미래지향적 기술이 접목 된 고가의 장비들을 신청하면 대여해줌으로서 학생들에ㅐ게 미래 기술을 체험 할 수 있는 기회와 경험을 쌓게 해주는 시간을 가졌다. 그래서 프로젝트 같은 개발 경험과 향후 개발자에게 필요한 미래 기술 인지를 가질 수 있을 것 같아 참여하는 계기가 되었다.<br><br>
 컴퓨터 부에서의 축제 작품은 게임이나 시뮬레이션같 은 시각적이면서 컨트롤이 들어가는 형식으로 만들 계획이었다. 왜냐하면, 대상은 학생들이었고, 공감대가 잘 형성될 게임으로 만들면 더욱 접근이 쉬울 것이라고 생각했기 때문이다. 게임 중에서도 평소 잘 접해보지 못한 가상현실 게임으로 개발을 하기로 정했다. <br><br>
 가상현실에는 모션 캡처, 전방위 트레드밀 같은 다양한 입력 도구를 이용한 분야들이 있는데 축제기간까지 개발기간이 약 2주밖에 남지 않아 기타 장비나 코딩과 같은 개발에 필요한 자원이나 시간이 많이 들지 않는 HMD를 이용해 시선처리만 가상현실이며 나머지 컨트롤 기기 같은 기존의 입력 도구를 통해 다루어지는 어플리케이션 게임을 컨셉으로 잡았다.<br><br>
 개발 엔진은 오픈소스가 다양하고 입문하기 쉬운 유니티를, 디바이스는 삼성 Gear VR을 사용했다. 게임의 내용은 복잡하지 않으면서도 누구나 쉽게 할 수 있고, 시간이 많이 걸리지 않는 미로 찾기를 타이틀로 결정했다. 

--------------

<br>  

### **본론**
-------
**개발 초기** : 
- 앞으로 다루게 될 VR 환경에 맞춰 유니티 인터페이스를 조정하고 개발에 필요한 자원인 라이브러리, 패키지 등을 서치 및 임포트 하여 쓰는 등 각 기 다른 VR 디바이스 호환성을 유니티에 맞추는 작업을 했다.



**개발 중반** :
- 미로 찾기 게임에는 미로로 구성되어 있는 벽 오브젝트, 사용자의 조작에 의해 움직여지는 캐릭터 오브젝트, 맵 벽 과 같은 다양한 시각적인 요소들과 사용자 HMD의 조작 뱡향에 의해 시선처리가 달라지는 카메라, 게임 전체의 틀이 되는 맵, 오브젝트 마다 필요한 스킨 등이 필요해 직접 수집했다.

- 맵 디자인에는 미로를 직접 그리려고 했으나 잘 구현이 되자 않아 인터넷을 참고했다. 미로의 벽과 캐릭터는 Cube 오브젝트를 사용하였고, 땅은 Plane 오브젝트를 사용하였다. 오브젝트의 이름은 혼동이 오지 않게 Cube, Player, Plane으로 설정했다.  <br><br>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src = asset_info.PNG width = 400 height = 60/>
<br><br>


- 캐릭터 오브젝트 움직임까진 유니티 엔진으로는 할 수 없었기 때문에 C#을 이용한 스크립트를 사용해야만했다. 사실 C#을 접해본 적이 많지 않아 유니티에 사용되는 스크립트의 기초적인 구조를 공부했다.

 **namespace** : 스크립트에서 사용할 수 있는 주요 명령어가 들어있는 곳의 위치다. C언어의 #include, 자바의 import랑 비슷한 기능이다.

**public class go** : 클래스와 클래스의 선언부이다. 유니티 C# 스크립트는 하나의 클래스이다.

**MonoBehaviour** : 기본 클래스를 상속하는 부분이다.

**void start()** : 변수들의 초기화를 할 때 사용된다. 이 부분에서 어떤 액션을 설정하면 게임시작 시 자동으로 실행된다.

 **void update()** : 매 프레임마다 반복 실행되는 함수이다. 예를 들어 초당 60프레임의 게임은 1초에 60번 화면을 그리므로 60번 호출이 된다.


* 여기까지가 공부한 유니티 스크립트의 기본 구조이다. 다음은 캐릭터의 움직임을 구현한 소스이다. 클래스의 이름은 go.sc다.

* 각 변수들은 float형으로 초기화를 했다. sp는 캐릭터의 움직임 속도를 설정한 변수이고, rotsp는 캐릭터의 회전속도를 설정한 변수이다.
 
<br><br>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src = cpp_1.PNG width = 3500 height = 350/>
<br><br>



* **this.transform.localScale = new Vector3(0.3f, 1f, 0.3f);**  
 this는 자기 자신, 즉 캐릭터를 나타내는 것이다. transform은 변화한다라는 뜻이 되는데 여기서는 localScale, 캐릭터의 크기를 조정하는 것이다. Vector3는 가로(x), 세로(y), 높이(z)를 모두 조정하는 함수이다. Vector2와는 가로(x), 세로(y)밖에 조정이 되지 않는 차이점이 있다.

* **if (Input.GetKey(KeyCode.UpArrow))**  
 Input.GetKey는 키보드 혹은 다른 입력장치에서 신호를 받아오는 함수이다. 여기서는 Gear VR 컨트롤러가 해당되었다. KeyCode.UpArrow는 밑의 화살표의 킷값을 받아오는 기능을 한다.

* **transform.Translate(Vector3.forward * sp * Time.deltaTime);**  
 여기서 Translate는 캐릭터가 이동하기위한 함수이다. 여기서 Vector3.forward는 방향 벡터값(앞)을 설정해주고 sp와 Time.deltaTime을 곱해줌으로써 속도와 방향이 결정된다. 만약 sp값을 곱해주지 않는다면 매 프레임마다 update를 호출하는 특성 상 좋은 PC와 안 좋은 PC에서 속도가 다르게 나타날 것이다.

* **this.transform.localScale = new Vector3(0.3f,0.3f,0.3f);**  
 이 부분은 DownArrow부분이다. 밑의 화살표 키를 눌렀을 때 localScale을 통해 캐릭터의 y축 부분을 줄여준다. 맵의 곳곳에 위쪽 부분이 뚫려있지 않은 부분이 있는데 그곳을 통과하기 위한 기능이다.

* **this.transform.Rotate(0.0f, -90.0f * Time.deltaTime * 2, 0.0f);**  
 LeftArrow와 RightArrow 부분에 한 줄 더 추가되어있는데, 이것은 회전하는 부분이다. Rotate가 회전기능을 해 주는 함수이다. Y축을 기준으로 왼쪽 방향으로 한 프레임 당 *2 번, 즉 2도씩 왼쪽으로 움직일 수 있다. 반대로 오른쪽으로 움직이려면 –90.0f를 90.0f로 바꿔주면 된다.

- 캐릭터의 움직임구현은 끝났지만, 카메라가 따라오지 않아 3인칭 시점이 되어 버렸다. 지금 나오는 코드는 캐릭터의 1인칭 시점으로 보이게 하는 코드이다. 클래스이름은 follow.sc다.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src = cpp_2.PNG width = 3500 height = 350/>
<br><br>


* **public 및 private 변수들**   Transform player는 유니티 캐릭터의 객체, 즉 카메라가 따라가야 할 대상이다. dist는 카메라와 캐릭터의 거리, height는 카메라의 높이, smoothRotate는 카메라가 이동하거나 회전할 때 더욱 부드럽게 만들어주는 변수이다. Transform tr은 카메라 자신의 Transform 변수이다.

* **tr = GetComponent<Transform>();**  
 tr변수에 Transform이란 오브젝트를 저장시킨다.


* **float currYAngle = Mathf.LerpAngle(tr.eulerAngles.y, player.eulerAngles.y, smoothRotate * Time.deltaTime);**  
 Mathf.LerpAngle(a,b,t)함수는 a부터 b까지 t시간동안 각도를 부드럽게 회전시켜주는 함수이다. tr.eulerAngles.y(카메라의 y축에 대한 오일러 각도)부터 player.eulerAngles.y(타겟의 각도)까지 지정한 시간까지 회전하는 것이다. 만약 회전 시간을 조정하고 싶으면 smoothRotate변수를 조정하면 된다.


*  **Quaternion rot = Quaternion.Euler(0, currYAngle, 0);**  
오일러의 법칙을 사용했지만, x,y,z축이 차례로 회전하기 때문에 짐벌락이라는 회전이 멈추는 현상이 발생하게 된다. 이를 해결하기 위한 것이 Quaternion.Euler()함수이다. 이 함수는 3개의 축을 한 번에 회전시키는 함수이기 때문에 짐벌락이 걸리지 않는다. 보통 transform.rotation함수를 사용하면 이 Quaternion.Euler()함수를 이용해 오일러를 Quaternion화를 사용하여 대입해야한다.

 * **tr.position = player.position - (rot * Vector3.forward * dist);**  
 이 부분은 카메라 위치를 타겟의 회전 각도만큼 회전을 하는 공식이다. 우리는 1인칭 시점으로 했기 때문에 height변수는 사용하지 않았다. 만약 사용한다면 뒤쪽에 +(Vector3.up*height);를 대입하면 카메라가 캐릭터 뒤의 모습을 보면서 플레이가 가능하다.

 * **tr.LookAt(player);**
 카메라가 player를 바라보는 부분이다.
개발 후반:
- 유니티 내에선 정상 작동 하지만 실제 인 앱에서도 제대로 되는지 확인 여부 필요 APK를 추출하고 핸드폰에 앱을 추가해 VR 디바이스에 연결 및 앱 테스트를 했다.


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src = miro.PNG width = 400 height = 300/>
-----------------

<br><br>

### **발생문제 및 해결방법**
----------
<br><br>

1. **Player가 벽을 계속 뚫는 버그.**

 개발 과정 중 미로 찾기를 하는 Player라는 객체가 벽을 계속 뚫는 현상이 발생했다. 기본적으로 Collider라는 컴포넌트가 제공이 되는데 이는 충돌을 감지해 주는 컴포넌트이다. 이 컴포넌트를 활성화하면 벽과의 충돌을 감지해 벽 밖으로 나가지 못하게 해주어야 하는데 전혀 적용이 되지 않았다. 처음에는 Collider를 이용해 이동/회전/크기변경이 생길 때마다 계산을 수행하기 위해 소스코드를 수정하려고 하였는데, 이 방법은 리소스를 많이 잡아먹어 상당히 비효율적이고 쉽지 않은 과정이었다. 그렇게 찾게 된 것이 Rigidbody라는 컴포넌트인데 Collider와의 차이점은 플레이하고 있는 객체가 동적인가 혹은 정적인가의 차이점이 있었다.<br><br> Collider는 미로의 벽처럼 가만히 있는 정적인 객체에 적용시켜주면 좋고, Rigidbody는 Player처럼 계속 움직이는 동적인 객체에 적용을 해 주면 좋다. Rigidbody라는 컴포넌트를 통해 벽을 계속 뚫는 버그를 해결할 수 있었다.<br><br>

2. **VR 컨트롤러와의 연동.**

 어플리케이션 디버깅까지 문제없이 진행되다가 마지막 컨트롤러가 게임과의 호환이 되질 않았다. 각 VR 디바이스마다 고유의 호환성이 있기에 소스 코딩에 쓰이는 함수가 제각각이었다. 우리의 경우 삼성 Gear VR은 패키지를 받아 OVR...() 이라는 메소드로 처리하는 것이 일반적이었다. 그러나 우리는 어떤 방법도 먹히질 않았고, 라이브러리를 끌어와 써도 응답하지 않았다. <br><br>사실 이 문제에 대해 제일 오래 고민했지만, 유니티 스크립트나 패키지 등 외부파일들을 일일이 다른 것을 다 넣어보고 계속 시도해봤는데 갑자기 연동이 돼서 어떤 파일을 임포트 하였는지는 모르지만 상당히 간단하게 해결이 되었다. 이 문제의 이유는 모르지만, 유니티에 적용되는 패키지 파일과 삼성 Gear VR의 패키지의 버전이 서로 호환이 돼서 해결이 된 것 같다.<br><br>

3. **카메라의 시선이 벽을 뚫고 보는 문제.**
 
  이 문제는 벽이나 지형이 카메라가 비추기에 좁은 공간으로 갈 때, 카메라가 벽을 뚫고 Player를 보여주는 방식이었다. 이 문제를 해결하기 위해 2가지 방법을 생각했다. 첫 번째로, 좁은 공간에 들어가면 약간 머리 쪽으로 비치게 해주는 방식이다. 이는 카메라와 Player의 거리조정을 하면 쉽게 해결될 문제였다.<br><br> 두 번째로는 카메라 주변 물체들을 탐색해서 옆에 벽이 있으면 그 이상으로 넘어가지 않는 방식을 생각했다. 이 2가지의 방법을 생각하던 중 Raycast()라는 함수가 있었다. 이것을 직역하면 광선을 쏜다라는 뜻인데, 어떤 충돌체와 붙이치면 RaycastHit에 그 정보를 보내 카메라가 벽을 뚫지 않고 보여줄 수 있어서 이 문제를 해결할 수 있었지만, 축제기간까지 시간이 얼마 남지 않아 이 부분을 추가하지 못한 것이 상당히 아쉽다고 느껴졌다.<br><br>

-----------


### **결론**
------------
<br>

 축제의 결과는 성공적이라고 했다. 되게 간단하지만 평소 잘 접하지 못하는 부류라 그런지 친구들이 많이 신기해했고, 재미있었다고 했다. 또한 시상식에서도 좋은 결과를 획득했다고 했다. 가끔 어지러움을 호소하는 학생도 있었다고 한다. 이는 VR 특성 상 어쩔 수 없는 부분이라고 생각하지만 모든 사용자들이 불편함을 느끼지 않고 사용할 수 있다면 개선을 하고 싶다.<br><br>
 어렵게만 생각하던 게임 엔진과 VR 같은 미래지향적 기술을 직접 다뤄보며 한층 가까워진 것 같고, 앞으로 더 나아가야 할 부분을 직접 찾아보고 다른 분야들도 해보고 싶은 배움의 욕심과 자신감을 얻게 되었다. 또한 지금까진 혼자서 공부하고 알아가는 개인 공부에 많은 시간을 보내었기에 본인 행동에 따른 결과는 본인의 문제였지만 이번 프로젝트를 통해 팀원들과 함께 작업을 하면서 나의 행동에 따른 책임은 나 자신만의 문제가 아니고 다른 팀원에게도 영향을 끼치는 책임감이라는 것을 얻게 되었다.<br><br>
 개발기간이 길지 않아 오픈소스나 유튜브, 구글링 등 인터넷 자료들이 있다고 단순히 사용만 하는 것이 아닌 본인의 것으로 만들지 않는 이상 구상하던 것의 결과가 나오지 않거나 본인이 이해를 하지 못해 개발에 차질이 생기는 경험을 해봤다. 이 개발과정을 적는 것도 내가 개발했던 것들을 다시 공부하기 위해서이고, 나중에도 사용하기 위해서이다. 또한 앞으로의 프로젝트에서도 개발 과정을 작성 할 것이다.<br><br>
 마지막으로 아직 해결하지 못했던 카메라의 시선처리를 좀 더 개선을 시키고, 다른 사용자와 매칭이 돼서 서로 경쟁을 펼치는 대중적인 어플로 업데이트를 개발하고 싶다. 그러면 이전에 진행하였던 소켓 프로그래밍을 좀 더 심도 있게 공부하고, 유니티 물리엔진을 더욱 정교하게 만들 수 있도록 노력해 나갈 것이다.
 
-------

[**[이력서 페이지로][go_resume]**]




 [go_resume]: https://github.com/HanBI24/Resume1/blob/main/README.md