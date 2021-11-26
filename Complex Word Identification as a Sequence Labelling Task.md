# Complex Word Identification as a Sequence Labelling Task
# 시퀀스 라벨링 작업으로 복잡한 단어 식별



## Abstract

Complex Word Identification (CWI) is concerned with detection of words in need of simplification and is a crucial first step in a simplification pipeline. It has been shown that reliable CWI systems considerably  mprove text simplification. However, most CWI systems to date address the task on a word-by-word basis, not taking the context into account. In this paper, we present a novel approach to CWI based on sequence modelling. Our system is capable of performing CWI in context, does not require extensive feature engineering and outperforms state-of-the-art systems on this task.
복잡한 단어 식별(CWI)은 단순화가 필요한 단어의 감지와 관련이 있으며 단순화 파이프라인의 중요한 첫 번째 단계입니다. 신뢰할 수 있는 CWI 시스템은 텍스트 단순화를 상당히 향상시키는 것으로 나타났습니다. 그러나 현재까지 대부분의 CWI 시스템은 문맥을 고려하지 않고 단어 단위로 작업을 처리합니다. 이 논문에서는 시퀀스 모델링을 기반으로 하는 CWI에 대한 새로운 접근 방식을 제시합니다. 우리 시스템은 상황에 따라 CWI를 수행할 수 있으며 광범위한 기능 엔지니어링이 필요하지 않으며 이 작업에서 최첨단 시스템보다 성능이 뛰어납니다.



## 1. Introduction

Lexical complexity is one of the main aspects contributing to overall text complexity (Dubay, 2004). It is typically addressed with lexical simplification (LS) systems that aim to paraphrase and substitute complex terms for simpler alternatives. Previous research has shown that Complex Word Identification (CWI) considerably improves lexical simplification (Shardlow, 2014; Paetzold and Specia, 2016a). This is achieved by identifying complex terms in text prior to word substitution. The performance of a CWI component is crucial, as low recall of this component might result in an overly difficult text with many missed complex words, while low precision might result in meaning distortions with an LS system trying to unnecessarily simplify non-complex words (Shardlow, 2013).
어휘 복잡성은 전체 텍스트 복잡성에 기여하는 주요 측면 중 하나입니다(Dubay, 2004). 이것은 일반적으로 더 간단한 대안을 복잡한 용어로 바꾸고 대체하는 것을 목표로 하는 어휘 단순화(LS) 시스템으로 해결됩니다. 이전 연구에서는 복잡한 단어 식별(CWI)이 어휘 단순화를 상당히 향상시키는 것으로 나타났습니다(Shardlow, 2014; Paetzold and Specia, 2016a). 이것은 단어를 대체하기 전에 텍스트에서 복잡한 용어를 식별함으로써 달성됩니다. CWI 구성요소의 성능은 매우 중요합니다. 이 구성요소의 낮은 재현율은 많은 복잡한 단어를 놓친 지나치게 어려운 텍스트를 초래할 수 있는 반면, 낮은 정밀도는 LS 시스템이 복잡하지 않은 단어를 불필요하게 단순화하려는 의미 왜곡을 초래할 수 있기 때문입니다(Shardlow , 2013).



CWI has recently attracted attention as a standalone application, with at least two shared tasks focusing on it. Current approaches to CWI, including state-of-the-art systems, have a number of limitations. First of all, CWI systems typically address this task on a word-by-word basis, using a large number of features to capture the complexity of a word. For instance, the CWI system by Paetzold and Specia (2016c) uses a total of 69 features, while the one by Gooding and Kochmar (2018) uses 27 features. Secondly, systems performing CWI in a static manner are unable to take the context into account, thus failing to predict word complexity for polysemous words as well as words in various metaphorical or novel contexts. For instance, consider the following two contexts of the word molar from the CWI 2018 shared task (Yimam et al., 2018). Molar has been annotated as complex in the first context (resulting in the binary annotation of 1) by 17 out of 20 annotators (thus, the “probabilistic” label of 0:85), and as non-complex (label 0) in the second context:
CWI는 최근 두 가지 이상의 공유 작업에 초점을 맞춘 독립 실행형 애플리케이션으로 주목받고 있습니다. 최신 시스템을 포함한 CWI에 대한 현재 접근 방식에는 여러 가지 제한 사항이 있습니다. 우선, CWI 시스템은 일반적으로 단어의 복잡성을 캡처하기 위해 많은 기능을 사용하여 단어별로 이 작업을 처리합니다. 예를 들어 Paetzold and Specia(2016c)의 CWI 시스템은 총 69개의 기능을 사용하는 반면 Gooding과 Kochmar(2018)의 시스템은 27개의 기능을 사용합니다. 둘째, 정적 방식으로 CWI를 수행하는 시스템은 컨텍스트를 고려할 수 없으므로 다의어 단어 및 다양한 은유적이거나 새로운 컨텍스트의 단어에 대한 단어 복잡성을 예측하지 못합니다. 예를 들어, CWI 2018 공유 작업(Yimam et al., 2018)에서 molar라는 단어의 다음 두 컨텍스트를 고려하십시오. Molar는 첫 번째 컨텍스트에서 20개 주석 중 17개(따라서 "확률적" 레이블 0:85)에 의해 복잡한 것으로 주석이 달렸고(이진 주석이 1이 됨) 두 번째 컨텍스트: