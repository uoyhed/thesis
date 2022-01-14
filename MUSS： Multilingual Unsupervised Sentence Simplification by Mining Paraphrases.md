# MUSS: Multilingual Unsupervised Sentence Simplification by Mining Paraphrases
# MUSS: 의역 마이닝을 통한 다국어 비지도 문장 단순화



## Abstract

Progress in sentence simplification has been hindered by a lack of labeled parallel simplification data, particularly in languages other than English. We introduce MUSS, a Multilingual Unsupervised Sentence Simplification system that does not require labeled simplification data. MUSS uses a novel approach to sentence simplification that trains strong models using sentence-level paraphrase data instead of proper simplification data. These models leverage unsupervised pretraining and controllable generation mechanisms to flexibly adjust attributes such as length and lexical complexity at inference time. We further present a method to mine such paraphrase data in any language from Common Crawl using semantic sentence embeddings, thus removing the need for labeled data. We evaluate our approach on English, French, and Spanish simplification benchmarks and closely match or outperform the previous best supervised results, despite not using any labeled simplification data. We push the state of the art further by incorporating labeled simplification data.
문장 단순화의 진행은 특히 영어 이외의 언어에서 레이블이 지정된 병렬 단순화 데이터의 부족으로 인해 방해를 받았습니다. 레이블이 지정된 단순화 데이터가 필요하지 않은 Multilingual Unsupervised Sentence Simplification 시스템인 MUSS를 소개합니다. MUSS는 적절한 단순화 데이터 대신 문장 수준의 의역 데이터를 사용하여 강력한 모델을 훈련하는 문장 단순화에 대한 새로운 접근 방식을 사용합니다. 이러한 모델은 비지도 사전 훈련 및 제어 가능한 생성 메커니즘을 활용하여 추론 시간에 길이 및 어휘 복잡성과 같은 속성을 유연하게 조정합니다. 의미론적 문장 임베딩을 사용하여 크롤링에서 모든 언어로 이러한 의역 데이터를 마이닝하는 방법을 추가로 제시하여 레이블이 지정된 데이터의 필요성을 제거합니다. 우리는 영어, 프랑스어 및 스페인어 단순화 벤치마크에 대한 접근 방식을 평가하고 레이블이 지정된 단순화 데이터를 사용하지 않았음에도 불구하고 이전의 지도학습 결과 중 최고와 밀접하게 일치하거나 능가합니다. 레이블이 지정된 단순화 데이터를 통합하여 최신 기술을 더욱 발전시킵니다.



### 1. Introduction



Sentence simplification is the task of making a sentence easier to read and understand by reducing its lexical and syntactic complexity, while retaining most of its original meaning. Simplification has a variety of important societal applications, for example increasing accessibility for those with cognitive disabilities such as aphasia (Carroll et al., 1998), dyslexia (Rello et al., 2013), and autism (Evans et al., 2014), or for non-native speakers (Paetzold and Specia, 2016). Research has mostly focused on English simplification, where source texts and associated simplified texts exist and can be automatically aligned, such as English Wikipedia and Simple English Wikipedia (Zhang and Lapata, 2017). However, such data is limited in terms of size and domain, and difficult to find in other languages. Additionally, simplifying a sentence can be achieved in multiple ways, and depend on the target audience. Simplification guidelines are not uniquely defined, outlined by the stark differences in English simplification benchmarks (Alva-Manchego et al., 2020a). This highlights the need for more general models that can adjust to different simplification contexts and scenarios.
문장 단순화는 원래 의미의 대부분을 유지하면서 어휘 및 구문 복잡성을 줄임으로써 문장을 읽고 이해하기 쉽게 만드는 작업입니다. 단순화는 실어증(Carroll et al., 1998), 난독증(Rello et al., 2013) 및 자폐증(Evans et al., 2014)과 같은 인지 장애가 있는 사람들의 접근성을 높이는 것과 같이 다양한 중요한 사회적 응용 프로그램을 가지고 있습니다. , 또는 원어민이 아닌 경우(Paetzold and Specia, 2016). 연구는 영어 Wikipedia 및 Simple English Wikipedia(Zhang and Lapata, 2017)와 같이 원본 텍스트 및 관련 단순화 텍스트가 존재하고 자동으로 정렬될 수 있는 영어 단순화에 주로 초점을 맞췄습니다. 그러나 이러한 데이터는 크기와 영역 면에서 제한적이며 다른 언어에서는 찾기가 어렵습니다. 또한 문장을 단순화하는 것은 여러 가지 방법으로 달성할 수 있으며 대상 독자에 따라 다릅니다. 단순화 지침은 영어 단순화 벤치마크의 극명한 차이로 요약되어 고유하게 정의되지 않습니다(Alva-Manchego et al., 2020a). 이는 다양한 단순화 컨텍스트 및 시나리오에 적응할 수 있는 보다 일반적인 모델의 필요성을 강조합니다.



In this paper, we propose to train controllable models using sentence-level paraphrase data only, i.e. parallel sentences that have the same meaning but phrased differently. In order to generate simplifications and not paraphrases, we use ACCESS (Martin et al., 2020) to control attributes such as length, lexical and syntactic complexity. Paraphrase data is more readily available, and opens the door to training flexible models that can adjust to more varied simplification scenarios. We propose to gather such paraphrase data in any language by mining sentences from Common Crawl using semantic sentence embeddings. We show that simplification models trained on mined paraphrase data perform as well as models trained on existing large paraphrase corpora (cf. Appendix D). Moreover, paraphrases are more straightforward to mine than simplifications, and we show that they lead to models with better performance than equivalent models trained on mined simplifications (cf. Section 5.4).
이 논문에서 우리는 문장 수준의 의역 데이터만 사용하여 제어 가능한 모델을 훈련할 것을 제안합니다. 즉, 의미는 같지만 구문이 다른 병렬 문장입니다. 의역이 아닌 단순화를 생성하기 위해 ACCESS(Martin et al., 2020)를 사용하여 길이, 어휘 및 구문 복잡성과 같은 속성을 제어합니다. 의역 데이터를 더 쉽게 사용할 수 있으며 더 다양한 단순화 시나리오에 맞게 조정할 수 있는 유연한 모델을 훈련할 수 있습니다. 의미론적 문장 임베딩을 사용하여 크롤링에서 문장을 마이닝하여 모든 언어로 이러한 의역 데이터를 수집할 것을 제안합니다. 마이닝된 의역 데이터에 대해 훈련된 단순화 모델이 기존의 대규모 의역 말뭉치에 대해 훈련된 모델만큼 성능을 발휘함을 보여줍니다(부록 D 참조). 더욱이, 의역은 단순화보다 마이닝에 더 간단하며, 마이닝된 단순화에 대해 훈련된 동등한 모델보다 더 나은 성능을 가진 모델로 이어짐을 보여줍니다(섹션 5.4 참조).



