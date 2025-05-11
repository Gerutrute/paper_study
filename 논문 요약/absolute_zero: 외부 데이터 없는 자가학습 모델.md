# Absolute Zero 상세 요약

## 1. 동기와 패러다임 전환
- **기존 RLVR 한계**  
  - RLVR(Reinforcement Learning with Verifiable Rewards)은 사람 손으로 만든 질문·정답 쌍을 필요로 함<br>→ 데이터 수집·확장성의 병목 :contentReference[oaicite:0]{index=0}:contentReference[oaicite:1]{index=1}  
  - ‘제로’ RLVR(Zero-RLVR)조차도 여전히 전문가가 고른 QA 분포를 요구 :contentReference[oaicite:2]{index=2}:contentReference[oaicite:3]{index=3}  
- **Absolute Zero 제안**  
  - **완전 자가 생성(self-play)**: 외부 데이터 없이, 하나의 모델이 스스로 과제를 만들고 풀면서 학습  
  - **학습가능도(learnability)**를 기준으로 **과제 난이도**를 조절  
  - **알파제로(AlphaZero)**식 자가 대국(self-play)을 자연어 추론·코딩 영역으로 확장 :contentReference[oaicite:4]{index=4}:contentReference[oaicite:5]{index=5}  

## 2. 핵심 메커니즘
### 2.1. 두 역할: Proposer & Solver  
1. **Proposer(πₚ)**  
   - 과제 τ 생성: 과제 유형(z ∈ {deduction, abduction, induction})과 과거 예제 K개를 조건으로 샘플링 :contentReference[oaicite:6]{index=6}:contentReference[oaicite:7]{index=7}  
   - **Learnability Reward** rₚ:  
     - n번 몬테카를로 롤아웃으로 Solver 성공률 \(\bar r_{\mathrm{solve}}\) 계산  
     - \[  
         r_{\mathrm{propose}} =  
         \begin{cases}  
           0, & \bar r_{\mathrm{solve}}=0 \text{ or }1 \\  
           1-\bar r_{\mathrm{solve}}, & \text{otherwise}  
         \end{cases}  
       \] :contentReference[oaicite:8]{index=8}:contentReference[oaicite:9]{index=9}  
2. **Solver(πₛ)**  
   - 생성된 과제 τ의 쿼리 x에 대해 답안 y 생성  
   - **Accuracy Reward** rₛ:  
     - 답이 정답 y⋆와 일치하면 1, 아니면 0  
     - \(r_{\mathrm{solve}} = \mathbf{1}(y=y^*)\) :contentReference[oaicite:10]{index=10}:contentReference[oaicite:11]{index=11}  

### 2.2. 합성 보상 구조  
- **형식 검사(format penalty)**: 응답이 형식에 어긋나면 –0.5~–1 패널티 적용  
- **최종 보상** \(R(y^\pi)\) = rₚ 또는 rₛ, 형식 에러 시 패널티 :contentReference[oaicite:12]{index=12}:contentReference[oaicite:13]{index=13}  

### 2.3. 세 가지 과제 유형  
1. **Deduction**: 코드 p + 입력 i → 출력 o 예측  
2. **Abduction**: 코드 p + 출력 o → 입력 i 추론  
3. **Induction**: 일부 입출력 예시 → 코드 p 합성  
- **코드 활용 이유**: 튜링 완전성, 검증 가능성, 오픈엔디드(task space) :contentReference[oaicite:14]{index=14}:contentReference[oaicite:15]{index=15}  

### 2.4. 학습 루프 (Algorithm 1)  
1. **버퍼 초기화**: seed 데이터로 D₀ᵈₑd, D₀ᵃᵇᵈ, D₀ⁱⁿᵈ 구성 :contentReference[oaicite:16]{index=16}:contentReference[oaicite:17]{index=17}  
2. 매 스텝마다  
   - **Propose**: 각 과제 유형별로 τ 생성 → 실행기로 유효성·안전성·결정성 검증 :contentReference[oaicite:18]{index=18}:contentReference[oaicite:19]{index=19}  
   - **Solve**: 검증된 과제(x,y⋆)를 Solver가 풀고 rₛ 계산  
   - **업데이트**: Task-Relative REINFORCE++(TRR++)로 πₚ·πₛ 동시 갱신 :contentReference[oaicite:20]{index=20}:contentReference[oaicite:21]{index=21}  

