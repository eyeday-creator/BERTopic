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

## 2026-09-16 추가: 파라미터 정정 (논문 본문 대조 결과)

`data/paper.pdf` 연구방법 2절을 직접 대조한 결과, 최초 지시했던 설정 중 아래 3가지가
실제 논문과 달랐다. 재현 재시도 시 반드시 이 값을 사용할 것:

| 항목 | 잘못 전달된 값 | 논문의 실제 값 |
|---|---|---|
| 임베딩 모델 | sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2 | **snunlp/KR-SBERT-V40K-klueNLI-augSTS** |
| HDBSCAN min_cluster_size | 15 | **20** |
| CountVectorizer | min_df=5, lowercase=True | 위와 동일 + **ngram_range=(1,2)** |

나머지 설정(UMAP n_neighbors=7/n_components=5/min_dist=0.0/cosine/random_state=42,
MMR diversity=0.3, top_n_words=10, nr_topics=9, reduce_outliers strategy='distributions',
Kiwi NNG/NNP 2글자 이상, 불용어 21개)은 원 지시와 논문이 일치함.

새 세션(`session_01JfgwXV3RDAUvXHnMDKK8pA`)은 huggingface.co 접속 테스트 단계에서
차단되어 멈춘 상태이므로 잘못된 설정으로 계산된 결과는 없음. 재시도 시 위 표의
정정값을 반영해서 시작할 것.

## 2026-09-16 추가 2: 원본 Colab 코드(0617.py) 확보 후 재정정 — 이 표가 최종 확정본

사용자가 실제 논문 작성에 쓴 원본 Colab 코드(`bertofic_...0617.py`)를 확인한 결과,
**"2026-09-16 추가"에서 정정했던 값이 다시 뒤집힌다.** 논문 본문 서술이 실제로는
틀렸고(사용자 확인 완료), 아래가 실제 코드 기준 최종 확정 설정이다.

| 항목 | 확정값(코드 기준) | 비고 |
|---|---|---|
| 임베딩 모델 | `sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2` | 최초 지시값과 동일(원복). snunlp/KR-SBERT는 논문 오기재였음 |
| HDBSCAN min_cluster_size | **15** | 최초 지시값과 동일(원복). 20은 논문 오기재였음 |
| CountVectorizer ngram_range | **미지정(기본값, 유니그램만)** | 논문의 "(1,2) 바이그램" 서술은 오기재였음 |
| 유사도 필터링 닻 문장 | "초등학교, 중학교, 고등학교 학교 현장의 인공지능(AI) 교육 도입과 AI 디지털 교과서 활용에 대한 교사, 학생, 학부모의 긍정적인 기대와 부정적인 문제점 우려" | 논문에 인용된 "…수용성과 저항" 문장은 오기재였음. 과제4(임계값 민감도) 재현 시 이 문장 사용 |
| 데이터 수집 검색 쿼리 | `"AI 디지털 교과서" OR "초중고 인공지능 교육" OR "학교 AI 교육" OR "생성형 AI 교실" OR "AI 교과서"` | 논문 서술("AIDT", "디지털 교과서 도입" 포함)과 다름 |
| 임베딩 대상 텍스트(cleaned_text)의 품사 범위 | **명사만이 아니라 조사(J)/어미(E)/부호(S) 태그만 제거한 일반 실질어**(동사·형용사·부사 포함) | 논문 표1의 "명사 위주 추출"은 BERTopic의 CountVectorizer(c-TF-IDF 키워드 추출) 단계에만 적용되는 별도 토크나이저(`tokenize_korean_nouns`, NNG/NNP만)이며, 임베딩용 cleaned_text 생성 단계와는 다른 파이프라인임 |

**재현 시 주의할 코드상 잠재 버그**: `CountVectorizer(tokenizer=tokenize_korean_nouns, stop_words=stop_words_list, ...)`에서
`tokenize_korean_nouns` 함수 자체는 `stop_words_list`를 참조하지 않음. sklearn이 custom
tokenizer 지정 시 stop_words 필터링을 기대대로 수행하지 않을 수 있어, 논문 표 2의
핵심 키워드에서 21개 불용어가 실제로 제거되었는지 재현 시 반드시 직접 검증할 것.

새 세션을 재개할 때는 이 표를 최종 확정본으로 사용하고, "2026-09-16 추가"(첫 번째 정정)의
snunlp/20/ngram(1,2) 값은 폐기할 것.
