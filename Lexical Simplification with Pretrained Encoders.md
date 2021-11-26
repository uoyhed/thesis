# Lexical Simplification with Pretrained Encoders
# 사전 훈련된 인코더를 사용한 어휘 단순화



## Abstract

Lexical simplification (LS) aims to replace complex words in a given sentence with their simpler alternatives of equivalent meaning. Recently unsupervised lexical simplification approaches only rely on the complex word itself regardless of the given sentence to generate candidate substitutions, which will inevitably produce a large number of spurious candidates. We present a simple LS approach that makes use of the Bidirectional Encoder Representations from Transformers (BERT) which can consider both the given sentence and the complex word during generating candidate substitutions for the complex word. Specifically, we mask the complex word of the original sentence for feeding into the BERT to predict the masked token. The predicted results will be used as candidate substitutions. Despite being entirely unsupervised, experimental results show that our approach obtains obvious improvement compared with these baselines leveraging linguistic databases and parallel corpus, outperforming the state-of-the-art by more than 12 Accuracy points on three well-known benchmarks.
어휘 단순화(LS)는 주어진 문장의 복잡한 단어를 동등한 의미의 더 간단한 대안으로 대체하는 것을 목표로 합니다. 최근에는 감독되지 않은 어휘 단순화 접근 방식은 주어진 문장에 관계없이 복잡한 단어 자체에만 의존하여 후보 대체를 생성하므로 필연적으로 많은 수의 가짜 후보가 생성됩니다. 우리는 복잡한 단어에 대한 후보 대체 단어를 생성하는 동안 주어진 문장과 복잡한 단어를 모두 고려할 수 있는 BERT(Bidirectional Encoder Representations from Transformers)를 사용하는 간단한 LS 접근 방식을 제시합니다. 특히, 마스킹된 토큰을 예측하기 위해 BERT에 공급하기 위해 원래 문장의 복잡한 단어를 마스킹합니다. 예측된 결과는 후보 대체 단어로 사용됩니다. 완전히 감독되지 않았음에도 불구하고 실험 결과에 따르면 우리의 접근 방식은 언어 데이터베이스 및 병렬 코퍼스를 활용하는 이러한 기준선과 비교하여 명백한 개선을 얻었으며 3개의 잘 알려진 벤치마크에서 12개 이상의 정확도 포인트로 최첨단을 능가합니다.



## 1. Introduction



Lexical Simplification (LS) aims at replacing complex words with simpler alternatives, which can help various groups of people, including children (De Belder and Moens 2010), non-native speakers (Paetzold and Specia 2016), people with cognitive disabilities (Feng 2009; Saggion 2017), to understand text better. The popular LS systems still predominantly use a set of rules for substituting complex words with their frequent synonyms from carefully handcrafted databases (e.g., WordNet) or automatically induced from comparable corpora (Devlin and Tait 1998; De Belder and Moens 2010). Linguistic databases like WordNet are used to produce simple synonyms of a complex word. (Lesk 1986; Sinha 2012; Leroy et al. 2013). Parallel corpora like Wikipedia-Simple Wikipedia corpus were also used to extract complex-to-simple word correspondences (Biran, Brody, and Elhadad 2011; Yatskar et al. 2010; Horn, Manduca, and Kauchak 2014). However, linguistic resources are scarce or expensive to produce, such asWordNet and Simple Wikipedia, and it is impossible to give all possible simplification rules from them.
어휘 단순화(LS)는 어린이(De Belder 및 Moens 2010), 비원어민(Paetzold 및 Specia 2016), 인지 장애가 있는 사람들(Feng 2009)을 포함한 다양한 그룹의 사람들을 도울 수 있는 보다 단순한 대안으로 복잡한 단어를 대체하는 것을 목표로 합니다.(Sagion 2017), 텍스트를 더 잘 이해하기 위해. 대중적인 LS 시스템은 복잡한 단어를 주의 깊게 손으로 만든 데이터베이스(예: WordNet)의 빈번한 동의어로 대체하거나 비교 가능한 말뭉치(Devlin and Tait 1998; De Belder and Moens 2010)에서 자동으로 유도하는 일련의 규칙을 여전히 주로 사용합니다. WordNet과 같은 언어 데이터베이스는 복잡한 단어의 간단한 동의어를 생성하는 데 사용됩니다. (Lesk 1986; Sinha 2012; Leroy et al. 2013). Wikipedia-Simple Wikipedia 말뭉치와 같은 병렬 말뭉치도 복합 단어에서 단순 단어로의 대응을 추출하는 데 사용되었습니다(Biran, Brody, Elhadad 2011; Yatskar et al. 2010; Horn, Manduca, and Kauchak 2014). 그러나 WordNet 및 Simple Wikipedia와 같은 언어 자원은 생산하기에 부족하거나 비용이 많이 들고 가능한 모든 단순화 규칙을 제공하는 것은 불가능합니다.



