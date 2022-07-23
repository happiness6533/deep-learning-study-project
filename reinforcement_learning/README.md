## reinforcement learning: 표현(행동) for best reward in any state

- finite concrete state, finite concrete action
	- policy-iteration
	- value-iteration

- finite ambiguous state, finite concrete action
	- monte carlo
	- sarsa
	- q learning

- infinite ambiguous state, finite concrete action
	- reinforce
	- deep sarsa
	- dqn
	- a2c
	- a3c
	- td3
	- ddpg
	- dddqn
	- ppo
	- adversary-dqn
	- behavioral-cloning
	- sac

- infinite ambiguous state, infinite ambiguous action
	- airy

- 강화학습은 mdp를 풀고 상태, 행동, 결과의 상관관계를 추론한다

1. 주어진 환경을 mdp로 재구성
	- mdp 구성요소
		- 상태
			- 확률변수 S(t) = s
			- 유한 / 무한 / 추상성
		- 행동
			- 확률변수 A(t) = a
			- 유한 / 무한 / 추상성
		- 보상
			- 확률변수 R(s, a) = E[R(t + 1) | S(t) = s, A(t) = a]

	- mdp 하이퍼 파라미터
		- 상태변환확률: 행동에 의해 어떤 상태에서 다른 상태로 변화할 확률들의 집합
		- 감가율: 미래의 보상을 신뢰하는 비율

	- mdp 최적화 대상
		- 확률변수 G~pi(t)
			- 타임 t에서, 현재 정책에 따른, 감가율을 고려한, 미래에 얻을 수 있는 모든 보상의 합
			  ```
			    R(t + 1) + gamma * R(t + 2) + gamma^2 * R(t + 3) + ...
			  ```
			- 예를 들어 5번의 행동으로 에피소드가 종료된 경우
			  ```		  
			    - G(5) = R(6)
			    - G(4) = R(5) + gamma * R(6)
			    - G(3) = R(4) + gamma * R(5) + gamma^2 * R(6)
			    - G(2) = R(3) + gamma * R(4) + gamma^2 + R(5) + gamma^3 * R(6)
			    - G(1) = R(2) + gamma * R(3) + gamma^2 * R(4) + gamma^3 * R(5) + gamma^4 * R(6)
			  ```

		- 확률변수 V~pi(S) = E[G~pi(t)]
			- 타임 t에서, 현재 정책에 따른, 감가율을 고려한, 미래에 얻을 수 있는 모든 보상의 합의 기대값

		- 값 v~pi(s)
			- 타임 t, 상태 s에서, 현재 정책에 따른, 감가율을 고려한, 미래에 얻을 수 있는 모든 보상의 합의 기대값
			  ```
			  v~pi(s) = E~pi[G(t) | S(t) = s]
			  v~pi(s) = E~pi[R(t + 1) + gamma * R(t + 2) + gamma^2 * R(t + 3) + ... | S(t) = s]
			  v~pi(s) = E~pi[R(t + 1) + gamma * G~pi(t + 1) | S(t) = s]
			  v~pi(s) = E~pi[R(t + 1) | S(t) = s] + gamma * E[G~pi(t + 1) | S(t) = s]
			  v~pi(s) = E~pi[R(t + 1) | S(t) = s] + gamma * E[G~pi(t + 1)]
			  v~pi(s) = E~pi[R(t + 1) | S(t) = s] + gamma * v~pi(S(t + 1))
			  v~pi(s) = E~pi[R(t + 1) + gamma * v~pi(S(t + 1)) | S(t) = s] = 가치함수에 대한 벨만 기대 방정식(계산 불가능)
			  ```

			- 가치함수에 대한 벨만 기대 방정식을 계산 가능한 형태로 바꾸고 다이나믹 프로그래밍으로 풀이 = policy iteration
			  ```
			  v~pi(s) = a에 대한 시그마(pi(a | s) * (r + gamma * s'에 대한 시그마(상태변환확률 * v~pi(s'))))
              => 상태 변환 확률을 1이라 가정
			  v~pi(s) = a에 대한 시그마(pi(a | s) * (r + gamma * v~pi(s')))
			  => 다이나믹 프로그래밍(divide and conquer + iterative)
			  v~pi(s) => v~k+1th-pi(s) = a에 대한 시그마(kth-pi(a | s) * (r + gamma * v~kth-pi(s')))
              ```

			- 가치함수에 대한 벨만 기대 방정식을 최적화하고 다이나믹 프로그래밍으로 풀이 = value iteration
			  ```			    
			  벨만 방정식 최적화의 해 optimal v~pi(s) = max v~pi(s) in {pi}
			  v~pi(s) = max E~pi[R(t + 1) + gamma * v~pi(S(t + 1)) | S(t) = s] in {pi} = 가치함수에 대한 벨만 최적 방정식(계산 불가능)
			  
              계산 가능한 형태로 고치면 아래와 같다
              v~pi(s) = max a에 대한 시그마(pi(a | s) * (r + gamma * v~pi(s')))
			  ```

			- 다음 상태까지 고려하지 않고 현재 상태에서 가능한 행동만을 고려해서 다음 행동을 결정하는 경우 큐함수를 고려한다
			- 동일한 논리를 큐함수에 대해서도 전개할 수 있다
			  ```									  
				- v파이(s) = a에 대한 시그마(pi(a | s) * q파이(s, a))
				- 이를 벨만 방정식에 대입해서 재서술하면 다음과 같다
				- a에 대한 시그마(pi(a | s) * q파이(s, a))
				= E파이[R(t + 1) + gamma * a에 대한 시그마(pi(a | s) * q파이(S(t + 1), A(t + 1))) | S(t) = s]
				- a를 고정하면 아래와 같다
				- q파이(s, a) = E파이[R(t + 1) + gamma * q파이(S(t + 1), A(t + 1))) | S(t) = s, A(t) = a]	
     								
				큐함수에 대한 벨만 최적화 방정식
				optimal q(s, a) = E[R(t+1) + gamma * max q(S(t+1), a') | S(t) = s, A(t) = a] in many pi
	          ```

		- 정책 pi(a|s)

