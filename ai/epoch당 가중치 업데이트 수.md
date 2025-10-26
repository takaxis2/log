
``` python
history=model.fit(X_train, y_train, epochs=50, batch_size=5, validation_split=0.2, callbacks=[tensorboard_callback])
```
이 코드에서 **역전파(Backpropagation)**는 **50번보다 훨씬 많이** 일어납니다.

## 역전파 횟수 결정 원리

역전파는 한 번의 **가중치 갱신(Weight Update)**을 위해 필요하며, 가중치 갱신은 $\text{Epoch}$당 $\text{Iteration}$ 수만큼 발생합니다.

### 1. Epoch (에포크): 전체 데이터 학습 횟수

- `epochs=50`은 전체 훈련 데이터셋($X_{\text{train}}$)을 **총 50번** 반복하여 학습하겠다는 의미입니다.
    

### 2. Batch Size (배치 크기): 한 번에 처리하는 데이터 양

- `batch_size=5`는 가중치를 한 번 갱신하기 위해 훈련 데이터에서 **5개의 샘플**을 사용하겠다는 의미입니다.
    
- **역전파 및 가중치 갱신은 배치당 한 번** 일어납니다.
    

### 3. Iteration (반복): 역전파 발생 횟수

역전파가 발생하는 횟수, 즉 **가중치 갱신 횟수**는 $\text{Iteration}$ 수와 같습니다.

$$\text{총 Iteration 수} = \text{Epoch 수} \times \text{Epoch당 Iteration 수}$$

$$\text{Epoch당 Iteration 수} = \lceil \frac{\text{전체 훈련 데이터 샘플 수}}{\text{Batch Size}} \rceil$$

---

## 예시 계산

훈련 데이터($X_{\text{train}}$)의 샘플 수가 **100개**라고 가정하고 계산해 보겠습니다.

1. **전체 훈련 데이터 수:** 100개
    
2. **Batch Size:** 5
    
3. **Epochs:** 50
    

- Epoch당 Iteration 수: $\frac{100 \text{개}}{5 \text{개}} = 20$번
    
    (전체 데이터를 1회 학습하는 동안 20번의 배치 처리 및 20번의 가중치 갱신이 일어남)
    
- **총 역전파 횟수 (총 Iteration 수):** $50 \text{ (Epoch)} \times 20 \text{ (Iteration/Epoch)} = \mathbf{1,000}\text{번}$
    

따라서, **역전파는 총 1,000번** 일어나게 됩니다 (데이터 수가 100개일 때). 실제 횟수는 $X_{\text{train}}$의 샘플 수에 따라 달라집니다.