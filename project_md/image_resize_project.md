[**[이력서 페이지로][go_resume]**]

-------------
# **프로젝트(텐서플로 이미지 복원)**
텐서플로를 이용해 저해상도 이미지들을 고해상도로 복원하는 이미지 복원 모델을 구현한다.
구현에는 입력 데이터인 이미지 데이터를 가능한 한 무수하게 수집, 이미지 사이즈를 학습에 맞게 리사이즈 후 배치처리, 이미지 복원 역할을 하는 학습 모델 구현 여기에는 다중 계층인 CNN을 모델로 뽑았다.

1. **데이터 모음**  
처음 데이터를 모으기 시작할 때는 이미지를 무료 제공하여 주는 픽사베이에서 하나씩 다운을 받았습니다. 그러나 필요한 데이터의 수량을 일일이 다운을 받아서 모으기에는 시간이 너무 많이 걸리기 때문에 지정한 페이지에 존재하는 이미지를 전부 다운을 받아주는 앱과 프로그램을 찾아보았고 그 중 Httrack Website Copier를 이용하여 픽사베이에서 강아지의 실제 이미지에 해당하는 것만을 다운을 받기 위하여 조건을 .jpg와 .jpeg로만 설정하여 다운을 받았습니다. <br><br> 그 후 Google Vision API를 이용하여서 개의 얼굴 부분만을 추출하려 하였으나 실패하였습니다. 그러나 조원들과의 토의 결과 프로젝트의 목표가 이미지의 픽셀을 복원하는 것이기 때문에 추출할 필요가 없었습니다. 이미지의 픽셀을 낮추기 위해서 16*16사이즈로 리사이즈를 하였고 픽셀을 복원을 비교하기 위한 사진을 32*32사이즈로 리사이즈를 하여서 output으로 설정하였습니다.  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src = resume/pro.PNG width = 500 height = 300/>

1. **배치처리**  
텐서플로를 이용한 기계학습 모델 작성 시 지금까지 해왔던 MNIST처럼 입력 데이터들이 항상 보기 좋게 정리되어 있진 않습니다. 실제로의 입력 데이터들은 적재나 분류 작업 같은 손 가는 일이 많다고 합니다. 그래서 입력 데이터를 입력받을 때 효율적으로 입력을 받을 파일 처리 작업을 다루어 봅니다. 작업 처리에는 주로 큐 방식을 이용한다고 합니다.<br><br>
모델을 학습시킬 시 트레이닝 데이터를 모델에 적용하는 방법을 피딩 이라고 부르는데 메모리상에 어떤 변수 리스트 형태로 값을 저장 후에 모델을 세션에 실행할 때 리스트에서 값을 하나씩 읽어와 모델에 넣는 방식이라고 합니다. <br><br>
그러나 이러한 피딩은 메모리에 모두 적재 되어야 특징이 있는데 실제로 모델 학습 때는 데이터양이 무수하기에 오버플로우 같은 문제로 모두 적재하고 학습시킬 수 없습니다. 해결책으론 파일에서 데이터를 읽어가면서 읽어드린 순서로 모델에 피딩을 하는 것이 이것이 큐 방식입니다. <br><br>  
큐에는 데이터가 들어갈 때 큐 러너라는 곳에서 들어오는데 큐에 데이터를 어떻게 넣을지 정의 같은 작업을 맡습니다. 이러한 큐 러너는 멀티 쓰레드로 작동하고 이러한 쓰레드들 역시 별도로 코디네이터라는 곳에서 관리합니다.
멀티 쓰레드 성질 덕에 쓰레드가 각기 다른 값이어도 랜덤하게 실행되어 데이터에 들어 가집니다. 파일 처리에는 큐뿐만 아니라 리더, 디코더 같은 부가적인 기능도 제공합니다. 리더는 파일에서 데이터를 읽어오는 컴포넌트로 파일명이 저장된 큐에서 하나씩 읽어 들이며 각기 파일에 맞는 형태를 리턴 하는 컴포넌트입니다. 디코더는 리더에서 읽어 들인 데이터가 아직 해석된 데이터가 아니라 각 필드 데이터값 같은 해석된 파일로 리턴 하는 컴포넌트입니다. <br><br>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src = resume/ju_1.PNG width = 500 height = 300/>  
이러한 큐, 리더, 디코더를 이용해 파일에서 데이터를 읽어 학습 데이터로 사용하기 위한 변환 작업을 하게 되면 이렇게 변환된 데이터를 읽어 들이는 전처리 작업 배치 처리를 다룰 수 있게 됩니다.
read_data 함수에서 csv 파일을 읽어 들여 파일 안에 있는 내용물들을 즉 필드 값들을 ‘,’ 기준으로 분류해 각기 형태에 맞는 변수로 read_data_batch 함수에 리턴 해줍니다. 이 함수가 리더와 디코더를 이용해 돌아갑니다.
read_data_batch 함수는 read_data 함수에 리턴 받을 변수를 선언해 값을 받은 다음 tf.train.batch 함수를 통해 정해둔 배치 값만큼의 배치 크기로 묶어 메인에 리턴 합니다. 즉 기존 입력 크기가 [x,y,z]였을 경우 배치처리를 통해 크기가 [batch_size,x,y,z]로 변환됩니다.<br><br> 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src = resume/ju_2.PNG width = 500 height = 300/>  
메인에선 배치처리까지 끝났으니 큐를 이용해 피딩 작업을 수행합니다. 큐, 큐 러너, 코디네이터, 멀티 쓰레드를 구현한다고 보면 됩니다.<br><br> 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src = resume/ju_3.PNG width = 500 height = 300/>  
결과 배치 사이즈 만큼 배치처리가 되어 결과가 출력됨을 볼 수 있습니다.
배치처리가 무엇인지 학습한 뒤로는 바로 작업에 들어갑니다.
이미지는 강아지 이미지들로만 준비되어 있으며 16 혹은 32 사이즈로 리사이즈 된 상태입니다.
이미지 하나당 본인의 X, Y 사이즈 값들이 있고 이미지가 흑백인지 컬러인지를 구분하는 RGB 값이 들어 있습니다.<br><br>