Our resulting Multilingual Unsupervised Sentence Simplification method, MUSS, is unsupervised because it can be trained without relying on labeled simplification data,1 even though we mine using supervised sentence embeddings.2 We additionally incorporate unsupervised pretraining (Liu et al., 2019, 2020) and apply MUSS on English, French, and Spanish to closely match or outperform the supervised state of the art in all languages. MUSS further improves the state of the art on all English datasets by incorporating additional labeled simplification data.
우리의 결과인 Multilingual Unsupervised Sentence Simplification 방법인 MUSS는 지도된 문장 임베딩을 사용하여 마이닝하더라도 레이블이 지정된 단순화 데이터에 의존하지 않고 훈련될 수 있기 때문에 감독되지 않습니다. 영어, 프랑스어 및 스페인어에 MUSS를 적용하여 모든 언어에서 감독되는 최신 기술과 밀접하게 일치하거나 능가합니다. MUSS는 레이블이 지정된 추가 단순화 데이터를 통합하여 모든 영어 데이터 세트에 대한 최신 기술을 더욱 향상시킵니다.

\1. We use the term labeled simplifications to refer to parallel datasets where texts were manually simplified by humans.
\2. Previous works have also used the term unsupervised simplification to describe works that do not use any labeled parallel simplification data while leveraging supervised components such as constituency parsers and knowledge bases (Kumar et al., 2020), external synonymy lexicons (Surya et al., 2019), and databases of simplified synonyms (Zhao et al., 2020). We shall come back to these works in Section 2.
\1. 우리는 텍스트가 인간에 의해 수동으로 단순화된 병렬 데이터 세트를 참조하기 위해 단순화라는 용어를 사용합니다.
\2. 이전 작업에서는 비지도 단순화라는 용어를 사용하여 레이블이 지정된 병렬 단순화 데이터를 사용하지 않는 반면 구성 요소 분석기 및 지식 기반(Kumar et al., 2020), 외부 동의어 사전(Surya et al. ., 2019) 및 단순화된 동의어 데이터베이스(Zhao et al., 2020). 우리는 섹션 2에서 이러한 작업으로 돌아올 것입니다.



To sum up, our contributions are as follows:
- We introduce a novel approach to training simplification models with paraphrase data only and propose a mining procedure to create large paraphrase corpora for any language. 
- Our approach obtains strong performance. Without any labeled simplification data, we match or outperform the supervised state of the art in English, French and Spanish. We further improve the English state of the art by incorporating labeled simplification data.
- We release MUSS pretrained models, paraphrase data, and code for mining and training3.
요약하자면, 우리의 기여는 다음과 같습니다:
- 우리는 의역 데이터만으로 단순화 모델을 훈련하는 새로운 접근 방식을 소개하고 모든 언어에 대한 대규모 의역 말뭉치를 생성하기 위한 마이닝 절차를 제안합니다.
- 우리의 접근 방식은 강력한 성능을 얻습니다. 레이블이 지정된 단순화 데이터가 없으면 영어, 프랑스어 및 스페인어로 감독되는 최신 기술과 일치하거나 능가합니다. 우리는 라벨이 붙은 단순화 데이터를 통합하여 영어의 최신 기술을 더욱 향상시킵니다.
- 우리는 MUSS 사전 훈련된 모델, 의역 데이터, 마이닝 및 훈련용 코드를 출시합니다.



### 2. Related work
### 2. 연관 작업



Data-driven methods have been predominant in English sentence simplification in recent years (Alva-Manchego et al., 2020b), requiring large supervised training corpora of complex-simple aligned sentences (Wubben et al., 2012; Xu et al., 2016; Zhang and Lapata, 2017; Zhao et al., 2018; Martin et al., 2020). Methods have relied on English and Simple EnglishWikipedia with automatic sentence alignment from similar articles (Zhu et al., 2010; Coster and Kauchak, 2011; Woodsend and Lapata, 2011; Kauchak, 2013; Zhang and Lapata, 2017). Higher quality datasets have been proposed such as the Newsela corpus (Xu et al., 2015), but they are rare and come with restrictive licenses that hinder reproducibility and widespread usage.
데이터 기반 방법은 최근 몇 년 동안 영어 문장 단순화에서 지배적이었고(Alva-Manchego et al., 2020b), 복잡하고 단순하게 정렬된 문장의 대규모 지도 교육 말뭉치들이 필요합니다(Wubben et al., 2012; Xu et al., 2016). ; Zhang and Lapata, 2017; Zhao et al., 2018; Martin et al., 2020). 방법은 유사한 기사의 자동 문장 정렬 기능이 있는 영어 및 Simple EnglishWikipedia에 의존했습니다(Zhu et al., 2010; Coster 및 Kauchak, 2011; Woodsend 및 Lapata, 2011; Kauchak, 2013; Zhang 및 Lapata, 2017). Newsela 말뭉치(Xu et al., 2015)와 같은 더 높은 품질의 데이터 세트가 제안되었지만 드물고 재현성과 광범위한 사용을 방해하는 제한적인 라이선스가 함께 제공됩니다.



Simplification in other languages has been explored in Brazilian Portuguese (Aluísio et al., 2008), Spanish (Saggion et al., 2015; Štajner et al., 2015), Italian (Brunato et al., 2015; Tonelli et al., 2016), Japanese (Goto et al., 2015; Kajiwara and Komachi, 2018; Katsuta and Yamamoto, 2019), and French (Gala et al., 2020), but the lack of a large labeled parallel corpora has slowed research down. In this work, we show that a method trained on automatically mined corpora can reach state-ofthe- art results in each language.
브라질 포르투갈어(Aluísio et al., 2008), 스페인어(Saggion et al., 2015; Štajner et al., 2015), 이탈리아어(Brunato et al., 2015; Tonelli et al., 2016), 일본어(Goto et al., 2015; Kajiwara and Komachi, 2018; Katsuta and Yamamoto, 2019), 프랑스어(Gala et al., 2020)를 포함하지만 레이블이 지정된 대규모 병렬 말뭉치의 부족으로 연구 속도가 느려졌습니다. 이 작업에서 우리는 자동으로 채굴된 말뭉치에 대해 훈련된 방법이 각 언어에서 최신 결과에 도달할 수 있음을 보여줍니다.



