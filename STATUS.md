# 진행 상태 (2026-09-16)

## 완료
- 원본 첨부 데이터(YouTube_Comments_Raw_meta.csv, YouTube_Comments_Filtered.csv,
  YouTube_Comments_Cleaned.csv, YouTube_Video_Metadata.csv)와 논문 PDF를 `data/`에 저장.
- `venv/`에 재현에 필요한 패키지 설치 및 임포트 확인 완료:
  torch, numpy, pandas, scikit-learn, scipy, sentence-transformers,
  bertopic, umap-learn, hdbscan, gensim, kiwipiepy.
- Kiwi 형태소 분석기 동작 확인 (NNG/NNP 명사 추출 가능).

## 차단됨 — 재현 실행 불가
`sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2` 임베딩 모델을
Hugging Face Hub에서 내려받아야 하는데, 이 세션의 아웃바운드 네트워크 정책이
huggingface.co 및 대체 경로(hf-mirror.com, modelscope.cn, gitee.com,
download.pytorch.org)를 전부 차단하고 있음. `pypi.org`, `github.com` 등
제한된 화이트리스트만 허용되는 상태.

이 모델 다운로드가 과제 1~4 전체의 선행 조건이므로, 해결 전까지 실제 숫자
산출이 불가능함. 추정치를 만들어 보고하지 않음.

## 다음 단계 (택1)
1. claude.ai/code 환경 설정에서 네트워크 정책을 huggingface.co 허용으로 변경 후 재시도
2. 기존 계산된 임베딩(.npy, 626건, cleaned_text 순서 일치) 업로드받아 진행
3. GitHub 미러를 통한 모델 파일 확보 시도 (무결성 검증 필요, 성공 불확실)
