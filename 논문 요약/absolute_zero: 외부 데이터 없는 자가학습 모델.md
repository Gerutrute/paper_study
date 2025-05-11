# Absolute Zero: Reinforced Self-play Reasoning with Zero Data

## 1. 동기와 패러다임 전환
- **기존 RLVR 한계**  
  - RLVR(Reinforcement Learning with Verifiable Rewards)은 사람 손으로 만든 질문·정답 쌍을 필요로 함  
    → 데이터 수집·확장성의 병목[^1]  
  - ‘제로’ RLVR(Zero-RLVR)조차도 여전히 전문가가 고른 QA 분포를 요구[^1]
- **Absolute Zero 제안**  
  - **완전 자가 생성(self-play)**: 외부 데이터 없이 모델이 스스로 과제를 만들고 풀며 학습  
  - **학습가능도(learnability)**를 기준으로 **과제 난이도**를 조절  
  - 알파제로(AlphaZero)의 self-play 개념을 자연어 추론·코딩 영역으로 확장[^2]

## 2. 핵심 메커니즘
### 2.1. 두 역할: Proposer & Solver
1. **Proposer (π<sub>p</sub>)**  
   - 과제 τ 생성: 유형(z ∈ {deduction, abduction, induction})과 과거 예제 K개를 조건으로 샘플링[^8]  
   - **Learnability Reward** _r_<sub>propose</sub>  

     ```markdown
     **r_propose(τ)** =
     - 0, if bar_r_solve is 0 or 1  
     - 1 − bar_r_solve, otherwise
     ```

   - 여기서 `bar_r_solve`는 Solver가 해당 과제 τ를 여러 번 풀었을 때의 평균 성공률(0∼1 사이)입니다.  
     - 성공률이 0 또는 1이면 보상 0 → 너무 어렵거나 너무 쉬운 과제 억제  
     - 그 외에는 `1 − bar_r_solve` 보상 → “적당히 도전적” 과제 선호[^9]
2. **Solver (π<sub>s</sub>)**  
   - 생성된 과제 τ의 쿼리 x에 대해 답안 y 생성  
   - **Accuracy Reward** _r_<sub>solve</sub> = 1 if y = y*, else 0

### 2.2. 합성 보상 구조
- **형식 검사(format penalty)**: 응답 형식 오류 시 –0.5∼–1 패널티 적용  
- **최종 보상**은 Proposer 또는 Solver 보상에 형식 페널티를 더하여 결정

### 2.3. 과제 유형 3가지
1. **Deduction**: 코드 p + 입력 i → 출력 o 예측  
2. **Abduction**: 코드 p + 출력 o → 입력 i 추론  
3. **Induction**: 일부 입출력 예시 → 코드 p 합성  
- 코드(task) 사용 이유: 튜링 완전성, 검증 가능성, 무한한 과제 공간 제공[^3]

### 2.4. 학습 루프 개요
1. **버퍼 초기화**: seed 데이터로 D₀<sup>ded</sup>, D₀<sup>abd</sup>, D₀<sup>ind</sup> 구성[^15]  
2. 매 스텝마다  
   - **Propose**: τ 생성 → 실행기(validity, safety, determinism) 검증[^4]  
   - **Solve**: 검증된 (x, y*)를 Solver가 풀고 _r_<sub>solve</sub> 계산  
   - **업데이트**: Task-Relative REINFORCE++(TRR++)로 π<sub>p</sub>, π<sub>s</sub> 동시 갱신[^11]

## 3. 구현 및 세부 설정
- **프레임워크**: veRL 코드베이스, QwQ Python executor 사용[^0]  
- **하이퍼파라미터**: Batch Size=384, LR=1e-6, 총 500 스텝, PPO Epochs=1 등  
- **포매팅·후처리**: 주석·docstring 유지, 글로벌 변수 유지가 성능상 유리[^4]

## 4. 실험 결과 및 검증
### 4.1. 주요 성능 지표
- **OOD 수학·코딩 벤치마크**  
  - HumanEval+, MBPP+, LiveCodeBench, AIME’24/’25, OlympiadBench 등에서 SOTA 달성[^7]  
  - AZR-Coder-7B는 제로-RLVR 대비 +1.8%p 성능 향상[^15]
- **크로스 도메인 일반화**  
  - 코딩 학습만으로 수학 문제 성능 +10∼15%p 향상 (기존 제로-RLVR은 +0.6%p)
- **스케일링 효과**  
  - 3B → 7B → 14B 모델 크기 확장 시 성능 향상폭 +5.7 → +10.2 → +13.2%p[^6]

### 4.2. 추가 분석
- **과제 복잡도·다양성**: Proposer가 점진적으로 더 복잡·다양한 과제 생성 (Appendix C.4)[^5]  
- **상호작용 안정성**: Propose ↔ Solve 순환이 안정적 self-play 형성 (Appendix C.3)[^5]  
- **Ablation Study**: 다양한 보상·초기화 대안 실험 후 최적 설계 결정 (Appendix D)[^5]

## 5. 한계 및 향후 과제
- **안전성 위험**: 자가 생성된 reasoning chain의 예기치 못한 행동 관찰[^7]  
- **확장 방향**:  
  - 웹·시뮬레이터 환경으로 확대 → 멀티모달 reasoning  
  - reward hacking 방지 위한 안전장치 강화[^1]

---

## 각주 (Footnotes)
[^0]: 논문 참고자료 중 veRL 및 QwQ executor 설명 부분  
[^1]: “제한된 외부 데이터 의존성”에 대한 비판 (Section 1)  
[^2]: AlphaZero self-play 아이디어 소개 (Section 2)  
[^3]: 코드 기반 검증 가능성 설명 (Section 3)  
[^4]: 실행기 검증 및 처리 실험 결과 (Section 4.1)  
[^5]: Appendix C, D의 추가 분석 및 ablation study  
[^6]: 모델 스케일링 실험 결과 (Section 5)  
[^7]: OOD 벤치마크 성능 (Section 6)  
[^8]: Proposer 과제 생성 메커니즘 (Section 2.1)  
[^9]: Learnability Reward 수식 설명 (Section 2.1)
