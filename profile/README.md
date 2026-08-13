<p align="center">
  <img src="./assets/로고.png" alt="2026 AI·SW중심대학 디지털 경진대회 AI부문" width="100%" />
</p>

# 2026 AI·SW중심대학 디지털 경진대회 : AI부문
## 0. 최종 결과(대상, 부총리상)

<img src="./assets/수상사진.png" alt="2026 AI·SW중심대학 디지털 경진대회 AI부문" width="100%" />

- **본선 1위(대상, 부총리상)**
- **인기상**
- [발표 자료 보기](./assets/발표자료.pdf)


## 1. 예선 최종 스코어 (2026.07.15 마감일 기준)

<p align="center">
  <img src="./assets/최종순위.png" alt="Private 리더보드 최종 순위 1위" width="100%" />
</p>

- **Private LB 1위**
- **최종 점수: 0.798786666**
- 팀명: **너만 오면 오인 큐**

## 2. 팀 멤버

<table>
  <tr>
    <td align="center">
      <a href="https://github.com/dev-jake-kim">
        <img src="https://github.com/dev-jake-kim.png?size=120" width="100px;" alt="김진수" />
        <br />
        <sub><b>김진수</b></sub>
        <br />
        <sub>팀장 · 추론 최적화</sub>
        <br />
        <sub>@dev-jake-kim</sub>
        <br/>
        <sub>jake.sw.engineer@gmai.com</sub>
      </a>
    </td>
    <td align="center">
      <a href="https://github.com/baeksuenang">
        <img src="https://github.com/baeksuenang.png?size=120" width="100px;" alt="백성현" />
        <br />
        <sub><b>백성현</b></sub>
        <br />
        <sub>seed variance 개선</sub>
        <br />
        <sub>@baeksuenang</sub>
        <br/>
        <sub>jbaegs409@gmail.com</sub>
      </a>
    </td>
    <td align="center">
      <a href="https://github.com/Talusor">
        <img src="https://github.com/Talusor.png?size=120" width="100px;" alt="이태호" />
        <br />
        <sub><b>이태호</b></sub>
        <br />
        <sub>입력 구조화</sub>
        <br />
        <sub>@Talusor</sub>
        <br/>
        <sub>aakmods@gmail.com</sub>
      </a>
    </td>
    <td align="center">
      <a href="https://github.com/hyundoria">
        <img src="https://github.com/hyundoria.png?size=120" width="100px;" alt="이현석" />
        <br />
        <sub><b>이현석</b></sub>
        <br />
        <sub>도메인 적응력 개선</sub>
        <br />
        <sub>@hyundoria</sub>
        <br/>
        <sub>hyundoria@gmail.com</sub>
      </a>
    </td>
  </tr>
</table>

## 3. 대회 개요

| 항목 | 내용 |
| --- | --- |
| 주제 | AI Agent 행동(Action) 의사결정 예측 챌린지 |
| 문제 유형 | AI 코딩 에이전트 세션 상태 기반 14-class action classification |
| 입력 정보 | 현재 사용자 발화, 직전까지의 대화·행동 이력, 세션 메타정보 |
| 주요 제한 | 추론 10분 이하, 제출 파일 1GB 이하 |

## 4. 접근 전략

```plain text
1. mmBERT-small 기반 14-class agent action 분류 baseline 구축
2. 현재 사용자 의도와 이전 행동/결과/세션 상태를 분리한 BERT pair 입력 설계
3. long-context 입력, TAPT, R-Drop, EMA, label smoothing으로 단일 모델 성능 개선
4. seed 탐색 및 softmax 평균 앙상블로 정체 구간 돌파
5. 지식증류(KD)로 앙상블 지식을 단일 모델에 압축
6. delta 압축, vocab pruning, embedding freeze로 1GB 제출 제한 대응
7. progressive early-exit과 동적 배칭으로 최종 앙상블 추론 시간 단축
```

- 이 대회는 단순 intent classification이 아니라, 대화의 현재 상태와 직전 도구 실행 결과를 함께 보고 다음 agent action을 예측하는 문제로 접근했다.
- 같은 사용자 발화라도 세션 단계에 따라 필요한 행동이 달라질 수 있으므로, 현재 프롬프트와 과거 상태를 하나의 문자열로 합치지 않고 `text1/text2` pair 입력으로 분리했다.
- 최종적으로는 자체 long-context 계열 5개 모델과 팀원 pair-input 계열 3개 모델을 결합한 하이브리드 앙상블을 사용했다.
- 제출 용량과 추론 시간 제한이 있었기 때문에, 성능 개선뿐 아니라 모델 패키징과 추론 파이프라인 최적화도 중요한 축으로 다뤘다.