Figure 1: Comparison of simplification candidates of complex words.
Given one sentence ”John composed these verses.” and complex words ’composed’ and ’verses’, the top three simplification candidates for each complex word are generated by our method BERT-LS and the state-of-theart two baselines based word embeddings (Glavaˇs(Glavaˇs and ˇ Stajner 2015) and Paetzold-NE (Paetzold and Specia 2017a)).
그림 1: 복잡한 단어의 단순화 후보 비교.
한 문장이 주어졌을 때 "John composed these verses." 및 복합 단어 'composed' 및 'verses', 각 복합 단어에 대한 상위 3개 단순화 후보는 우리의 방법 BERT-LS와 최신 2개의 베이스라인 기반 단어 임베딩에 의해 생성됩니다(Glavaˇs(Glavaˇs 및 ˇ Stajner 2015) 및 Paetzold-NE(Paetzold 및 Specia 2017a)).



For avoiding the need for resources such as databases or parallel corpora, recent work utilizes word embedding models to extract simplification candidates for complex words (Glavaˇs and ˇ Stajner 2015; Paetzold and Specia 2016; 2017a). Given a complex word, they extracted from the word embedding model the simplification candidates whose vectors are closer in terms of cosine similarity with the complex word. This strategy achieves better results compared with rule-based LS systems. However, the above methods generated simplification candidates only considering the complex word regardless of the context of the complex word, which will inevitably produce a large number of spurious candidates that can confuse the systems employed in the subsequent steps.
데이터베이스 또는 병렬 말뭉치와 같은 리소스의 필요성을 피하기 위해 최근 작업에서는 단어 임베딩 모델을 사용하여 복잡한 단어에 대한 단순화 후보를 추출합니다(Glavaˇs and ˇ Stajner 2015; Paetzold and Specia 2016; 2017a). 복잡한 단어가 주어지면 단어 임베딩 모델에서 복잡한 단어와의 코사인 유사성 측면에서 벡터가 더 가까운 단순화 후보를 추출했습니다. 이 전략은 규칙 기반 LS 시스템에 비해 더 나은 결과를 달성합니다. 그러나, 위의 방법들은 복합어의 문맥에 관계없이 복합어만을 고려하여 단순화 후보를 생성하였고, 이는 필연적으로 많은 수의 가짜 후보를 생성하여 후속 단계에서 사용되는 시스템을 혼동시킬 수 있다.



Therefore, we present an intuitive and innovative idea completely different from existing LS systems in this paper. We exploit recent advances in the pre-trained transformer language model BERT (Devlin et al. 2018) to find suitable simplifications for complex words. The masked language model (MLM) used in BERT randomly masks some percentage of the input tokens, and predicts the masked word based on its context. If masking the complex word in a sentence, the idea in MLM is in accordance with generating the candidates of the complex word in LS. Therefore, we introduce a novel LS approach BERT-LS that uses MLM of BERT for simplification candidate generation. More specifically, we mask the complex word w of the original sentence S as a new sentence S`, and we concatenate the original sequence S and S` for feeding into the BERT to obtain the probability distribution of the vocabulary corresponding to the masked word. The advantage of our method is that it generates simplification candidates by considering the whole sentence, not just the complex word.
따라서 본 논문에서는 기존 LS 시스템과 완전히 다른 직관적이고 혁신적인 아이디어를 제시한다. 복잡한 단어에 적합한 단순화를 찾기 위해 pre-trained transformer language model BERT(Devlin et al. 2018)의 최근 발전을 활용합니다. BERT에서 사용되는 MLM(masked language model)은 입력 토큰의 일부를 무작위로 마스킹하고 컨텍스트를 기반으로 마스킹된 단어를 예측합니다.문장에서 복잡한 단어를 마스킹하는 경우 MLM의 아이디어는 LS에서 복잡한 단어의 후보를 생성하는 것과 같습니다. 따라서 단순화 후보 생성을 위해 BERT의 MLM을 사용하는 새로운 LS 접근 방식 BERT-LS를 소개합니다. 보다 구체적으로, 우리는 원래 문장 S의 복잡한 단어 w를 새로운 문장 S`로 마스킹하고, 마스크된 단어에 해당하는 어휘의 확률 분포를 얻기 위해 BERT에 공급하기 위해 원래 시퀀스 S와 S`를 연결합니다. 우리 방법의 장점은 복잡한 단어뿐만 아니라 전체 문장을 고려하여 단순화 후보를 생성한다는 것입니다.