When labeled parallel simplification data is unavailable, systems rely on unsupervised simplification techniques, often inspired from machine translation. The prevailing approach is to split a monolingual corpora into disjoint sets of complex and simple sentences using readability metrics. Then simplification models can be trained by using automatic sentence alignments (Kajiwara and Komachi, 2016, 2018), auto-encoders (Surya et al., 2019; Zhao et al., 2020), unsupervised statistical machine translation (Katsuta and Yamamoto, 2019), or back-translation (Aprosio et al., 2019). Other unsupervised simplification approaches iteratively edit the sentence until a certain simplicity criterion is reached (Kumar et al., 2020). The performance of unsupervised methods are often below their supervised counterparts. MUSS bridges the gap with supervised method and removes the need for deciding in advance how complex and simple sentences should be separated, but instead trains directly on paraphrases mined from the raw corpora.
레이블이 지정된 병렬 단순화 데이터를 사용할 수 없는 경우 시스템은 종종 기계 번역에서 영감을 받은 비지도 단순화 기술에 의존합니다. 일반적인 접근 방식은 단일 언어 말뭉치를 가독성 메트릭을 사용하여 복잡하고 간단한 문장의 분리된 세트로 분할하는 것입니다. 그런 다음 자동 문장 정렬(Kajiwara and Komachi, 2016, 2018), 자동 인코더(Surya et al., 2019; Zhao et al., 2020), 감독되지 않은 통계 기계 번역(Katsuta 및 Yamamoto, 2019)을 사용하여 단순화 모델을 훈련할 수 있습니다. ) 또는 역번역(Aprosio et al., 2019). 다른 비지도 단순화 접근 방식은 특정 단순 기준에 도달할 때까지 문장을 반복적으로 편집합니다(Kumar et al., 2020). 감독되지 않은 방법의 성능은 종종 감독되는 방법보다 낮습니다. MUSS는 지도 방법으로 간극을 메우고 복잡하고 간단한 문장을 어떻게 분리해야 하는지 미리 결정할 필요를 제거하지만, 대신 원시 말뭉치에서 추출한 의역에 대해 직접 학습합니다.



Previous work on parallel dataset mining have been used mostly in machine translation using document retrieval (Munteanu and Marcu, 2005), language models (Koehn et al., 2018, 2019), and embedding space alignment (Artetxe and Schwenk, 2019b) to create large corpora (Tiedemann, 2012; Schwenk et al., 2019). We focus on paraphrasing for sentence simplifications, which presents new challenges. Unlike machine translation, where the same sentence should be identified in two languages, we develop a method to identify varied paraphrases of sentences, that have a wider array of surface forms, including different lengths, multiple sentences, different vocabulary usage, and removal of content from more complex sentences.
병렬 데이터 세트 마이닝에 대한 이전 작업은 문서 검색(Munteanu 및 Marcu, 2005), 언어 모델(Koehn et al., 2018, 2019) 및 임베딩 공간 정렬(Artetxe 및 Schwenk, 2019b)을 사용하여 기계 번역에 주로 사용되었습니다. 큰 말뭉치(Tiedemann, 2012; Schwenk et al., 2019). 우리는 새로운 도전을 제시하는 문장 단순화를 위한 의역에 중점을 둡니다. 동일한 문장이 두 가지 언어로 식별되어야 하는 기계 번역과 달리, 다른 길이, 여러 문장, 다른 어휘 사용 및 콘텐츠 제거를 포함하여 표면 형태가 더 넓은 다양한 문장의 의역을 식별하는 방법을 개발합니다. 더 복잡한 문장에서.



Previous unsupervised paraphrasing research has aligned sentences from various parallel corpora (Barzilay and Lee, 2003) with multiple objective functions (Liu et al., 2019). Bilingual pivoting relied on MT datasets to create large databases of word-level paraphrases (Pavlick et al., 2015), lexical simplifications (Pavlick and Callison-Burch, 2016; Kriz et al., 2018), or sentence-level paraphrase corpora (Wieting and Gimpel, 2018). This has not been applied to multiple languages or to sentence-level simplification. Additionally, we use raw monolingual data to create our paraphrase corpora instead of relying on parallel MT datasets.
이전의 감독되지 않은 의역 연구에서는 다양한 병렬 말뭉치(Barzilay and Lee, 2003)의 문장을 다중 목적 함수(Liu et al., 2019)와 정렬했습니다. 이중 언어 피벗은 MT 데이터 세트에 의존하여 단어 수준 의역(Pavlick et al., 2015), 어휘 단순화(Pavlick and Callison-Burch, 2016; Kriz et al., 2018) 또는 문장 수준 의역 말뭉치( Wieting 및 Gimpel, 2018). 이것은 여러 언어 또는 문장 수준 단순화에 적용되지 않았습니다. 또한 병렬 MT 데이터 세트에 의존하는 대신 원시 단일 언어 데이터를 사용하여 의역 말뭉치를 생성합니다.



Table 1: Statistics on our mined paraphrase training corpora compared to standard simplification datasets (see section 4.3 for more details).
표 1: 표준 단순화 데이터 세트와 비교하여 마이닝된 의역 훈련 말뭉치에 대한 통계(자세한 내용은 섹션 4.3 참조).



# 3 Method

In this section we describe MUSS, our approach to mining paraphrase data and training controllable simplification models on paraphrases.
이 섹션에서는 의역 데이터 마이닝 및 의역에 대한 제어 가능한 단순화 모델 교육에 대한 접근 방식인 MUSS에 대해 설명합니다.



## 3.1 Mining Paraphrases in Many Languages
## 3.1 다양한 언어로 된 마이닝 의역


#### Sequence Extraction
Simplification consists of multiple rewriting operations, some of which span multiple sentences (e.g. sentence splitting or fusion). To allow such operations to be represented in our paraphrase data, we extract chunks of texts composed of multiple sentences, we refer to these small pieces of text by sequences.
시퀀스 추출
단순화는 여러 재작성 작업으로 구성되며 그 중 일부는 여러 문장에 걸쳐 있습니다(예: 문장 분할 또는 융합). 이러한 작업을 의역 데이터에 표시할 수 있도록 여러 문장으로 구성된 텍스트 덩어리를 추출하고 이러한 작은 텍스트 조각을 시퀀스로 참조합니다.



We extract such sequences by first tokenizing a document into individual sentences {s1, s2, ..., sn} using the NLTK sentence tokenizer (Bird and Loper, 2004). We then extract sequences of adjacent sentences with maximum length of 300 characters:
e.g. {[s1], [s1,s2], [s1, ..., sk], [s2], [s2, s3], ...}.
먼저 NLTK 문장 토크나이저를 사용하여 문서를 개별 문장 {s1, s2, ..., sn}으로 토큰화하여 이러한 시퀀스를 추출합니다(Bird and Loper, 2004). 그런 다음 최대 300자 길이의 인접 문장 시퀀스를 추출합니다.
예를 들어 {[s1], [s1,s2], [s1, ..., sk], [s2], [s2, s3], ...}.



We can thus align two sequences that contain a different number of sentences, and represent sentence splitting or sentence fusion operations.
따라서 다른 수의 문장을 포함하는 두 시퀀스를 정렬하고 문장 분할 또는 문장 융합 작업을 나타낼 수 있습니다.