배치처리에 앞서 각 이미지 들의 필드 값을 csv에 저장하여 배치처리 작업을 할 수 있게 환경 마련해야 합니다.
csv는 총 2개로 작성하는데 하나는 배치처리 작업에 이용하기 위한 파일이며 하나는 각기 이미지와 필드 값들이 잘 작성되는지 확인을 위한 확인 여부 용으로 작성합니다.
2중 for 문을 돌리는데 OS 함수를 이용해 이미지 파일들이 들어가 있는 디렉터리를 중심으로 이미지 파일 수 만큼 반복문이 돌아가며 이미지 하나당 필드 값과 이미지 명이 CSV에 작성들 됩니다.

CSV 작성 작업이 끝나면 배치처리 작업을 수행합니다.


텐서플로가 돌아가면 각기의 CSV가 생성됨을 확인할 수 있습니다.

또한, 배치처리까지 됨을 확인할 수 있습니다.<br><br>


* **모델 구현**  
이미지 복원과 모자이크 제거 등의 작업에서 좋은 성능을 얻기 위해 CNN을 이용한 모델을 설명합니다. 우리의 모델은 16x16 사이즈의 컬러 이미지를 32x32 사이즈로 복원하는 것이 목표입니다.<br>
<br>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src = resume/cnn.PNG width = 600 height = 150/> <br><br>

* **Image High Resolution**

우리의 모델은 가로 16, 세로 16 그리고 3개의 채널을 갖는 이미지를 입력으로 받아 가로 32, 세로 32 그리고 3개의 채널을 갖는 이미지로 복원합니다. 먼저 이미지를 고해상도로 복원하기 위해서는 입력받은 소스 이미지를 충분한 크기로 확장해야 합니다. 이미지를 확장하는 방법에는 몇 가지 방법이 있고 우리는 Deconvolution이라는 방법을 선택했습니다. 이미지가 충분히 확장된 후에는 다시 (32, 32, 3) 사이즈로 압축 해야 됩니다. 압축 과정은 딥러닝 분야에서 큰 성공을 거둔 convolution 레이어를 거쳐 수행됩니다. 우리가 작성한 모델은 (16, 16, 3) -> (32, 32, 27) -> (32, 32, 9) -> (32, 32, 3) 순으로 확장과 압축을 진행하지만 더 좋은 하드웨어가 있다면 모델을 더 복잡하게 구성해도 될 것입니다.<br><br> 우리의 모델에서 특이한 점은 바로 maxpooling은 진행하지 않는다는 점인데 maxpooling은 데이터에서 특징을 추출하는 성공적인 방법이지만 우리가 목표로 하는 이미지 복원에서는 maxpooling을 하게 되면 이미지의 데이터가 생략되게 되고 이렇게 되면 정확한 복원을 할 수 없게 되기 때문입니다. 모델의 손실함수는 convolution 레이어를 거쳐 최종적으로 얻은 예측 이미지와 정답 이미지의 mean square error이고 optimizer는 Adam을 사용합니다.<br><br>

