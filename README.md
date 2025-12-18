# 🇰🇷🇺🇸 한국어 → 영어 기계번역 (Seq2Seq vs Seq2Seq+Attention)

## 📌 프로젝트 요약
한국어 문장을 영어로 번역하는 NMT(Neural Machine Translation) 모델을 직접 구현하고,
기본 Seq2Seq 모델과 Attention 적용 모델의 성능을 학습 로그/샘플 번역/BLEU로 비교했다.

---

## 목표
- 한국어(ko) → 영어(mt) 번역 모델 구축
- 2가지 구조 비교
  1) GRU 기반 Seq2Seq (Attention 없음)
  2) Bahdanau Attention 기반 Seq2Seq + (SentencePiece 토크나이징)

---

## 데이터
- JSON 형식
- 각 샘플은 아래 키를 가짐
  - "ko": 한국어 원문
  - "mt": 영어 번역문

예시 경로(환경에 맞게 수정)
- train: .../일상생활및구어체_한영_train_set.json
- valid: .../일상생활및구어체_한영_valid_set.json

---

## 전체 흐름
1. 데이터 로드 (JSON → (ko, en) pair)
2. 토크나이징 / 정수 인덱스 변환
   - 기본 모델: (단어 기반 토큰화)
   - Attention 모델: SentencePiece(unigram, vocab_size=8000)
3. 패딩(padding) + DataLoader 구성
4. 모델 정의
   - Encoder(GRU) / Decoder(GRU)
   - Attention 모델은 Decoder가 매 타임스텝마다 “어떤 입력 단어를 참고할지” 가중치를 계산
5. 학습(Teacher Forcing) + 검증 loss로 best 저장
6. 평가
   - 샘플 번역(정성)
   - BLEU(정량)

---

## 실험 설정
- batch_size: 64
- embedding_dim: 256
- hidden_dim: 256
- encoder_layers / decoder_layers: 1
- dropout: 0.1
- lr: 0.001
- epochs: 30

---

## 🟨 결과 요약

### 1) 기본 Seq2Seq (Attention 없음)
- 학습 Loss가 꾸준히 감소
  - Epoch 1 Loss: 0.7674
  - Epoch 30 Loss: 0.2691
- 하지만 샘플 번역에서 반복/의미 없는 출력이 자주 발생 (정성 평가에서 한계가 뚜렷)

### 2) Seq2Seq + Bahdanau Attention + SentencePiece
- 검증 loss 기준 best 모델이 존재
  - best Val Loss: 4.819 (Epoch 20에서 best)
- 최종 BLEU
  - BLEU: 0.1298
- 샘플 번역에서 UNK(⁇) 토큰이 다수 등장 → 토크나이저/어휘/디코딩 전략 개선 여지

---

## 결과 해석
- 기본 Seq2Seq는 입력 문장이 길어질수록 Encoder가 정보를 압축해서 “기억”하기가 어려움
  → 번역이 길어지면 의미가 무너지거나 반복이 발생하기 쉬움
- Attention은 “현재 번역할 단어에 필요한 입력 위치”를 매번 다시 보게 해주므로 구조적으로 유리
  → 다만 이번 실험에서는
    - 데이터 규모/문장 다양성
    - SentencePiece 설정
    - greedy decoding(탐욕적 디코딩) 한계 때문에 BLEU가 크게 나오지 못함

---

## 개선 아이디어
- 디코딩을 greedy → beam search로 변경
- SentencePiece vocab_size / model_type(bpe/unigram) 재탐색
- teacher forcing 비율 스케줄링
- dropout, layer 수, hidden size 튜닝
- Transformer 계열로 확장