Here, we give an example shown in Figure 1 to illustrate the advantage of our method BERT-LS. For complex words ’composed’ and ’verses’ in the sentence ”John composed these verses.”, the top three substitution candidates of the two complex words generated by the LS systems based on word embeddings (Glavaˇs and ˇ Stajner 2015; Paetzold and Specia 2017a) are only related with the complex words itself without without paying attention to the original sentence. The top three substitution candidates generated by BERT-LS are not only related with the complex words, but also can fit for the original sentence very well. Then, by considering the frequency or order of each candidate, we can easily choose ’wrote’ as the replacement of ’composed and ’poems’ as the replacement of ’verses’. In this case, the simplification sentence ’John wrote these poems.’ is more easily understood than the original sentence.
여기에서는 BERT-LS 방법의 이점을 설명하기 위해 그림 1에 표시된 예를 제공합니다. ”John composed these verses.”라는 문장에서 복잡한 단어 'composed'와 'verses'의 경우, 단어 임베딩을 기반으로 LS 시스템에 의해 생성된 두 개의 복잡한 단어의 상위 3개 대체 후보(Glavaˇs and ˇ Stajner 2015; Paetzold and Specia 2017a)는 원문에 신경을 쓰지 않고 복잡한 단어 자체와 관련이 있습니다. BERT-LS에 의해 생성된 상위 3개의 대체 후보는 복잡한 단어와 관련이 있을 뿐만 아니라 원래 문장에 매우 잘 맞습니다. 그러면 각 후보의 빈도나 순서를 고려하여 'composed'을 'wrote'으로, 'verses'를 'poems'로 대체하여 쉽게 선택할 수 있습니다. 이 경우 원문보다 'John wrote these poems.'라는 단순화 문장이 더 이해하기 쉽습니다.



The contributions of our paper are as follows:
(1) BERT-LS is a novel BERT-based method for LS, which can take full advantages of BERT to generate and rank substitution candidates. Compared with existing methods, BERT-LS is easier to hold cohesion and coherence of a sentence, since BERT-LS considers the whole sentence not the complex word itself during generating candidates.
(2) BERT-LS is a simple, effective and unsupervised LS method.
- 1)Simple: many steps used in existing LS systems have been eliminated from our method, e.g., morphological transformation and substitution selection.
- 2) Effective: it obtains new state-of-the-art results on three benchmarks.
- 3) Unsupervised: our method cannot rely on any parallel corpus or linguistic databases.
(3) To our best knowledge, this is the first attempt to apply Pre-Trained Transformer Language Models on lexical simplification tasks. The code to reproduce our results is available at https://github.com/anonymous.
우리 논문의 기고는 다음과 같습니다.
(1) BERT-LS는 LS를 위한 새로운 BERT 기반 방법으로, BERT를 최대한 활용하여 대체 후보를 생성하고 순위를 매길 수 있습니다. BERT-LS는 후보를 생성할 때 복잡한 단어 자체가 아니라 전체 문장을 고려하기 때문에 기존 방법에 비해 문장의 응집력과 일관성을 유지하기가 더 쉽습니다.
(2) BERT-LS는 간단하고 효과적이며 비지도 LS 방법입니다.
- 1) 단순: 기존 LS 시스템에서 사용된 많은 단계(예: 형태 변환 및 대체 선택)가 우리 방법에서 제거되었습니다.
- 2) 효과적: 3가지 벤치마크에서 새로운 최첨단 결과를 얻습니다.
- 3) 비지도학습: 우리의 방법은 병렬 코퍼스 또는 언어 데이터베이스에 의존할 수 없습니다.
(3) 우리가 아는 한, 이것은 Pre-Trained Transformer Language Models을 어휘 단순화 작업에 적용하려는 첫 번째 시도입니다. 결과를 재현하는 코드는 https://github.com/anonymous에서 사용할 수 있습니다.