These sequences are further filtered to remove noisy sequences with more than 10% punctuation characters and sequences with low language model probability according to a 3-gram Kneser-Ney language model trained with kenlm (Heafield, 2011) on Wikipedia.
Wikipedia의 kenlm(Heafield, 2011)으로 훈련된 3-gram Kneser-Ney 언어 모델에 따라 이러한 시퀀스는 구두점 문자가 10% 이상인 잡음이 있는 시퀀스와 언어 모델 확률이 낮은 시퀀스를 제거하기 위해 추가로 필터링됩니다.



We extract these sequences from CCNet (Wenzek et al., 2019), an extraction of Common Crawl (an open source snapshot of the web) that has been split into different languages using fasttext language identification (Joulin et al., 2017) and various language modeling filtering techniques to identify high quality, clean sentences. For English and French, we extract 1 billion sequences from CCNet. For Spanish we extract 650 millions sequences, the maximum for this language in CCNet after filtering out noisy text.
CCNet(Wenzek et al., 2019), Fasttext 언어 식별(Joulin et al., 2017)을 사용하여 여러 언어로 분할된 Common Crawl(웹의 오픈 소스 스냅샷) 추출 및 다양한 고품질의 깨끗한 문장을 식별하기 위한 언어 모델링 필터링 기술. 영어와 프랑스어의 경우 CCNet에서 10억 개의 시퀀스를 추출합니다. 스페인어의 경우 잡음이 있는 텍스트를 필터링한 후 CCNet에서 이 언어의 최대값인 6억 5천만 개의 시퀀스를 추출합니다.



#### Creating a Sequence Index Using Embeddings
To automatically mine our paraphrase corpora, we first compute n-dimensional embeddings for each extracted sequence using LASER (Artetxe and Schwenk, 2019b). LASER provides joint multilingual sentence embeddings in 93 languages that have been successfully applied to the task of bilingual bitext mining (Schwenk et al., 2019). In this work, we show that LASER can also be used to mine monolingual paraphrase datasets.
임베딩을 사용하여 시퀀스 인덱스 만들기
패러프레이즈 말뭉치를 자동으로 마이닝하기 위해 먼저 LASER를 사용하여 추출된 각 시퀀스에 대한 n차원 임베딩을 계산합니다(Artetxe 및 Schwenk, 2019b). LASER는 이중 언어 바이텍스트 마이닝 작업에 성공적으로 적용된 93개 언어로 공동 다국어 문장 임베딩을 제공합니다(Schwenk et al., 2019). 이 작업에서 우리는 LASER가 단일 언어 의역 데이터 세트를 마이닝하는 데에도 사용될 수 있음을 보여줍니다.



#### Mining Paraphrases
After computing the embeddings for each language, we index them for fast nearest neighbor search using faiss.
마이닝 의역
각 언어에 대한 임베딩을 계산한 후 faiss를 사용하여 빠른 최근접이웃 검색을 위해 이를 인덱싱합니다.



Each of these sequences is then used as a query qi against the billion-scale index that returns a set of top-8 nearest neighbor sequences according to the semantic LASER embedding space using L2 distance, resulting in a set of candidate paraphrases are {ci,1, ..., ci,8}
그런 다음 각 시퀀스는 L2 거리를 사용하는 의미론적 LASER 임베딩 공간에 따라 상위 8개 최근접 이웃 시퀀스 세트를 반환하는 10억 규모 인덱스에 대한 쿼리 qi로 사용되며, 결과적으로 후보 패러프레이즈 세트는 {ci,1, ..., ci,8}



We then use an upper bound on L2 distance and a margin criterion following (Artetxe and Schwenk, 2019a) to filter out sequences with low similarity. We refer the reader to Appendix Section A.1 for technical details.
그런 다음 L2 거리의 상한선과 여백 기준(Artetxe 및 Schwenk, 2019a)을 사용하여 유사성이 낮은 시퀀스를 필터링합니다. 기술적인 세부 사항은 부록 A.1을 참조하십시오.



The resulting nearest neighbors constitute a set of aligned paraphrases of the query sequence:
{(qi, ci,1), ..., (qi, ci,j)}.
We finally apply poor alignment filters. We remove sequences that are almost identical with character-level Levenshtein distance ≤ 20%, when they are contained in one another, or when they were extracted from two overlapping sliding windows of the same original document.
결과적으로 가장 가까운 이웃은 쿼리 시퀀스의 정렬된 패러프레이즈 세트를 구성합니다.
{(qi, ci,1), ..., (qi, ci,j)}.
마지막으로 형편없는 정렬 필터를 적용합니다. 문자 수준 리벤슈테인 거리 ≤ 20%로 거의 동일한 시퀀스가 서로 포함되어 있거나 동일한 원본 문서의 두 개의 겹치는 슬라이딩 창에서 추출된 경우 시퀀스를 제거합니다.



We report statistics of the mined corpora in English, French and Spanish in Table 1, and qualitative examples of the resulting mined paraphrases in Appendix Table 6. Models trained on the resulting mined paraphrases obtain similar performance than models trained on existing paraphrase datasets (cf. Appendix Section D).
우리는 표 1에 영어, 프랑스어 및 스페인어로 마이닝된 말뭉치의 통계를 보고하고 부록 표 6에 결과 마이닝된 의역의 정성적 예를 보고합니다. 결과 마이닝된 의역에 대해 훈련된 모델은 기존의 다른 표현 데이터세트에서 훈련된 모델과 유사한 성능을 얻습니다(cf. 부록 섹션 D).



### 3.2 Simplifying with ACCESS
### 3.2 ACCESS로 단순화



In this section we describe how we adapt ACCESS (Martin et al., 2020) to train controllable models on mined paraphrases, instead of labeled parallel simplifications.
이 섹션에서는 레이블이 지정된 병렬 단순화 대신 마이닝된 의역에 대한 제어 가능한 모델을 훈련하기 위해 ACCESS(Martin et al., 2020)를 적용하는 방법을 설명합니다.



ACCESS is a method to make any seq2seq model controllable by conditioning on simplification-specific control tokens. We apply it on our seq2seq pretrained transformer models based on the BART (Lewis et al., 2019) architecture (see next subsection).
ACCESS는 단순화별 제어 토큰을 조건으로 seq2seq 모델을 제어 가능하게 만드는 방법입니다. BART(Lewis et al., 2019) 아키텍처를 기반으로 하는 seq2seq 사전 훈련된 변환기 모델에 적용합니다(다음 하위 섹션 참조).



