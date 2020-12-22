# kookey - Keyword Synonym Extractor
훌륭한 키워드 추출기들이 있지만, **영어, 한글, 특수문자가 혼용되고 특정 양식이나 HTML, 에러 로그 등 다양한 형식의 문장을 포함하는 corpus**에서 키워드, 동의어를 추출하는 데에 어려움이 있었습니다. 또한, 어떤 키워드 추출기들은 작은 사양의 컴퓨터에서 실행하기에는 어려움이 있었습니다. 따라서, 조금은 더 가볍고, 다양한 키워드를 추출할 수 있는 키워드, 동의어 추출기를 만들고자 했습니다.

이 패키지에는 아래와 같은 이론(또는 기술, 모델)이 적용되었습니다.
- 공통 : 형태소 분석
- 전처리 : 정규표현식, 문장 분리, 자카드 유사도
- 키워드 추출 : N-gram, PPMI, 동시등장빈도
- 동의어 추출 : FastText pre-training, fine-tuning(pooling & cosine-loss / Bi-LSTM), 코사인 유사도

*문장으로 학습한 단어 임베딩 모델을 fine-tuning을 통해 의미 있는 동의어를 추출해내고자 했으나, 이는 실험적인 기능입니다. 이 기능을 업그레이드 할 수 있는 아이디어는 언제나 환영입니다. -> [의견](https://github.com/HyeyeonKoo/kookey/issues)*

*해당 패키지의 코드에는 이기창님께서 작성하신 [한국어임베딩 튜토리얼](https://github.com/ratsgo/embedding)의 자소분해 및 복원 코드가 포함되어 있으며, 해당 함수에 표기되어 있습니다.*

## Install
```
pip install kookey
```
```
import kookey
kookey.__version__
```

## Quick Start
아래와 같은 방법으로 패키지를 이용해보실 수 있습니다. 더 자세한 기능은 [test](https://github.com/HyeyeonKoo/kookey/tree/main/test)를 참고하십시오.

### Preprocessing
- 실행
```{.python}
from kookey.preprocess import Preprocess

preprocess = Preprocess()
result = preprocess.preprocess("안녕하세요. 반갑습니다. 성함이     어떻게 되세요?")
```
- 결과
``` python
["안녕하세요.", "반갑습니다.", "성함이 어떻게 되세요?"]
```

### Extracting Keywords
- 실행
```{.python}
sentences = [
    "오늘은 약 10-12도 맑음이니까 날씨가 좋아요.",
    "복합명사를 추출할 수 있는데, 이런게 복합이죠!.",
    "이번에 새로 들여온 설비의 이름은 built-in-982입니다.",
    "로그를 보니 ^$^&%&#@#$ 이렇게 적혀있네요."
]

from kookey.extract_keyword import KeywordExtractor

keyword_extractor = KeywordExtractor(
    calculate_ppmi=True,
    save_characters=["-"], 
    save_morphs=["NNG", "SL", "SN", "OL", "NR"]
)

keyword_extractor = KeywordExtractor(calculate_ppmi=True,
    save_characters=["-"], save_morphs=["NNG", "SL", "SN", "OL", "NR"])
keywords = keyword_extractor.get_keyword_candidate(sentences)
```
- 결과
```
[('복합', 2, '1_gram'), ('오늘', 1, '1_gram'), ('10-12', 1, '1_gram'), ('날씨', 1, '1_gram'), ('명사', 1, '1_gram'), ('추출', 1, '1_gram'), ('이번', 1, '1_gram'), ('설비', 1, '1_gram'), ('이름', 1, '1_gram'), ('built-in', 1, '1_gram'), ('-982', 1, '1_gram'), ('로그', 1, '1_gram'), ('복합명사', 1, '2_gram'), ('built-in-982', 1, '2_gram'), ('built-in-982', 2.48, 'ppmi-2_gram'), ('복합명사', 1.79, 'ppmi-2_gram')]
```

### Extracting Synonym
- 실행
```{.python}
sentences = [
    "사과는 과일이다.", 
    "토마토는 과일이 아니다.", 
    "딸기는 과일이다."
]
eval_data = [
    ("딸기", "토마토", 1), 
    ("토마토", "사과", 1)
]
base_words = ["딸기"]
compare_words = ["토마토", "사과"]

from kookey.extract_synonym import SynonymExtractor

synonym_extractor = SynonymExtractor(em_threshold=0)
candidate = synonym_extractor.get_keyword_candidate(
    sentences, 
    base_words, 
    compare_words
)
```
- 결과
```
[('딸기', '사과', 0.0498803)]
```
- 실행
```{.python}
train_data = [("사과", "딸기"), ("사과", "토마토")]
train_label = [1, 0]
test_data = [("딸기", "토마토")]
test_label = [0]

from kookey.extract_synonym import SynonymExtractor

synonym_extractor = SynonymExtractor(tuning_mode=True, tune_threshold=0)
candidate = synonym_extractor.get_keyword_candidate(
    sentences, base_words, compare_words, 
    train_data, train_label, 
    test_data, test_label
)
```
- 결과
```
[('딸기', '토마토', 0.652692), ('딸기', '사과', 0.652692)]
```

