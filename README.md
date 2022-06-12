# ETF Scoring Model
비정형 투자 전략  


비정형 데이터 ETF 상품 투자 전략 설계를 위한 ETF 상품 Scoring 모델입니다.  
빅카인즈에서 크롤링으로 수집된 기사제목들을 CNN Encoder를 활용하여 일자별로 Encoding 후,  
1차분된 상품 가격 데이터와 결합하여 미래 가격 상승 가능성을 예측합니다.  


![모델 이미지](https://user-images.githubusercontent.com/32697109/173226742-80bb065e-cd22-43e9-8cd7-ed0b4ece406d.png)


사용 데이터:  


ETF 상품별 가격 데이터 + 빅카인즈 뉴스기사 제목 + 전문가 분석 리포트  
Train data 기간: 2010-12 ~ 2018-12  
Test data 기간: 2019-1 ~ 2021-6  


OKT parser를 사용하여 Text tokenizing후 Word2Vec 알고리즘을 사용하여 Initial word embedding하여 Text Data Preprocessing 하였습니다.  
Price data는 1차분하여 비정상성 제거 후 사용하였습니다.  


![ACF](https://user-images.githubusercontent.com/32697109/173226778-c7bdb0a2-e824-47a9-b1b3-0114d35c7581.png) 


CNN Encoder


https://arxiv.org/abs/1408.5882

CNN Encoder의 경우, Kim, Y의 모델을 사용하였습니다.  
이 모델의 경우 긴 Text를 사용하며, LSTM을 사용 할 경우 Sequential 모델의 특성인 Gradient explosion/vanishing 문제,  
그리고 너무 긴 training time을 피할 수 없다 생각했습니다.  BERT의 경우도, Memory 이슈가 있어 긴 text를 모두 읽지는 못하였습니다.  
이런 긴 text를 빠르게 처리 할 수 있는 CNN encoder가 적합하다 생각하였습니다.  
(하지만, CNN 모델의 경우 하루의 모든 Text를 3-dimensional array로 너무 간단하게 summarize하여 information loss가 심한 것으로 보임, 추후에 다른 인코더 사용해 볼 것)  



위 encoder에서 process된 text는 price데이터와 결합되어 LSTM 모델에 입력되고 최종 hidden layer가 FC layer를 통과하여 score를 내보내게 됩니다.



결과:


![훈련결과](https://user-images.githubusercontent.com/32697109/173227146-19601422-5e62-461c-bbc1-c8cc185efcd4.png)


Training error는 하락하고 Testing error는 상승함으로 Overfitting되어 가고 있는 것이 보입니다.  
하지만 Accuracy와 Precision이 지속적으로 상승하는 것을 확인 할 수 있으며, False Negative를 무시한다면 좋은 투자 전략을 만들 수 있을 것으로 보입니다.


![훈련결과2](https://user-images.githubusercontent.com/32697109/173227149-168cb177-5ebe-4d2a-ba0b-10a23ac822ce.png)


Threshold를 증가하여 예측할때 마다 예측 Precision이 증가하는것을 볼 수 있습니다.  
특히, .5 이후 강한 upward linear pattern을 확인 할 수 있습니다.  
.5 이상의 점수를 받은 경우, 투자를 고려하는 전략을 선택할 수 있을 만하며, 큰 점수일 경우 큰 confidence로 볼 수도 있습니다.  
이러한 우상향 특성은 좋은 투자 전략으로 사용될 수 있을 것으로 보입니다.   
.7 이상 점수를 받는 경우에 precision이 급락하는걸 볼 수 있는데, 이건 .7 이상 점수를 받은 경우가 드물어서일 가능성이 클 것으로 보이나, 주의가 필요합니다.  

![성능평가](https://user-images.githubusercontent.com/32697109/173227150-6d8b5e80-7606-4d70-adc2-4c6b1dc29bdd.png)



