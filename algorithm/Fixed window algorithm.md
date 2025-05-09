Fixed Window 알고리즘은 시간을 고정된 간격(윈도우)으로 나누어 각 윈도우마다 요청 수를 제한하는 간단한 속도 제한(rate limiting) 방식입니다.

## 작동 원리

1. 시간을 고정된 길이의 윈도우로 나눕니다 (예: 1분 단위).
2. 각 윈도우마다 허용되는 최대 요청 수를 설정합니다.
3. 요청이 들어올 때마다 해당 윈도우의 카운터를 증가시킵니다.
4. 카운터가 최대 허용치에 도달하면 추가 요청을 거부합니다.
5. 새 윈도우가 시작되면 카운터를 초기화합니다.

## 장점

1. 구현이 간단하고 이해하기 쉽습니다.
2. 메모리 사용량이 적어 효율적입니다.
3. Redis와 같은 분산 환경에서 사용하기 적합합니다.

## 단점

1. 윈도우 경계에서 트래픽 급증이 발생할 수 있습니다.
2. 요청 유형이나 우선순위를 구분하지 않습니다.
3. 긴 윈도우 사용 시 사용자 경험이 불균형할 수 있습니다.

Fixed Window 알고리즘은 간단하고 효율적이지만, 윈도우 경계에서의 트래픽 급증 문제와 유연성 부족이 주요 단점입니다. 그럼에도 불구하고 API 속도 제한, 사용자 인증 프로세스, IoT 기기 등 리소스가 제한된 환경에서 널리 사용됩니다.