#### Training with Control Tokens
At training time, the model is provided with control tokens that give oracle information on the target sequence, such as the amount of compression of the target sequence relative to the source sequence (length control). For example, when the target sequence is 80% of the length of the original sequence, we provide the <NumChars_80%> control token. At inference time we can then control the generation by selecting a given target control value. We adapt the original Levenshtein similarity control to only consider replace operations but otherwise use the same controls as Martin et al. (2020). The controls used are therefore character length ratio, replace-only Levenshtein similarity, aggregated word frequency ratio, and dependency tree depth ratio. For instance we will prepend to every source in the training set the following 4 control tokens with samplespecific values, so that the model learns to rely on them: <NumChars_XX%> <LevSim_YY%> <WordFreq_ZZ%> <DepTreeDepth_TT%>. We refer the reader to the original paper (Martin et al., 2020) and Appendix A.2 for details on ACCESS and how those control tokens are computed.
#### 제어 토큰을 사용한 교육
훈련 시 모델은 소스 시퀀스에 대한 타겟 시퀀스의 압축 정도(길이 제어)와 같은 타겟 시퀀스에 대한 오라클 정보를 제공하는 제어 토큰과 함께 제공됩니다. 예를 들어 타겟 시퀀스가 ​​원래 시퀀스 길이의 80%인 경우 <NumChars_80%> 제어 토큰을 제공합니다. 추론 시간에 우리는 주어진 목표 제어 값을 선택하여 생성을 제어할 수 있습니다. 우리는 원래 Levenshtein 유사성 제어를 적용하여 교체 작업만 고려하지만 그렇지 않으면 Martin et al과 동일한 제어를 사용합니다. (2020). 따라서 사용되는 컨트롤은 문자 길이 비율, 교체 전용 Levenshtein 유사성, 집계된 단어 빈도 비율 및 종속성 트리 깊이 비율입니다. 예를 들어 훈련 세트의 모든 소스 앞에 샘플별 값이 있는 다음 4개의 제어 토큰을 추가하여 모델이 <NumChars_XX%> <LevSim_YY%> <WordFreq_ZZ%> <DepTreeDepth_TT%>에 의존하도록 학습합니다. ACCESS 및 이러한 제어 토큰이 계산되는 방법에 대한 자세한 내용은 독자에게 원본 문서(Martin et al., 2020) 및 부록 A.2를 참조하십시오.



#### Selecting Control Values at Inference
Once the model has been trained with oracle controls, we can adjust the control tokens to obtain the desired type of simplifications. Indeed, sentence simplification often depends on the context and target audience (Martin et al., 2020). For instance shorter sentences are more adapted to people with cognitive disabilities, while using more frequent words are useful to second language learners. It is therefore important that supervised and unsupervised simplification systems can be adapted to different conditions: (Kumar et al., 2020) do so by choosing a set of operation-specific weights of their unsupervised simplification model for each evaluation dataset, (Surya et al., 2019) select different models using SARI on each validation set. Similarly, we set the 4 control hyper-parameters of ACCESS using SARI on each validation set and keep them fixed for all samples in the test set.4. These 4 control hyper-parameters are intuitive and easy to interpret: when no validation set is available, they can also be set using prior knowledge on the task and 4Details in Appendix A.2 still lead to solid performance (cf. Appendix C).
#### 추론에서 제어 값 선택
모델이 oracle 컨트롤로 훈련되면 원하는 유형의 단순화를 얻기 위해 컨트롤 토큰을 조정할 수 있습니다. 실제로 문장 단순화는 종종 컨텍스트와 대상 청중에 따라 다릅니다(Martin et al., 2020). 예를 들어, 더 짧은 문장은 인지 장애가 있는 사람들에게 더 적합하지만 더 자주 단어를 사용하는 것은 제 2 언어 학습자에게 유용합니다. 따라서 지도 및 비지도 단순화 시스템이 서로 다른 조건에 적용될 수 있다는 것이 중요합니다. (Kumar et al., 2020) 각 평가 데이터 세트에 대한 비지도 단순화 모델의 작업별 가중치 세트를 선택하여 그렇게 합니다(Surya et al. ., 2019) 각 검증 세트에서 SARI를 사용하여 다른 모델을 선택합니다. 유사하게, 우리는 각 검증 세트에서 SARI를 사용하여 ACCESS의 4가지 제어 하이퍼 파라미터를 설정하고 테스트 세트의 모든 샘플에 대해 고정된 상태로 유지합니다.4. 이 4가지 제어 하이퍼 파라미터는 직관적이고 해석하기 쉽습니다. 사용 가능한 유효성 검사 세트가 없는 경우 작업에 대한 사전 지식을 사용하여 설정할 수도 있으며 부록 A.2의 4세부 사항은 여전히 ​​견고한 성능으로 이어집니다(부록 C 참조).



### 3.3 Leveraging Unsupervised Pretraining
### 3.3 비지도 사전 훈련 활용


We combine our controllable models with unsupervised pretraining to further extend our approach to text simplification. For English, we finetune the pretrained generative model BART (Lewis et al., 2019) on our newly created training corpora. BART is a pretrained sequence-to-sequence model that can be seen as a generalization of other recent pretrained models such as BERT (Devlin et al., 2018). For non-English, we use its multilingual generalization MBART (Liu et al., 2020), which was pretrained on 25 languages.
우리는 제어 가능한 모델을 감독되지 않은 사전 훈련과 결합하여 텍스트 단순화에 대한 접근 방식을 더욱 확장합니다. 영어의 경우 새로 생성된 훈련 말뭉치에서 사전 훈련된 생성 모델 BART(Lewis et al., 2019)를 미세 조정합니다. BART는 BERT(Devlin et al., 2018)와 같은 다른 최근 사전 훈련된 모델의 일반화로 볼 수 있는 사전 훈련된 시퀀스 대 시퀀스 모델입니다. 영어가 아닌 경우 25개 언어로 사전 훈련된 다국어 일반화 MBART(Liu et al., 2020)를 사용합니다.


## 4. Experimental Setting
## 4. 실험 세팅


We assess the performance of our approach on three languages: English, French, and Spanish. We detail our experimental procedure for mining and training in Appendix Section A. In all our experiments, we report scores on the test sets averaged over 5 random seeds with 95% confidence intervals.
우리는 영어, 프랑스어 및 스페인어의 세 가지 언어에 대한 접근 방식의 성능을 평가합니다. 부록 A에서 마이닝 및 훈련을 위한 실험 절차를 자세히 설명합니다. 모든 실험에서 95% 신뢰 구간으로 5개의 무작위 시드에 대해 평균을 낸 테스트 세트의 점수를 보고합니다.



### 4.1 Baselines
### 4.1 기준선



In addition to comparisons with previous works, we implement and hereafter describe multiple baselines to assess the performance of our models, especially for French and Spanish where no previous simplification systems are available.
이전 작업과의 비교 외에도, 특히 이전 단순화 시스템을 사용할 수 없는 프랑스어 및 스페인어의 경우 모델의 성능을 평가하기 위해 여러 기준선을 구현하고 설명합니다.



