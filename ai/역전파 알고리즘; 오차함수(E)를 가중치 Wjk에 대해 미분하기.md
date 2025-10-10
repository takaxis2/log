### 📘 수식의 의미

우리가 최적화하려는 오차 함수는 다음과 같아요:

$$E = (t_k - o_k)^2$$
- t_k: 목표값 (target)
    
- o_k: 출력값 (output)
    
- w_{jk}: 은닉층 j에서 출력층 k로 가는 가중치
    

### 🧮 미분 과정 해설

1. **오차 함수의 미분 시작** $$ \frac{\partial E}{\partial w_{jk}} = \frac{\partial}{\partial w_{jk}} (t_k - o_k)^2 $$ → 출력값 oko_k는 가중치 w_{jk}에 영향을 받기 때문에, 연쇄 법칙(chain rule)을 사용합니다.
    
2. **연쇄 법칙 적용** $$ \frac{\partial E}{\partial w_{jk}} = \frac{\partial E}{\partial o_k} \cdot \frac{\partial o_k}{\partial w_{jk}} $$ → 오차를 출력값에 대해 미분하고, 출력값을 가중치에 대해 미분한 거예요.
    
3. **첫 번째 미분 계산** $$ \frac{\partial E}{\partial o_k} = -2(t_k - o_k) $$ → 단순한 제곱 함수의 미분입니다.
    
4. **두 번째 미분 계산** 출력값 oko_k는 시그모이드 함수로 표현됩니다: $$ o_k = \sigma\left(\sum_j w_{jk} \cdot o_j\right) $$
    
    시그모이드 함수의 미분은 다음과 같아요: $$ \frac{d}{dx} \sigma(x) = \sigma(x)(1 - \sigma(x)) $$
    
    따라서, $$ \frac{\partial o_k}{\partial w_{jk}} = o_k(1 - o_k) \cdot o_j $$ → 여기서 o_j는 은닉층 j의 출력값입니다.
    
5. **최종 미분 결과** 모든 걸 합치면: $$ \frac{\partial E}{\partial w_{jk}} = -2(t_k - o_k) \cdot o_k(1 - o_k) \cdot o_j $$
    
    또는 학습률 alpha를 곱해서 가중치 업데이트 식으로 쓰면: $$ \Delta w_{jk} = \alpha \cdot (t_k - o_k) \cdot o_k(1 - o_k) \cdot o_j $$
    

### 🔍 요약

- 오차를 줄이기 위해 가중치를 얼마나 바꿔야 하는지를 계산하는 과정입니다.
    
- 시그모이드 함수의 미분이 간단해서 계산이 쉬워요.
    
- 이 수식은 딥러닝에서 **가중치 업데이트의 핵심 공식**이에요.