## 2. Related Work
## 2. 관련 사항



Lexical simplification (LS) contains identifying complex words and finding the best candidate substitution for these complex words (Shardlow 2014; Paetzold and Specia 2017b). The best substitution needs to be more simplistic while preserving the sentence grammatically and keeping its meaning as much as possible, which is a very challenging task. The popular lexical simplification (LS) approaches are rule-based, which each rule contain a complex word and its simple synonyms (Lesk 1986; Pavlick and Callison-Burch 2016; Maddela and Xu 2018). In order to construct rules, rule-based systems usually identified synonyms from WordNet for a predefined set of complex words, and selected the ”simplest” from these synonyms based on the frequency of word (Devlin and Tait 1998; De Belder and Moens 2010) or length of word (Bautista et al. 2011). However, rule-based systems need a lot of human involvement to manually define these rules, and it is impossible to give all possible simplification rules.
어휘 단순화(LS)에는 복잡한 단어를 식별하고 이러한 복잡한 단어에 대한 최상의 대체 후보를 찾는 작업이 포함됩니다(Shardlow 2014; Paetzold and Specia 2017b). 가장 좋은 대체는 문장을 문법적으로 보존하고 의미를 최대한 유지하면서 더 단순해야 하는 매우 어려운 작업입니다. 인기 있는 어휘 단순화(LS) 접근 방식은 규칙 기반으로, 각 규칙에는 복잡한 단어와 간단한 동의어가 포함되어 있습니다(Lesk 1986; Pavlick and Callison-Burch 2016; Maddela and Xu 2018). 규칙을 구성하기 위해 규칙 기반 시스템은 일반적으로 사전 정의된 복잡한 단어 집합에 대해 WordNet에서 동의어를 식별하고 단어 빈도에 따라 이러한 동의어에서 "가장 단순한" 것을 선택했습니다(Devlin and Tait 1998; De Belder and Moens 2010). 또는 단어의 길이(Bautista et al. 2011). 그러나 규칙 기반 시스템은 이러한 규칙을 수동으로 정의하기 위해 많은 사람의 개입이 필요하며 가능한 모든 단순화 규칙을 제공하는 것은 불가능합니다.



As complex and simplified parallel corpora are available, especially, the ’ordinary’ English Wikipedia (EW) in combination with the ’simple’ English Wikipedia (SEW), the paradigm shift of LS systems is from knowledge-based to data-driven simplification (Biran, Brody, and Elhadad 2011; Yatskar et al. 2010; Horn, Manduca, and Kauchak 2014). Yatskar et al. (2010) identified lexical simplifications from the edit history of SEW. They utilized a probabilistic method to recognize simplification edits distinguishing from other types of content changes. Biran et al. (2011) considered every pair of the distinct word in the EW and SEW to be a possible simplification pair, and filtered part of them based on morphological variants andWordNet. Horn et al. (2014) also generated the candidate rules from the EW and SEW, and adopted a context-aware binary classifier to decide whether a candidate rule should be adopted or not in a certain context. The main limitation of the type of methods relies heavily on simplified corpora.
복잡하고 단순화된 병렬 말뭉치, 특히 '단순한' 영어 위키백과(SEW)와 결합된 '일반' 영어 위키백과(EW)가 가능해짐에 따라 LS 시스템의 패러다임은 지식 기반에서 데이터 기반 단순화( Biran, Brody, Elhadad 2011, Yatskar et al. 2010, Horn, Manduca, Kauchak 2014). Yatskar et al. (2010) SEW의 편집 기록에서 어휘 단순화를 식별했습니다. 그들은 다른 유형의 콘텐츠 변경과 구별되는 단순화 편집을 인식하기 위해 확률적 방법을 사용했습니다. Biranet al. (2011) EW와 SEW에 있는 고유한 단어의 모든 쌍을 가능한 단순화 쌍으로 간주하고 형태학적 변이와 WordNet을 기반으로 일부를 필터링했습니다. Horn et al. (2014) 또한 EW 및 SEW에서 후보 규칙을 생성하고 특정 컨텍스트에서 후보 규칙을 채택해야 하는지 여부를 결정하기 위해 컨텍스트 인식 바이너리 분류기를 채택했습니다. 방법 유형의 주요 제한은 단순화된 말뭉치에 크게 의존합니다.