#### Identity
The entire original sequence is kept unchanged and used as the simplification.
#### 동일성
전체 원본 시퀀스는 변경되지 않고 유지되고 단순화로 사용됩니다.



#### Truncation
The original sequence is truncated to the first 80% words. It is a strong baseline according to standard simplification metrics.
#### 잘림
원래 시퀀스는 처음 80% 단어로 잘립니다. 표준 단순화 메트릭에 따른 강력한 기준선입니다.


#### Pivot
We use machine translation to provide a baseline for languages for which no simplification corpus is available. The source non-English sentence is translated to English, simplified with our best supervised English simplification system, and then translated back into the source language. For French and Spanish translation, we use CCMATRIX (Schwenk et al., 2019) to train Transformer models with LayerDrop (Fan et al., 2019). We use the BART+ACCESS supervised model trained on MINED + WikiLarge as the English simplification model. While pivoting creates potential errors, recent improvements of translation systems on high resource languages make this a strong baseline.
#### 피벗
우리는 기계 번역을 사용하여 단순화 말뭉치를 사용할 수 없는 언어에 대한 기준을 제공합니다. 영어가 아닌 소스 문장은 영어로 번역되고, 최고의 감독을 받는 영어 간체 시스템으로 단순화된 다음 다시 소스 언어로 번역됩니다. 프랑스어 및 스페인어 번역의 경우 CCMATRIX(Schwenk et al., 2019)를 사용하여 LayerDrop(Fan et al., 2019)으로 Transformer 모델을 훈련합니다. 우리는 영어 단순화 모델로 MINED + WikiLarge에서 훈련된 BART+ACCESS 지도 모델을 사용합니다. 피봇팅은 잠재적인 오류를 생성하지만, 최근에 자원이 풍부한 언어에 대한 번역 시스템의 개선은 이것을 강력한 기준으로 만듭니다.



#### Gold Reference
We report gold reference scores for ASSET and TurkCorpus as multiple references are available. We compute scores in a leave-oneout scenario where each reference is evaluated against all others. The scores are then averaged over all references.
#### 골드 레퍼런스
여러 참조를 사용할 수 있으므로 ASSET 및 TurkCorpus에 대한 골드 참조 점수를 보고합니다. 우리는 각 참조가 다른 모든 참조에 대해 평가되는 1대1 시나리오에서 점수를 계산합니다. 그런 다음 점수는 모든 참조에 대해 평균을 냅니다.



## 4.2 Evaluation Metrics
## 4.2 평가 매트릭



We evaluate with the standard metrics SARI and FKGL. We report BLEU (Papineni et al., 2002) only in Appendix Table 10 due its dubious suitability for sentence simplification (Sulem et al., 2018).
우리는 표준 메트릭 SARI 및 FKGL로 평가합니다. BLEU(Papineni et al., 2002)는 문장 단순화에 대한 모호한 적합성 때문에 부록 표 10에만 보고합니다(Sulem et al., 2018).



#### SARI
Sentence simplification is commonly evaluated with SARI (Xu et al., 2016), which compares model-generated simplifications with the source sequence and gold references. It averages F1 scores for addition, keep, and deletion operations. We compute SARI with the EASSE simplification evaluation suite (Alva-Manchego et al., 2019).5
#### 사리
문장 단순화는 일반적으로 모델 생성 단순화를 소스 시퀀스 및 골드 참조와 비교하는 SARI(Xu et al., 2016)로 평가됩니다. 추가, 유지 및 삭제 작업에 대한 평균 F1 점수입니다. EASSE 단순화 평가 제품군으로 SARI를 계산합니다(Alva-Manchego et al., 2019).5


5 We use the latest version of SARI implemented in EASSE (Alva-Manchego et al., 2019) which fixes bugs and inconsistencies from the traditional implementation. We thus recompute scores from previous systems that we compare to, by using the system predictions provided by the respective authors available in EASSE.
5 우리는 EASSE(Alva-Manchego et al., 2019)에 구현된 최신 버전의 SARI를 사용하여 기존 구현의 버그와 불일치를 수정합니다. 따라서 EASSE에서 사용할 수 있는 각 작성자가 제공한 시스템 예측을 사용하여 비교하는 이전 시스템의 점수를 다시 계산합니다.


#### FKGL
We report readability scores using the Flesch-Kincaid Grade Level (FKGL) (Kincaid et al., 1975), a linear combination of sentence lengths and word lengths. FKGL was designed to be used on English texts only, we do not report it on French and Spanish.
#### FKGL
문장 길이와 단어 길이의 선형 조합인 FKGL(Flesch-Kincaid Grade Level)(Kincaid et al., 1975)을 사용하여 가독성 점수를 보고합니다. FKGL은 영어 텍스트에서만 사용하도록 설계되었으며 프랑스어 및 스페인어에서는 보고하지 않습니다.



## 4.3 Training Data
## 4.3 학습 데이터



For all languages we use the mined data described in Table 1 as training data. We show that training with additional labeled simplification data leads to even better performance for English. We use the labeled datasets WikiLarge (Zhang and Lapata, 2017) and Newsela (Xu et al., 2015). WikiLarge is composed of 296k simplification pairs automatically aligned from English Wikipedia and Simple EnglishWikipedia. Newsela is a collection of news articles with  professional simplifications, aligned into 94k simplification pairs by Zhang and Lapata (2017).6
모든 언어에 대해 표 1에 설명된 마이닝된 데이터를 훈련 데이터로 사용합니다. 레이블이 지정된 추가 단순화 데이터를 사용한 학습이 영어에 대해 훨씬 더 나은 성능으로 이어진다는 것을 보여줍니다. 레이블이 지정된 데이터 세트 WikiLarge(Zhang and Lapata, 2017)와 Newsela(Xu et al., 2015)를 사용합니다. WikiLarge는 영문 Wikipedia와 Simple EnglishWikipedia에서 자동으로 정렬된 296k 단순화 쌍으로 구성됩니다. Newsela는 Zhang과 Lapata(2017)가 94k 단순화 쌍으로 정렬한 전문적인 단순화가 포함된 뉴스 기사 모음입니다.6



## 4.4 Evaluation Data
## 4.4 평가 데이터