2. 에이전트의 value, policy 정의
	- 어떤 상태에서 어떤 행동을 할 확률( p(A(t) = a | S(t) = s) = pi(a | s) )에 대한 함수 pmf/pdf 를 policy라고 한다
	- 동적 프로그래밍(가치, 벨류 이터레이션) => 상태 차원의 저주 + 환경 모델의 불완정성 문제(상태 변환 확률과 보상은 현실에서 사전에 완벽하게 알 수 없다)
	- 상태 s가 유한한 경우 v(s), q(s, a)를 정의하는 방법
		- 모든 상태 s가 유한한 경우, 모든 v(s)를 직접 계산할 수 있다
		- gt의 정의로부터 아래의 식을 유도한다
		- v(s) = 시그마(a 확률 * (reward + 감가율 * v(s'))
		- 위의 수식에서, 현재 상태 s와 어떤 행동 a에 대해 얻어지는 상태 s'의 관계는
		- s에서 a를 해서 s'이 됨
		- 과 같으므로, 우리는 v(s')을 s와 a에 대한 함수로도 생각할 수 있다. 이러한 함수를 q 함수라고 한다면 다음이 성립한다
		- v(s') = q(s, a)
		- 따라서 상태 s는 가능한 행동 a들에 대해 여러 q(s, a)를 가진다
		- 따라서 위의 식을 다시 쓰면 아래와 같다
		- v(s) = 시그마(행동 확률 * (reward + 감가율 * q(s, a))
		- 여기까지 따라왔다면, 위의 수식을 조금 더 일반화시키면 아래와 같음을 이해할 수 있다
		- q(s, a) = 시그마(행동 확률 * (reward + 감가율 * q(s', a'))
		- 결론적으로 아래의 두 수식을 활용한다
		- v(s) = 시그마(행동 확률 * (reward + 감가율 * v(s')): 정책 이터레이션, 가치 이터레이션
		- q(s, a) = 시그마(행동 확률 * (reward + 감가율 * q(s', a')): sarsa, q러닝, deepSarsa, dqn

	- 상태 s가 무한한 경우 v(s)를 정의하는 방법
		- 모든 상태 s가 무한한 경우: 모든 v(s)를 직접 계산할 수 없다: mc(몬테카를로) 방법을 사용한다
		- 에피소드1: 지나온 경로를 기록하고, 경로에 따른 Gt를 각 스테이트에 기록하자
		- 에피소드2: 지나온 경로를 기록하고, 경로에 따른 Gt를 각 스테이트에 기록하자
		- 에피소드 반복
		- 이제 기록된 모든 스테이트에 따른 G(s) 기록들의 평균을 구하자 = v(s)
		- 이 평균은 지금 정책에 따라 얻어지는 참 가치함수에 수렴한다
		- 그런데 이렇게 하려면, 에피소드 1000번을 미리 시행하고 기록한 후 업데이트를 해야 하며, 이건 메모리가 많이 든다
		- 따라서 테크닉을 사용, 임의의 n번째 에피소드가 종료되면 즉시 v(s)를 업데이트 해도
		- 1000번의 에피소드를 마치고 v(s)를 업데이트한 결과와 같도록 수식을 구성한다
		- 업데이트할 v(s) = 기존의 v(s) + 1/n * (현재 G(s) - 기존의 v(s))
			- 1/n을 스텝사이즈로 보고 알파로 놓자
			- G(s)는 타겟, V(s)는 현재, 알파 = 러닝레이트, gradient_ascend와 유사한 방식!
		- 이 방법을 쓰려면 적어도 에피소드가 1번은 끝나야만 다음 에피소드 진행 도중에 이전 에피소드를 활용해서 업데이트 가능함

	- 상태 s가 무한한 경우 q(s, a)를 정의하는 방법
		- 모든 상태 s가 무한한 경우 : 모든 v(s)를 직접 계산할 수 없다 : td(temporal difference) 방법을 사용한다
		- full 에피소드 없이, 액션에 일어날 때마다 v(s) 실시간 업데이트
		- 업데이트할 v(s) = 기존의 v(s) + learning_rate * (현재 받는 리워드(에피소드 필요ㄴㄴ) + r * v(s')(에피소드 필요ㄴㄴ) - 기존의 v(s))
		- 반복해서 현재 정책에 따른 참 가치함수값을 얻고, 정책을 업데이트하고, 이를 반복한다

3. value, policy 최적화
	1. 유한한 상태의 세상에서 value and policy 최적화
		- policy iteration
			- 모든 상태 s가 유한한 경우
			- 모든 상태의 v(s)를 적당한 값으로 초기화

			- 현재 정책에 따른 모든 상태의 real(semi도 가능) v(s)를 구하기 위해서 v(s) 업데이트를 여러번 반복한다
			- 얻어진 real(semi도 가능) v(s)에서 그리디를 적용해서 정책 1회 업데이트

			- 업데이트 된 새로운 정책에 따른 모든 상태의 real(semi도 가능) v(s)를 구하기 위해서 v(s) 업데이트를 여러번 반복한다
			- 얻어진 real(semi도 가능) v(s)에서 그리디를 적용해서 정책 1회 업데이트

			- 위의 과정을 반복하면, optimal 정책이 얻어진다

		- value iteration
			- 모든 상태 s가 유한한 경우
			- 모든 상태의 v(s)를 적당한 값으로 초기화

			- 현재 정책에 따른 모든 상태의 real(semi도 가능) v(s)를 구하기 위해서 v(s) 업데이트를 여러번 반복한다
			- v(s)를 업데이트 할 때, 평균이 아니라 max v(s')을 사용하면 그리디를 적용하는 효과가 있기 때문에, 정책을 따로 업데이트 할 필요가 없다

			- 위의 과정을 반복하면 v(s)가 optimal 정책을 내제하게 된다
	2. 모호한 상태의 세상에서 value and policy 최적화(상태 변환 확률을 알 수 없다)
		- monte carlo
		- sarsa
			- q(s, a) = q(s, a) + 알파 (r + gamma * q(s', a') - q(s, a)) = 벨만 기대 방정식
			- 액션으로 업데이트 하는 td 방법상 q함수를 이용해야 계산이 가능하다. v를 이용하려면 상태변환확률을 알고 max를 취해야 하는데 이를 알 수 없기 때문
			- td로 정책을 평가하고 + 입실론 그리디 정책 발전으로 발전시키며 + 가치 이터레이션
			- 살사는 온폴리지 시간차 제어, 즉 자신이 행동하는 대로 학습하는 시간차 제어임
			- 입실론 그리디를 쓴 경우 다음 행동 a'이 분명히 안좋은 결과를 만들 것임에도 e의 확률로 a'을 수행하고, 이는 a에 대한 q를 낮게 만들어서 갇힘을 만듬
			- 그러나 탐험 e는 꼭 필요
			- 오프폴리시 td 제어인 큐러닝을 도입한다

		- q learning
			- q(s, a) = q(s, a) + 알파 (r + gamma * max q(s', a') - q(s, a)) = 벨만 최적 방정식
			- 실제로 a'은 입실론으로 하지만, 업데이트할땐 무조건 걍 max로 업데이트 한다

	3. 무한한 상태의 세상에서 value and policy 최적화
		- reinforce = monte carlo + deep learning
			- 정책 = 네트워크 = 정책망 = pi~세타(a | s), where maximize v~pi(s)
				- v(s)를 최대로 만드는 정책 파이를 얻자 = v(s)를 세타로 표현된 정책 파이를 이용해서 식을 쓰고 gradient ascent하자
				- 폴리시 그래디언트
				    ```
					gradient v(s) = 시그마(상태 s의 확률) * 시그마(gradient 파이(a|s) * q(s, a)))
					gradient v(s) = 시그마(상태 s의 확률) * 시그마(파이(a|s) * gradient log 파이(a|s) * q(s, a)))
					gradient v(s) = E~파이[gradient log 파이(a|s) * q(s, a))] -> 샘플링으로 대체하자
					gradient v(s) = gradient log 파이(a|s) * q(s, a)) -> 정책 기반이라서 상태에서 큐 없이 바로 행동을 뽑을거라 q가 없다 -> Gt로 대체하자
					gradient v(s) = gradient log 파이(a|s) * Gt
					gradient v(s) = gradient (log 파이(a|s) * Gt)
					=> loss = - log 파이(a|s) * Gt => 이는 Gt가 일종의 y, 파이가 일종의 a가 되는 지도학습의 크로스 엔트로피로 해석할 수 있다
					```
			- 정책 업데이트
			- 입실론을 쓰지 않는다 : 모든 행동이 확률적으로 나오기 때문에, 확률뷴포에 따르는 선택을 한다 : 따라서 가만히 있으면 -1점을 주어야 움직임
			- 정책 기반 강화학습 = 가치 없이 걍 상태 보고 바로 행동을 얻는 강화학습

		- deep sarsa = sarsa + deep learning
			- 큐함수 = 네트워크 = 가치망
			  ```
			  loss = (r + q(s', a') - q(s, a))^2
			  loss = 평균제곱오차
			  ```
			- 큐함수 업데이트
			- td : 다음 행동을 미리 그리디하게 선택하고 업데이트
			- 가치 기반 강화학습 = 가치를 기반으로 업데이트 하면서 학습

		- dqn = q learning + deep learning
			- q러닝 알고리즘 : 큐함수 업데이트 + 정책 업데이트
			- 큐함수 = 네트워크가 근사하는 함수
			    ```
				loss = (r + max q(s', a') - q(s, a))^2
				loss = 평균제곱오차
				```
			- 큐함수 업데이트
			- td : sars'을 리플레이 메모리에서 랜덤하게 추출, 다음 q를 maxq 선택 후 업데이트, 리플레이 메모리가 꽉 차면 점점 더 좋은 메모리르 채워나간다
			- 리플레이 메모리에서 추출한 여러 샘플로 업데이트를 하면 신경망 업데이트가 당연히 안정적이다(미니배치)
			- 리플레이 메모리는 과거이므로 오프폴리시, 즉 큐러닝 계열이다
			- 타겟 네트워크와 업데이트 네트워크를 분리한다

		- a2c = reinforce 발전
			- 가치 네트워크 + 정책 네트워크
			- 입실론을 쓰지 않는다 : 모든 행동이 확률적으로 나오기 때문에, 확률뷴포에 따르는 선택을 한다
			- gradient v(s) = gradient log 파이(a|s) * q(s, a)) -> 정책 기반이라서 상태에서 큐 없이 바로 행동을 뽑을거라 q가 없다 -> q도 네트워크로 근사하자 =
			  가치신경망 = 크리틱
			- 정책을 평가하는건 q네트워크로, 정책 발전은 정책망 업데잍로

			- reinforce를 떠올려 보면 loss = - log 파이(a|s) * Q(s, a) 인데 여기에서 Q에 따라 loss가 많이 변화하므로 목적함수의 분산이 커진다
			- 목적함수를 안정화시키기 위해서 어느정도 정규화를 해야하는데 이 방법으로 베이스라인을 사용
			- A(s, a) = Q(s, a) - V(s)라고 어드밷티지 함수값을 정의하고 이것보다는 오로지 V만을 써서 재정의하면
			- 델타 = R(t + 1) + gamma * V(S(t+1)) - V(S(t))
			- 이걸 Q 대신 씀

			- 또한 Q네트워크도 업데이트 해야 하니까 두번쨰 LOSS를 정의한다
			- (R + gamma * V(S') - V(S))^2으로 최적화

		- a3c = dqn 발전
			- dqn의 리플레이 메모리는 일단 메모리땜에 느리고 이전 정책으로 학습한다는 오프폴리시의 단점이 있으며 메모리사이의 연관성 문제도 존재
			- 메모리끼리의 연관성 문제를 많은 메모리양으로 해결하지 말고 병렬 + 비동기로 해결 = 멀티스레딩
			- 여러 에이전트가 병렬로 작동해서 리플레이 메모리 문제의 연관성 문제를 해결하고 글로벌 네트워크를 업데이트 한다
			- q에 엔트로피 수식을 추가해서 exploration 경향성을 추가한다

		- 브레이크 아웃
			- 4개의 이미지를 프레임 스킵으로 받아오고 모두 차원을 1로 줄여서 (84 84 4)의 텐서를 cnn의 입력으로 받아 처리함

		- 아이디어
			- 일단 상위 에이전트가 상태를 분할할 수 있어야 한다
			- 분할된 상태를 작은 에이전트가 해결하고 결과로 얻어진 상태를 추상화해서 상위 에이전트에 전달
			- 상위 에이전트는 하위 에이전트들의 추상화된 상태들의 집합을 자신의 상태로 인식한다
			