## 3. 구현 및 세부 설정
- **프레임워크**: veRL 코드베이스, QwQ Python executor 사용 :contentReference[oaicite:22]{index=22}:contentReference[oaicite:23]{index=23}  
- **하이퍼파라미터**:  
  - Batch Size=64×6, LR=1e-6, 총 스텝 500, PPO Epochs=1 등 :contentReference[oaicite:24]{index=24}:contentReference[oaicite:25]{index=25}  
- **포매팅·후처리 실험**:  
  - 주석·docstring 제거 시 성능 저하 → 최종 실험에서는 유지 :contentReference[oaicite:26]{index=26}:contentReference[oaicite:27]{index=27}  
  - 글로벌 변수 제거 시에도 성능 저하 관찰 → 제거하지 않음 :contentReference[oaicite:28]{index=28}:contentReference[oaicite:29]{index=29}  

## 4. 실험 결과 및 검증
### 4.1. 주요 성능 지표
- **OOD 수학·코딩 벤치마크**  
  - HumanEval+, MBPP+, LiveCodeBench, AIME’24/’25, OlympiadBench 등에서 SOTA 달성 :contentReference[oaicite:30]{index=30}:contentReference[oaicite:31]{index=31}  
  - AZR-Coder-7B가 기존 제로-RLVR 대비 +1.8%p 코딩 성능 ↑ :contentReference[oaicite:32]{index=32}:contentReference[oaicite:33]{index=33}  
- **크로스 도메인 일반화**  
  - 코딩 학습만으로 수학 문제 성능 +10~15%p ↑ (기존 제로-RLVR +0.6%p) :contentReference[oaicite:34]{index=34}:contentReference[oaicite:35]{index=35}  
- **스케일링 효과**  
  - 3B→7B→14B 모델에서 성능 향상폭 +5.7→+10.2→+13.2%p :contentReference[oaicite:36]{index=36}:contentReference[oaicite:37]{index=37}  

### 4.2. 추가 분석  
- **과제 복잡도·다양성 지표**: Proposer가 점진적으로 더 복잡·다양한 과제를 생성 (Appendix C.4) :contentReference[oaicite:38]{index=38}:contentReference[oaicite:39]{index=39}  
- **Propose vs. Solve 상호작용**: 버퍼 업데이트와 보상 구조가 서로를 강화, 안정적 self-play 순환 형성 (Appendix C.3) :contentReference[oaicite:40]{index=40}:contentReference[oaicite:41]{index=41}  
- **비교 실험(ablation)**:  
  - 초기 p(z) 조정, 보상 설계 변화 등 다양한 대안 시도하였으나 최종 설계가 최적 (Appendix D) :contentReference[oaicite:42]{index=42}:contentReference[oaicite:43]{index=43}  

## 5. 한계 및 향후 과제
- **안전성 위험**: 자가 생성된 reasoning chain의 예기치 못한 행동 관찰 :contentReference[oaicite:44]{index=44}:contentReference[oaicite:45]{index=45}  
- **확장 방향**:  
  - 웹·시뮬레이터 환경 확장 → 비코드 멀티모달 reasoning  
  - reward hacking 방지 위한 안전성 장치 강화 :contentReference[oaicite:46]{index=46}:contentReference[oaicite:47]{index=47}  

---

위 요약을 통해 논문이 제시한 **자가 생성 학습 패러다임**, **구체적 보상 설계**, **검증 파이프라인**, 그리고 **광범위한 실험적 증명**을 한눈에 파악하실 수 있습니다. 필요에 따라 각 섹션의 풍부한 수식과 알고리즘 의사코드를 참고해 보시면 좋겠습니다.