#### English
We evaluate our English models on ASSET (Alva-Manchego et al., 2020a), TurkCorpus (Xu et al., 2016) and Newsela (Xu et al., 2015). TurkCorpus and ASSET were created using the same 2000 valid and 359 test source sentences. TurkCorpus contains 8 reference simplifications per source sentence and ASSET contains 10 references per source. ASSET is a generalization of TurkCorpus with a  more varied set of rewriting operations, and considered simpler by human judges (Alva-Manchego et al., 2020a). For Newsela, we evaluate on the split from (Zhang and Lapata, 2017), which includes 1129 validation and 1077 test sentence pairs.
#### 영어
ASSET(Alva-Manchego et al., 2020a), TurkCorpus(Xu et al., 2016) 및 Newsela(Xu et al., 2015)에서 영어 모델을 평가합니다. TurkCorpus와 ASSET은 동일한 2000개의 유효 문장과 359개의 테스트 소스 문장을 사용하여 생성되었습니다. TurkCorpus는 소스 문장당 8개의 참조 단순화를 포함하고 ASSET은 소스당 10개의 참조를 포함합니다. ASSET은 보다 다양한 재작성 작업 세트로 TurkCorpus를 일반화한 것으로, 인간 심사 위원이 더 간단하다고 간주합니다(Alva-Manchego et al., 2020a). Newsela의 경우 1129개의 검증과 1077개의 테스트 문장 쌍을 포함하는 (Zhang and Lapata, 2017)의 분할을 평가합니다.



#### French
For French, we use the ALECTOR dataset (Gala et al., 2020) for evaluation. ALECTOR is a collection of literary (tales, stories) and scientific (documentary) texts along with their manual document-level simplified versions. These documents were extracted from material available to French primary school pupils. We split the dataset in 450 validation and 416 test sentence pairs (see Appendix A.3 for details).
#### 프랑스어
프랑스어의 경우 평가를 위해 ALECTOR 데이터 세트(Gala et al., 2020)를 사용합니다. ALECTOR는 수동 문서 수준의 단순화된 버전과 함께 문학(이야기, 이야기) 및 과학(다큐멘터리) 텍스트의 모음입니다. 이 문서는 프랑스 초등학교 학생들이 사용할 수 있는 자료에서 추출했습니다. 데이터 세트를 450개의 검증과 416개의 테스트 문장 쌍으로 분할했습니다(자세한 내용은 부록 A.3 참조).



#### Spanish
For Spanish we use the Spanish part of Newsela (Xu et al., 2015). We use the alignments from (Aprosio et al., 2019), composed of 2794 validation and 2795 test sentence pairs. Even though sentences were aligned using the CATS simplification alignment tool (Štajner et al., 2018), some alignment errors remain and automatic scores should be taken with a pinch of salt.
#### 스페인어
스페인어의 경우 Newsela의 스페인어 부분을 사용합니다(Xu et al., 2015). 우리는 2794개의 검증과 2795개의 테스트 문장 쌍으로 구성된 (Aprosio et al., 2019)의 정렬을 사용합니다. CATS 단순화 정렬 도구(Štajner et al., 2018)를 사용하여 문장을 정렬했지만 일부 정렬 오류가 남아 있고 약간의 데이터로 자동 점수를 가져와야 합니다.



# Appendices

## A Experimental details

Figure 2: Sentence Simplification Models for Any Language without Simplification Data.
Sentences from the web are used to create a large scale index that allows mining millions of paraphrases. Subsequently, we finetune pretrained models augmented with controllable mechanisms on the paraphrase corpora to achieve sentence simplification models in any language.
그림 2: 단순화 데이터가 없는 모든 언어에 대한 문장 단순화 모델.
웹의 문장은 수백만 개의 의역을 마이닝할 수 있는 대규모 색인을 만드는 데 사용됩니다. 그 후, 우리는 모든 언어에서 문장 단순화 모델을 달성하기 위해 paraphrase corpora에 대한 제어 가능한 메커니즘으로 보강된 사전 훈련된 모델을 미세 조정합니다.



## A.1 Mining Details
### Sequence Extraction
We only consider documents from the HEAD split in CCNet— this represents the third of the data with the best perplexity using a language model.
#### 서열 추출
CCNet에서 HEAD 분할의 문서만 고려합니다. 이는 언어 모델을 사용하여 가장 난해한 데이터의 3분의 1을 나타냅니다.



#### Paraphrase Mining
We compute LASER embeddings of dimension 1024 and reduce dimensionality with a 512 PCA followed by random rotation. We further compress them using 8 bit scalar quantization. The compressed embeddings are then stored in a faiss inverted file index with 32,768 cells (nprobe=16). These embeddings are used to mine pairs of paraphrases. We return the top-8 nearest neighbors, and keep those with L2 distance lower than 0.05 and relative distance compared to other top-8 nearest neighbors lower than 0.6.
#### 패러프레이즈 마이닝
차원 1024의 LASER 임베딩을 계산하고 512 PCA와 무작위 회전으로 차원을 줄입니다. 8비트 스칼라 양자화를 사용하여 더 압축합니다. 압축된 임베딩은 그런 다음 32,768개의 셀(nprobe=16)이 있는 faiss 역 파일 인덱스에 저장됩니다. 이러한 임베딩은 의역 쌍을 마이닝하는 데 사용됩니다. 우리는 상위 8개 최근접이웃을 반환하고 L2 거리가 0.05보다 작고 다른 상위 8개 최근접이웃과 비교한 상대 거리가 0.6보다 작은 것을 유지합니다.



#### Paraphrases Filtering
The resulting paraphrases are filtered to remove almost identical paraphrases by enforcing a case-insensitive character-level Levenshtein distance (Levenshtein, 1966) greater or equal to 20%. We remove paraphrases that come from the same document to avoid aligning sequences that overlapped each other in the text. We also remove paraphrases where one of the sequence is contained in the other. We further filter out any sequence that is present in our evaluation datasets.
#### 패러프레이즈 필터링
결과 의역은 대소문자를 구분하지 않는 문자 수준 Levenshtein distance(Levenshtein, 1966)를 20% 이상으로 적용하여 거의 동일한 의역을 제거하도록 필터링됩니다. 텍스트에서 서로 겹치는 시퀀스가 정렬되지 않도록 동일한 문서에서 나온 의역을 제거합니다. 또한 시퀀스 중 하나가 다른 시퀀스에 포함된 의역을 제거합니다. 평가 데이터 세트에 있는 모든 시퀀스를 추가로 필터링합니다.



## A.2 Training Details
### Seq2Seq training
#### We implement our models
with fairseq (Ott et al., 2019). All our models are Transformers (Vaswani et al., 2017) based on the BART Large architecture (388M parameters), keeping the optimization procedure and hyper-parameters fixed to those used in the original implementation (Lewis et al., 2019)7. We either randomly initialize weights for the standard sequence-to-sequence experiments or initialize with pretrained BART for the BART experiments. When initializing the weights randomly, we use a learning rate of 3.10-4 versus the original 3.10-5 when finetuning BART. For a given seed, the model is trained on 8 Nvidia V100 GPUs during approximately 10 hours.
#### 우리는 우리의 모델을 구현합니다
Fairseq 사용(Ott et al., 2019). 우리의 모든 모델은 BART Large 아키텍처(388M 매개변수)를 기반으로 하는 Transformers(Vaswani et al., 2017)로, 최적화 절차와 하이퍼 매개변수를 원래 구현에 사용된 것에 고정된 상태로 유지합니다(Lewis et al., 2019)7. 표준 시퀀스 간 실험을 위해 가중치를 무작위로 초기화하거나 BART 실험을 위해 사전 훈련된 BART로 초기화합니다. 가중치를 무작위로 초기화할 때 BART를 미세 조정할 때 원래 3.10-5에 비해 학습률 3.10-4를 사용합니다. 주어진 시드에 대해 모델은 약 10시간 동안 8개의 Nvidia V100 GPU에서 학습됩니다.