In order to entirely avoid the requirement of lexical resources or parallel corpora, LS systems based on word embeddings were proposed (Glavaˇs and ˇ Stajner 2015). They extracted the top 10 words as candidate substitutions whose vectors are closer in terms of cosine similarity with the complex word. Instead of a traditional word embedding model, Paetzold and Specia (2016) adopted context-aware word embeddings trained on a large dataset where each word is annotated with the POS tag. Afterward, they further extracted candidates for complex words by combining word embeddings withWordNet and parallel corpora (Paetzold and Specia 2017a).
어휘 자원 또는 병렬 말뭉치의 요구 사항을 완전히 피하기 위해 단어 임베딩을 기반으로 하는 LS 시스템이 제안되었습니다(Glavaˇs 및 ˇ Stajner 2015). 그들은 복소수 단어와의 코사인 유사성 측면에서 벡터가 더 가까운 후보 치환으로 상위 10개 단어를 추출했습니다. Paetzold와 Specia(2016)는 전통적인 단어 임베딩 모델 대신 각 단어에 POS 태그가 주석으로 추가된 대규모 데이터 세트에서 훈련된 컨텍스트 인식 단어 임베딩을 채택했습니다. 그 후 WordNet 및 병렬 말뭉치로 단어 임베딩을 결합하여 복잡한 단어의 후보를 추가로 추출했습니다(Paetzold 및 Specia 2017a).



After examining existing LS methods ranging from rulesbased to embedding-based, the major challenge is that they generated simplification candidates for the complex word regardless of the context of the complex word, which will inevitably produce a large number of spurious candidates that can confuse the systems employed in the subsequent steps.
규칙 기반에서 임베딩 기반에 이르기까지 기존 LS 방법을 검토한 후 주요 문제는 복잡한 단어의 컨텍스트에 관계없이 복합 단어에 대한 단순화 후보를 생성한다는 것입니다. 후속 단계에서 사용됩니다.



In this paper, we will first present a BERT-based LS approach that requires only a sufficiently large corpus of regular text without any manual efforts. Pre-training language models (Devlin et al. 2018; Lee et al. 2019; Lample and Conneau 2019) have attracted wide attention and has shown to be effective for improving many downstream natural language processing tasks. Our method exploits recent advances in BERT to generate suitable simplifications for complex words. Our method generates the candidates of the complex word by considering the whole sentence that is easier to hold cohesion and coherence of a sentence. In this case, many steps used in existing steps have been eliminated from our method, e.g., morphological transformation and substitution selection. Due to its fundamental nature, our approach can be applied to many languages.
이 백서에서는 먼저 수동 작업 없이 충분히 큰 정규 텍스트 코퍼스만 필요한 BERT 기반 LS 접근 방식을 제시합니다. Pre-training language models(Devlin et al. 2018; Lee et al. 2019; Lample and Conneau 2019)는 많은 관심을 끌었으며 많은 다운스트림 자연어 처리 작업을 개선하는 데 효과적인 것으로 나타났습니다. 우리의 방법은 복잡한 단어에 대한 적절한 단순화를 생성하기 위해 BERT의 최근 발전을 활용합니다. 우리의 방법은 문장의 응집력과 일관성을 유지하기 쉬운 전체 문장을 고려하여 복잡한 단어의 후보를 생성합니다. 이 경우 기존 단계에서 사용된 많은 단계(예: 형태학적 변환 및 대체 선택)가 우리 방법에서 제거되었습니다. 근본적인 특성으로 인해 우리의 접근 방식은 많은 언어에 적용될 수 있습니다.


--*------------- 다음 시간에 -*-*--------------
## 3. Unsupervised Lexical Simplification