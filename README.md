# ShowmetheMusic

보아즈 Adv proejct 쇼미더뮤직팀 '텍스트 감정추출을 통한 노래 추천 시스템' 프로젝트 깃허브입니다.

## 주제소개
- 텍스트 감정 추출을 통한 노래 추천 서비스

## 선정배경
- 최근 코로나 이후 라이브 음악산업의 수요는 줄어들고 유튜브 뮤직, 바이브같은 음원 스트리밍 산업의 수요는 더욱 급격화
- 음원 스트리밍 산업의 증가로 원하는 노래를 더 쉽고 빠르게 찾아 들을 수 있게 됨. 하지만 기존 노래 추천 방식은 평소 듣는 노래와 비슷한 노래로 추천되는 방식이기에, 노래들을 개인의 감정과 심리, 상황에 맞춰 추천을 하고자 함
- 음원 스트리밍 산업 수요의 증가와 하루를 정리하면서 노래를 통해 공감받고 위로받고자 하는 심리가 결합되어 일길, 리뷰, 정보 등 감정이 반영된 다양한 글들이 SNS와 플랫폼에서 공유되고 있음
- 위와 같은 상황을 기반으로 일상의 감정과 어울리는 노래를 사람들에게 추천해주고 싶다는 생각을 하게 되어, NLP 기법과 개인의 취향을 반영할 수 있는 개인화 기법을 결합해 나만의 노래를 추천받을 수 있도록 구성함.

## 모델링 아키텍처
![image](https://user-images.githubusercontent.com/57586314/152271321-2c2d2c77-4169-4c4b-97f2-88700c147933.png)

- 텍스트 입력 : 사용자에게 하루를 정리하는 다이어리 형식의 글이나, 감정을 나타내는 글을 입력받음
- NLP : KcBERT model을 사용해 감정(기쁨, 분노, 슬픔)을 분류
- 추천시스템 : Sentence Transformers와 감정-노래 Matrix를 기반으로 NLP에서 나온 감정과 유사한 노래 추천
- 재추천 : 사용자의 선호도와 개인의 성향을 반영하여 유사도를 다시 구한 후 재추천

## NLP 모델링

### - 최종 데이터, 감정 label
- 최종 데이터셋 : AI Hub의 감성대화 말뭉치 데이터, 한국어 단발성 대화 데이터
- 감정 label : 총 4개의 label로 재 라벨링 (0: 기쁨,행복/ 1:분노,불안,당황,놀람/ 2. 슬픔,상처/ 3: 중립)

### - 감정 분류
![image](https://user-images.githubusercontent.com/77534419/152346947-cda619b1-651d-49a3-b7aa-af59a675fc63.png)

### - 문장 분리(매우 긴 문장 처리 방법)
![image](https://user-images.githubusercontent.com/57586314/152272661-24e3cf5b-9095-4bcd-8899-905ccc8c3feb.png)

- 일반적인 감정 분류의 문제점 : 학습 시킨 데이터는 1~2문장으로 매우 짧은 문장이며, 사용자가 매우 긴 문장을 입력했을 때 글의 감정을 추출하기에는 매우 어려움
- 위와 같은 문제점을 해결하고자 3가지 방법으로 실험 진행
- 1) 생성요약 : Kobart를 활용해서 진행. 뉴스 기사 책과 같은 공식적인 글은 잘 요약하나 댓글, sns, 일기와 같은 비공식적인 글을 요약하지 못한다는 문제점 존재
- 2) 추출요약 : Textrank를 활용해서 진행. 첫 문장을 핵심 문장으로 뽑는 경우가 많고 빈도가 높은 불필요한 단어를 추출하는 경우가 많아, 노이즈가 많이 섞여 있고 여러 감정이 담겨있을 수 있는 다이어리에 적합하지 않다고 판단
- 3) 다중 문장을 여러 문장으로 분리하는 문장 분리기 : kss 라이브러리를 활용해서 진행. 다중 문장을 단일 문장으로 분리해 기존 훈련된 모델을 통해 중립문장을 제외하고 나머지 문장에서 나온 감정의 softmax값을 구함. 각각의 값들을 다시 합친 후 확률의 형태인 softmax로 변환하여 추천시스템 모델로 전달



## 추천시스템 모델링

### - 1차 추천

![추천1](https://user-images.githubusercontent.com/76245088/152142565-b8ea8bae-3624-40f1-81d8-743d7b3345dd.jpg)

- 사용자에게 감정과 가사의 중요도를 INPUT으로 받아 감정과 가사에 가중치를 반영하여 글과 유사한 노래순으로 나열
- 사용자에게 감정과 유사한, 감정과 반대되는 노래를 제공
- 사용자의 선택에 따라 데이터셋 고정 

### - 2차 추천

![추천2](https://user-images.githubusercontent.com/76245088/152143461-6ff44945-f263-48fc-a0d8-0c41073d799f.jpg)

- 1차 추천에서 사용자가 선택한 데이터셋에서 감정과 가사의 중요도를 반영하여 텍스트와 가장 유사한 노래 5곡을 추천
- 사용자에게 5곡에 대한 5점 척도의 점수를 받음 
- 사용자가 가장 높은 점수를 준 노래의 8가지 감정값을 GET
- 해당 감정값과 노래 데이터 사이의 유사도를 계산
- 계산된 유사도를 기반으로 노래 데이터셋을 재정렬하고 유사도를 기반으로 재추천 

### - 노래 추천 프로세스 

![추천3](https://user-images.githubusercontent.com/76245088/152144213-b6b02bae-53d2-4744-b2d9-6dceb0575bbb.jpg)

- NLP모델을 활용해 기쁨, 분노, 슬픔의 세가지 감정 중 사용자의 텍스트가 어디에 해당하는지 분류
- 사용자에게 노래를 추천받을 때, 감정과 가사 중 어떤것을 더 중요시 여기는지 5점척도로 입력을 받음
- 사용자가 입력한 중요도를 함께 고려하여 텍스트와 유사도를 기반으로 정렬함
- 텍스트의 감정과 유사한 감정을 가지는 노래, 반대되는 감정을 가지는 노래를 추천하여 사용자의 선택에 따라 데이터 고정
- 유사도 top 5인 노래를 재추천하고, 사용자에게 추천에 대한 평가를 5점 척도로 받아 가장 높은 점수의 노래를 기반으로 다른 노래와의 유사도를 계산하여 5곡씩 반복하여 추천

## 웹
- NLP와 추천시스템의 모델을 배포하기 위해 웹개발 진행 중
- HTML, CSS, JavaScript를 이용한 프론트엔드 구상
    - 사용자의 편의와 모델의 기능을 고려한 UX/UI 디자인
    - 사용자는 일기를 작성하고 일기의 감정을 확인 가능
    - 사용자의 노래 감상 취향을 반영
- Django, Flask를 통해 모델 서빙 진행 중

## 결론 및 제언