## 5. 주요 제출 결과

| 제출번호 | 파일 | 추론 시간 | 점수 | 비고 |
| --- | --- |  --- | --- | --- |
| 34837 | `fast_final.zip` | 5분 14초 | **0.798786666** | 최종 제출, 추론 최적화 적용 |
| 34531 | `gen14_900011.zip` | 1분 52초 | 0.7970027741 | 최종 순위기준 9-10위, 본선진출자 중 내 encoder기반 단일 모델중 최고성능/최속성능 |
| 28346 | `submit.zip` |  1분 29초 | 0.7830539834 | 최초 베이스라인 |

## 6. 단일 모델 Architecture

### 6-1) 최초 베이스라인

- `jhu-clsp/mmbert-small` 기반 14-class 행동 예측 모델이다.
- 현재 사용자 프롬프트뿐 아니라 직전 agent 행동, 실행 결과, 작업공간 상태, 열린 파일 목록, 최근 action sequence를 함께 사용했다.
- 입력은 현재 의도(`text1`)와 이전 행동/세션 문맥(`text2`)을 분리한 BERT pair 구조로 구성했다.
- 초기 모델은 CV macro-F1 약 `0.782`를 기록했고, 운영진측 베이스라인 `0.427` 대비 큰 폭으로 개선되었다.
<details>
<summary>입력 예시(눌러서 확인)</summary>
<br>

```text
<TURN_4>
<CUR> 이제 parser 테스트만 한번 돌려봐 가볍게 </CUR>
<PREV_USER> command이 sh -c로 감싸는구나... 이거 인젝션 표면이네. 실제 동작 한번만 찍어보자
</PREV_USER>
[SEP]
<LAST_ACTION> <A_RUN_TESTS> </LAST_ACTION>
<LAST_RESULT> <RESULT_FAILED> <HAS_TEST_FAIL> FAIL: 191 tests failing </LAST_RESULT>
<LAST_ARGS> path=tests/Button.test.tsx ext=tsx pattern=none cmd=<CMD_UNKNOWN>
</LAST_ARGS>

<PREV_ACTION> <A_EDIT_FILE> </PREV_ACTION>
<ACTION_SEQ> <A_READ_FILE> <A_EDIT_FILE> <A_RUN_TESTS> </ACTION_SEQ>

<OPEN_FILES> package.json </OPEN_FILES>
<PROMPT_FEATURE> <P_WORD_RUN> <P_WORD_TEST> </PROMPT_FEATURE>
<SESSION_FEATURE> <DIRTY_TRUE> <CI_FAILED> <OPEN_SOME> turn=4 history=6 actions=3
loc=58700 budget=70033 primary_lang=ts open_exts=json </SESSION_FEATURE>
```

</details>

### 6-2) long-context 확장

| 항목 | baseline | long-context |
| --- | --- | --- |
| 과거 실행 결과 | 없음 | 최근 6개 |
| 직전 결과 원문 | 220자 | 600자 |
| 열린 파일 목록 | 8개 | 24개 |
| action 시퀀스 | 12개 | 24개 |

- baseline은 직전 action 1건의 결과만 볼 수 있어, 세션이 진행될수록 누적된 실행 흐름을 파악할 수 없었다.
- `<HISTORY_RESULTS>` 블록을 신설해, 직전 1건을 제외한 그 이전 action들의 이름과 결과 원문을 함께 제공했다.
- `<ACTION_SEQ>`가 "무엇을 했는지"의 요약이라면, `<HISTORY_RESULTS>`는 "그래서 어떤 결과가 나왔는지"를 담당한다.
- short context 모델과 long context 모델을 함께 앙상블해 단기 관점과 장기 관점을 상호 보완했다.

### 6-3) 정규화

**TAPT — 과제 적응 사전학습**