* **실험결과**
우리는 실험에 약 9000개의 학습 이미지와 1000개의 테스트 이미지를 사용하였습니다. Batch size는 64개이고 optimizer은 Adam을 사용합니다.
			
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <img src = resume/result.PNG width = 500 height = 150/> <br><br>

<그림 2>는 테스트 데이터의 타겟 이미지이고 <그림 3>은 모델이 예측한 이미지입니다. 아주 간단한 모델이지만 매우 좋은 결과를 낸다는 것을 확인할 수 있습니다.<br><br>


 이번 프로젝트는 육안으로 판별이 불가능한 16x16 사이즈의 픽셀 같은 화질이 뭉개진 사진을 조금이나마나 판별이 가능하게끔 32x32 사이즈로 리사이즈 하여 화질을 높이는 작업을 개발자인 사람이 아닌 기계가 다양하면서도 비슷한 케이스들로 학습을 하여 결과를 추출해 낼 수 있게끔 기계 학습시켰다. 이러한 기계 학습으로 지금까지 학습이나 테스트를 위해 써왔던 이미지들만이 아닌 처음 보는 이미지도 무리 없이 화질을 개선시키는 범용적으로 기계가 학습됨을 확인할 수 있었다.<br><br>
 이러한 경험을 바탕으로 더욱 다양하면서 분야를 가리지 않고 심도 있는 모델을 개발함으로써 교육적, 상업적, 의료적으로 접근할 수 있다고 생각했다. 예를 들어, 사람 얼굴이나 동물 얼굴, 사물이 찍혀있는 사진이나 GIF를 입력받아 인식 및 분류해 사진의 내용이 무엇인지 혹은 명칭을 알려주는 모델을 개발해 사람들이 평소 인터넷이나 TV를 통해 보게 되는 인물 혹은 사물 이름을 바로 알려줘 궁금증을 해결해 줄 수 있는 모델로 개발할 수 있다.<br><br>
 또한, 우리가 개발했던 위 모델을 훨씬 발전시키면 해상도가 뭉개진 CCTV 영상이나 사진의 화질을 최대로 높여 미제 사건으로 남았던 사건들을 해결해주거나 모자이크 처리되어있는 사진, 영상 그리고 모자이크와 같은 임의로 조작한 사진들을 수정함으로서 앞으로도 일어날 범죄를 하루 빨리 해결할 수 있게끔 사회에 공헌하는 모델을 개발해 낼 수 있다는 것이다. 이 모델을 활용하여 우리나라의 재범률은 평균적으로 20%대를 유지하는데 기존의 수사방식처럼 비슷한 범죄자를 일일이 형사들이 찾아가서 확인하는 것이 아닌 범죄자의 사진이나 인적 특징이 있다면 바로 비교하여 수사의 시간을 훨씬 단축시킬 수 있고 재범률을 낮추는 것도 가능하다고 생각된다.<br><br>
 그러나 우리가 개발한 모델은 원본 사진이 반드시 필요하다는 결함이 있다. 원래의 목적은 사진의 해상도를 높여 더욱 선명하게 볼 수 있는 모델을 구현하는 것이었다. 그러나 모델을 학습시키기 위해선 수많은 사진들이 필요했고, 원본 사진을 조작하여 모델을 학습시킬 수  밖에 없던 것이었다. 만약 이 모델을 더욱 개선시킬 수 있다면, 우리가 사용했던 강아지 사진 외에도 사람이나 사물 등 다양한 데이터 셋을 활용하여 더욱 범용성을 높여 갈 것이고, 원본 사진 없이 더욱 선명하게 볼 수 있는 모델을 개발하는 것이 목표이다.<br><br>
 우리는 화질 개선이라는 모델을 개발하였기 때문에 넓은 의미로 보자면 영상처리 관련 분야를 다뤄봤다고 볼 수 있다. 그렇기에 이 모델을 개선할 수 있다면 모자이크 제거나 오래된 CCTV 영상 화질 등을 개선할 수 있는 모델을 개발할 것이다. 이 분야는 현재 구글에서 개발 중인 단계로 우리 역시 이 분야에 더욱 개발에 매진하게 된다면 더욱 빨리 실용화 단계로 갈 수 있게 진척이 있을 것으로 기대된다.

 -----------

 [**[이력서 페이지로][go_resume]**]


 [go_resume]: https://github.com/HanBI24/Resume1/blob/main/README.md