#### Controllable Generation
For controllable generation, we use the open-source ACCESS implementation (Martin et al., 2018). We use the same control parameters as the original paper, namely length, Levenshtein similarity, lexical complexity, and syntactic complexity.8
#### 제어 가능한 생성
제어 가능한 생성을 위해 오픈 소스 ACCESS 구현을 사용합니다(Martin et al., 2018). 우리는 길이, Levenshtein 유사성, 어휘 복잡성 및 구문 복잡성과 같은 원본 논문과 동일한 제어 매개변수를 사용합니다.8


8 We modify the Levenshtein similarity parameter to only consider replace operations, by assigning a 0 weight to insertions and deletions. This change helps decorrelate the Levenshtein similarity control token from the length control token and produced better results in preliminary experiments.
8 우리는 삽입 및 삭제에 0 가중치를 할당하여 교체 작업만 고려하도록 Levenshtein 유사성 매개변수를 수정합니다. 이 변경은 Levenshtein 유사성 제어 토큰을 길이 제어 토큰과 상관관계를 해제하는 데 도움이 되며 예비 실험에서 더 나은 결과를 생성했습니다.



As mentioned in Section ”Simplifying with ACCESS”, we select the 4 ACCESS hyperparameters using SARI on the validation set. We use zero-order optimization with the NEVERGRAD library (Rapin and Teytaud, 2018). We use the One- PlusOne optimizer with a budget of 64 evaluations (approximately 1 hour of optimization on a single GPU). The hyper-parameters are contained in the [0.2, 1.5] interval.
섹션 "ACCESS로 단순화"에서 언급했듯이 검증 세트에서 SARI를 사용하여 4개의 ACCESS 하이퍼파라미터를 선택합니다. NEVERGRAD 라이브러리와 함께 0차 최적화를 사용합니다(Rapin and Teytaud, 2018). 우리는 64개 평가의 예산으로 One-PlusOne 옵티마이저를 사용합니다(단일 GPU에서 약 1시간 최적화). 하이퍼 매개변수는 [0.2, 1.5] 간격에 포함됩니다.



#### Translation Model for Pivot Baseline
For the pivot baseline we train models on ccMatrix (Schwenk et al., 2019). Our models use the Transformer architecture with 240 million parameters with LayerDrop (Fan et al., 2019). We train for 36 hours on 8 GPUs following the suggested parameters in Ott et al. (2019).
#### 피벗 베이스라인에 대한 번역 모델
피벗 기준선의 경우 ccMatrix에서 모델을 훈련합니다(Schwenk et al., 2019). 우리 모델은 LayerDrop과 함께 2억 4천만 개의 매개변수가 있는 Transformer 아키텍처를 사용합니다(Fan et al., 2019). Ott et al.에서 제안한 매개변수에 따라 8개의 GPU에서 36시간 동안 훈련합니다. (2019).



#### Gold Reference Baseline
To avoid creating a discrepancy in terms of number of references be-tween the gold reference scores, where we leave one reference out, and when we evaluate the models with all references, we compensate by duplicating one of the other references at random so that the total number of references is unchanged.
#### 골드 참조 기준선
하나의 참조를 제외하고 모든 참조가 포함된 모델을 평가할 때 골드 참조 점수 사이에 참조 수의 불일치가 발생하지 않도록 다른 참조 중 하나를 무작위로 복제하여 보상합니다. 총 참조 수는 변경되지 않습니다.



## C Set ACCESS Control Parameters Without Parallel Data
## C 병렬 데이터 없이 ACCESS 제어 매개변수 설정


In our experiments we adjusted our model to the different dataset conditions by selecting our ACCESS control tokens with SARI on each validation set. When no such parallel validation set exists, we show that strong performance can still be obtained by using prior knowledge for the given downstream application. This can be done by setting all 4 ACCESS control hyper-parameters to an intuitive guess of the desired compression ratio.
실험에서 우리는 각 유효성 검사 세트에서 SARI가 있는 ACCESS 제어 토큰을 선택하여 다양한 데이터 세트 조건에 맞게 모델을 조정했습니다. 그러한 병렬 검증 세트가 존재하지 않을 때 우리는 주어진 다운스트림 애플리케이션에 대한 사전 지식을 사용하여 여전히 강력한 성능을 얻을 수 있음을 보여줍니다. 이것은 4개의 모든 ACCESS 제어 하이퍼 매개변수를 원하는 압축 비율의 직관적인 추측으로 설정하여 수행할 수 있습니다.



To illustrate this for the considered evaluation datasets, we first independently sample 50 source sentences and 50 random unaligned simple sentences from each validation set. These two groups of non-parallel sentences are used to approximate the character-level compression ratio between complex and simplified sentences. We do so by dividing the average length of the simplified sentences by the average length of the 50 source sentences. We finally use this approximated compression ratio as the value of all 4 ACCESS hyper-parameters. In practice, we obtain the following approximations: ASSET = 0.8, TurkCorpus = 0.95, and Newsela = 0.4 (rounded to 0.05). Results in Table 7 show that the resulting model performs very close to when we adjust the ACCESS hyper-parameters using SARI on the complete validation set.
고려된 평가 데이터 세트에 대해 이를 설명하기 위해 먼저 각 유효성 검사 세트에서 50개의 소스 문장과 50개의 정렬되지 않은 무작위 단순 문장을 독립적으로 샘플링합니다. 평행하지 않은 문장의 이 두 그룹은 복잡한 문장과 간단한 문장 사이의 문자 수준 압축 비율을 근사하는 데 사용됩니다. 우리는 단순화된 문장의 평균 길이를 50개의 원본 문장의 평균 길이로 나눔으로써 그렇게 합니다. 마지막으로 이 근사 압축 비율을 4개의 ACCESS 하이퍼 매개변수 값으로 사용합니다. 실제로 ASSET = 0.8, TurkCorpus = 0.95, Newsela = 0.4(0.05로 반올림) 근사치를 얻습니다. 표 7의 결과는 결과 모델이 완전한 검증 세트에서 SARI를 사용하여 ACCESS 하이퍼 매개변수를 조정할 때와 매우 유사하게 수행됨을 보여줍니다.