- 분류 fine-tuning 이전에, 대회 데이터의 텍스트만으로 MLM을 이어서 학습해 backbone을 태스크 도메인에 적응시킨다.
- 분류 단계와 동일한 직렬화로 코퍼스를 구성해, 새로 추가한 특수 토큰이 사전학습 단계에서부터 문맥을 획득하도록 했다.
- 라벨을 쓰지 않는 자기지도 단계이며, 첫 시드에서 한 번만 실행한 뒤 backbone을 캐시해 모든 시드가 공유한다.

**R-Drop — 드롭아웃 일관성 정규화**

- 같은 배치를 dropout mask만 다르게 하여 두 번 forward하고, 두 예측 분포가 어긋난 정도를 양방향 KL로 벌점화한다.
- 학습 시(dropout on)와 추론 시(dropout off)의 동작 불일치를 좁혀, 특수 토큰에 대한 과적합을 억제한다.

- 함께 적용한 정규화: EMA, label smoothing, weighted CE, embedding freeze(원본 vocab은 동결하고 특수 토큰만 학습)

## 7. 대회 제약사항에 따른 모델 학습 및 추론

### 7-1) 지식증류

- 앙상블 모델 개수를 선형적으로 늘릴 경우 용량/시간적 제한을 넘어서므로 여러 모델을 하나로 압축시켜 seed variance를 감소시켰다.
<details>
<summary>증류 예시(눌러서 확인)</summary>
<br>
<img src="./assets/증류.png" alt="2026 AI·SW중심대학 디지털 경진대회 AI부문" width="100%" />
</details>

### 7-2) Delta 기반 앙상블 압축

- 용량 제한 안에 모델을 넣기 위해, 전체 checkpoint를 저장하지 않고 `base + delta` 구조로 압축했다.

|  | 적용 전 | 적용 후 | 비고 |
| --- | --- | --- | --- |
| 용량 | 1.8Gb | 0.88Gb | 약 57%감소 |

### 7-3) 델타친화 웜스타트 증류

- 성능이 가장 좋았던 증류 모델은 base와 가중치 거리가 멀어 delta 압축이 되지 않았고, 그대로는 용량 한도에 들어가지 못했다.
- 새로 학습하는 대신, 이미 완성된 checkpoint에서 전체 가중치(분류 헤드 포함)를 warm-start한 뒤 낮은 학습률로 증류만 추가 학습했다.
- 임베딩은 freeze되어 차이가 0으로 유지되고, 나머지 파라미터도 출발점 근처에 머물러 delta가 형제 시드 수준으로 작아진다.
- 성능을 유지하면서 압축 가능한 형태로 만들어 최종 앙상블에 편입할 수 있었다.

### 7-4) 병렬처리
 - 모델추론: GPU consumming job
 - 모델 압축 해제: CPU consumming job
 - 위 두 과정 병렬처리
### 7-5) Progressive Early-Exit
 - 앙상블 과정에서 지금까지 실행된 모델의 확률을 합산시 margin (1위 클래스 –2위 클래스확률차) > remaining weight (남은 모델 가중치 합) 이면 조기 종료

### 7-6) 길이 기반 동적 배칭

- Transformer 배치 추론은 배치 내 최장 샘플 길이에 맞춰 전체가 패딩되므로, 입력 길이 편차가 클수록 패딩 연산이 낭비된다.
- 직렬화·토크나이즈를 1회만 수행해 실제 토큰 길이를 확보하고(이 결과는 앙상블의 모든 멤버가 재사용), 길이 내림차순으로 정렬해 비슷한 길이의 샘플이 같은 배치에 모이게 했다.
- 고정 개수 대신 **토큰 예산**(배치 크기 × 배치 내 최장 길이) 기준으로 배치를 가변 구성하고, 추론 후 원래 순서로 복원한다.
- 예산을 기존 고정 배치의 최악 케이스와 동일한 값으로 잡아, 메모리 상한을 그대로 유지하면서 패딩 낭비와 GPU step 수를 함께 줄였다.
- LayerNorm 기반이고 패딩 토큰은 attention mask로 배제되므로, 배치 구성이 달라져도 예측 결과는 동일하다.


## 8. 참고

- 공식 대회 링크: [DACON 2026 AI·SW중심대학 디지털 경진대회 : AI부문](https://dacon.io/competitions/official/236694/overview/description)
## 9. 포스터
<img src="./assets/포스터.png" alt="2026 AI·SW중심대학 디지털 경진대회 AI부문" width="70%" />
