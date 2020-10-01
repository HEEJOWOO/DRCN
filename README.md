# DRCN : https://arxiv.org/abs/1511.04491

#### <I studied while referring to various sites, but it is not enough. Thanks anytime for feedback.>
### <heejo5@naver.com>

Deeply-Recursive Convolutional Network for Image Super-Resolution
-----------------------------------------------------------------
* 새로운 Convolution연산을 수행하기 위해 필요한 추가적인 파라미터 없이 성능을 향상
* 추가적인 파라미터 없이 성능을 향상 시키기 위해서는 가중치를 공유하게 되는데 이 부분에 있어 Gradient Descent방법을 사용하면 Gradient Vanishing 또는 Exploding문제가 발생
* 이 문제를 해결하기 위해 첫 번쨍 방법으로 Recursive-Supervision과 두 번째 방법으로 Skip Connection 방법을 사용

기존 네트워크 문제점 지적
-----------------------
* Image Super Resolution에서 CNN의 Receptive field는 입력영상의 특정 영역을 의마하고 또한 맥락정보의 양을 결정
* Receptive Field를 안다면 영상을 복원하는데 많은 도움이 된다는 뜻인데 Deep Convolution Network에서는 성공적으로 Receptive field를 늘리는데 성공했고 성공하는데 사용한 방법 중 하나는 Network의 깊이를 늘리는 방법을 사용
* 하지만 Network 의 깊이를 늘리게 되면 생기는 문제가 Convolution 필터가 1*1 보다 크게되면 parameter수가 크게 증가하고, 또한 Pooling layer를 사용하기에는 Super Resolution에서는 정보 손실로 사용을 못하고 Weight Layer를 증가시키게 되면 또한 많은 파라미터로 인해 오버피팅과 비대해지는 문제가 발생
* 더 깊이 층을 쌓는 것에 대한 문제점을 제한된 크기에서 깊은 모델을 구현하자고 생각을 하였고, 따라서 재귀적으로 Receptive field를 확장하여 모델 크기가 늘어나지 않은 네트워크를 제안

DRCN Network -Embedding Network
-------------------------------
![image](https://user-images.githubusercontent.com/61686244/94774317-974b4f80-03f8-11eb-9cab-696c5b469a3b.png)
* 단계는 inference로 들어갈 수 있는 특징 맵으로 만드는 단계로 채널 확장 및 기본 특징 검출을 하며 다른 종류의 레이어들과 end-to-end로 학습을 진행
* 총 Embedding 단계는 2개의 Convolution으로 구성 돼 있으며 오른쪽에 보이는 식과 같이 우선 입력으로 보간된  x 가 들어오면 W-1  필터로 Convolution이 진행이 되고 후에 ReLu를 통과하고 출력이 다시 두번째 Convolution층으로 들어가 똑같이 W0와 Convolution을 진행한 뒤에 똑같이 ReLu를 통과하여 특징을 추출하게됨

DRCN Network -Inference Network
-------------------------------
![image](https://user-images.githubusercontent.com/61686244/94774456-ec876100-03f8-11eb-92c6-e723ecfd4f67.png)
* Inference단계로 재귀 방법을 통해 특징을 잡아내며 앞에서 파라미터의 추가없이 성능을 높이기 위해 컨볼루션이 같은 단일 재귀 계층으로 이루어져 있음
* Inference Network를 펴보게 되면 다음 그림의 Network 재귀 구조로 구성이 돼 있으며 이 재귀 층은 모두는 똑같은 Convolution와 ReLU로 구성이  있어 다음 식과 같이 표현을 할 수 있음
* 재귀 층의 D개의 Convolution은 파라미터를 공유하고 있어 파라미터의 추가 없이 성능을 높일 수 있었고 재귀 방법을 통해 Receptive Field의 크기가 넓어지게됨

DRCN Network -Reconstruction Network
------------------------------------
* 이지미를 흑백 또는 칼라 채널로 재구성하는 단계
* 재귀로 구성된 Inference Network가 입력으로 들어오고 한번더 Wd+2 필터로 Convolution과 ReLU가 진행이됨
* Inference Network에서 같은 weight, bias를 사용하기에 Gradient Vanishing, Exploding문제가 발생하였다고함
* 또한 입력된 LR img 와 HR Img 사이에는 매우 유사한 정보들을 가지고 있는데, 이렇게 재귀를 통해 계속 뽑아 내다보면 유사한 정보들이 사라지게됨

Recursive-Supervision & Skip Connection
---------------------------------------
![image](https://user-images.githubusercontent.com/61686244/94774908-bd252400-03f9-11eb-9503-256a343ab765.png)
* 해결방법을 추가한 최종 네트워크 구조 
* Recursive-Supervision을 추가하여 최종 출력값을 재귀 레이어의 중간 예측값들의 평균을 사용하여 해결
* 이 방법을 사용하면 서로 다른 예측 손실로부터 역전파 된 모든 Gradient를 합산하면 하나의 튀는 Gradient의 가중치가 큰 영향을 미치지 못하므로 Gradient 스무딩 효과가 나타나 Gradient Vanishing, Exploding을 방지
* 재귀층 또한 너무 깊어지면 얻는 이득에 한계가 존재
* 두 번째 문제를 해결하기위해 skip-Connection을 추가하여 재귀 레이어의 중간 예측값 과 input값 모두 뒤를 건너 뛰고 바로 Reconstruction에 입력을 하여 문제 2를 해결 하였고, Skip connection의 사용으로 input img의 신호를 재귀과정 중에도 저장할 수 있게 만듦


Loss Function
-------------
![image](https://user-images.githubusercontent.com/61686244/94775269-7683f980-03fa-11eb-9210-2edeaf12e017.png)

Recursion versus Performance
----------------------------
![image](https://user-images.githubusercontent.com/61686244/94775347-9d423000-03fa-11eb-91b0-8160783bb074.png)
* 4개의 동일한 모델을 recursion 횟수만 달리하여 학습시킨 결과로 recursion 횟수가 많을 수록 PSNR이 높게 나왔음
* Recursion을 16개 층 보다 더 깊게 실험을 해보았으나 recursion 재귀 단계를 아무리 깊게 한다고 좋은 성능이 나오지 않았고 16개가 가장 최적의 모델이였다고함
* 16 recusrion모델은 최종적으로 20개의 conv를 지나간다고함

Ensemble effect
---------------
![image](https://user-images.githubusercontent.com/61686244/94775439-c1057600-03fa-11eb-90b2-57f0a109391b.png)
* 각각의 재귀 과정 중 중간 값을 빼오는 Recursive supervision의 효과에 대해서 말해주는 그래프 
* 재귀 중간값들을 구해 더해서 예측한값이 Recursive supervision을 사용하지 않은 값 보다 더 효과가 좋은걸 증명

Average PSNR/SSIMs
------------------
![image](https://user-images.githubusercontent.com/61686244/94775555-02962100-03fb-11eb-9215-ba5837aab1aa.png)

Super Resolution Resluts
------------------------
![image](https://user-images.githubusercontent.com/61686244/94775598-1b063b80-03fb-11eb-8d42-00530eb620bb.png)


