# Attention Is All You Need



## Abstract


The dominant sequence transduction models are based on complex recurrent or convolutional neural networks that include an encoder and a decoder. The best performing models also connect the encoder and decoder through an attention mechanism. We propose a new simple network architecture, the Transformer, based solely on attention mechanisms, dispensing with recurrence and convolutions entirely. Experiments on two machine translation tasks show these models to be superior in quality while being more parallelizable and requiring significantly less time to train. Our model achieves 28.4 BLEU on the WMT 2014 Englishto-German translation task, improving over the existing best results, including ensembles, by over 2 BLEU. On the WMT 2014 English-to-French translation task, our model establishes a new single-model state-of-the-art BLEU score of 41.8 after training for 3.5 days on eight GPUs, a small fraction of the training costs of the best models from the literature. We show that the Transformer generalizes well to other tasks by applying it successfully to English constituency parsing both with large and limited training data.
지배적 시퀀스 변환 모델은 인코더와 디코더를 포함하는 복잡한 순환 신경망 또는 합성곱 신경망을 기반으로 합니다. 최고 성능의 모델은 또한 주의 메커니즘을 통해 인코더와 디코더를 연결합니다. 우리는 주의 메커니즘에만 기반을 둔 새로운 단순 네트워크 아키텍처인 Transformer를 제안하고, 반복 및 회선을 완전히 제거합니다. 두 가지 기계 번역 작업에 대한 실험은 이러한 모델이 더 병렬화 가능하고 훈련하는 데 훨씬 적은 시간이 소요되는 동시에 품질이 우수함을 보여줍니다. 우리 모델은 WMT 2014 영어-독일어 번역 작업에서 28.4 BLEU를 달성하여 앙상블을 포함한 기존 최고의 결과보다 2 BLEU 이상 향상되었습니다. WMT 2014 영어-프랑스어 번역 작업에서 우리 모델은 8개의 GPU에서 3.5일 동안 교육한 후 41.8의 새로운 단일 모델 최신 BLEU 점수를 설정합니다. 이는 최고 교육 비용의 작은 부분입니다. 문학에서 모델. 우리는 Transformer가 대규모 및 제한된 교육 데이터 모두를 사용하여 영어 구성 요소 구문 분석에 성공적으로 적용함으로써 다른 작업에 잘 일반화됨을 보여줍니다.



## 1. Introduction



Recurrent neural networks, long short-term memory [13] and gated recurrent [7] neural networks in particular, have been firmly established as state of the art approaches in sequence modeling and transduction problems such as language modeling and machine translation [35, 2, 5]. Numerous efforts have since continued to push the boundaries of recurrent language models and encoder-decoder architectures [38, 24, 15].
순환 신경망, 특히 LSTM[13] 및 GRU[7]는 언어 모델링 및 기계 번역과 같은 시퀀스 모델링 및 변환 문제에서 최첨단 접근 방식으로 확고히 확립되었습니다[35, 2 , 5]. 그 이후로 반복적인 언어 모델과 인코더-디코더 아키텍처의 경계를 넓히기 위한 수많은 노력이 계속되었습니다[38, 24, 15].


Equal contribution. Listing order is random. Jakob proposed replacing RNNs with self-attention and started the effort to evaluate this idea. Ashish, with Illia, designed and implemented the first Transformer models and has been crucially involved in every aspect of this work. Noam proposed scaled dot-product attention, multi-head attention and the parameter-free position representation and became the other person involved in nearly every detail. Niki designed, implemented, tuned and evaluated countless model variants in our original codebase and tensor2tensor. Llion also experimented with novel model variants, was responsible for our initial codebase, and efficient inference and visualizations. Lukasz and Aidan spent countless long days designing various parts of and implementing tensor2tensor, replacing our earlier codebase, greatly improving results and massively accelerating our research.
동등한 기여. 나열 순서는 무작위입니다. Jakob은 RNN을 self-attention으로 대체할 것을 제안하고 이 아이디어를 평가하기 위한 노력을 시작했습니다. Ashish는 Illia와 함께 최초의 Transformer 모델을 설계 및 구현했으며 이 작업의 모든 측면에 결정적으로 관여했습니다. Noam은 scaled dot-product Attention, multi-head Attention 및 parameter-free position representation을 제안했고 거의 모든 세부 사항에 관련된 다른 사람이 되었습니다. Niki는 원래 코드베이스와 tensor2tensor에서 수많은 모델 변형을 설계, 구현, 조정 및 평가했습니다. Llion은 또한 새로운 모델 변형을 실험했고 초기 코드베이스와 효율적인 추론 및 시각화를 담당했습니다. Lukasz와 Aidan은 tensor2tensor의 다양한 부분을 설계하고 구현하고, 이전 코드베이스를 교체하고, 결과를 크게 개선하고, 연구를 가속화하는 데 셀 수 없이 긴 시간을 보냈습니다.



Recurrent models typically factor computation along the symbol positions of the input and output sequences. Aligning the positions to steps in computation time, they generate a sequence of hidden states ht, as a function of the previous hidden state ht−1 and the input for position t. This inherently sequential nature precludes parallelization within training examples, which becomes critical at longer sequence lengths, as memory constraints limit batching across examples. Recent work has achieved significant improvements in computational efficiency through factorization tricks [21] and conditional computation [32], while also improving model performance in case of the latter. The fundamental constraint of sequential computation, however, remains.
순환 모델은 일반적으로 입력 및 출력 시퀀스의 기호 위치를 따라 계산을 고려합니다. 계산 시간의 단계에 위치를 정렬하면 이전 숨겨진 상태 ht−1 및 위치 t에 대한 입력의 함수로 숨겨진 상태 ht의 시퀀스가 생성됩니다. 이 본질적으로 순차적인 특성은 메모리 제약이 예제 간의 일괄 처리를 제한하기 때문에 더 긴 시퀀스 길이에서 중요해지는 훈련 예제 내 병렬화를 배제합니다. 최근 연구는 인수분해 트릭[21]과 조건부 계산[32]을 통해 계산 효율성에서 상당한 개선을 달성했으며 후자의 경우 모델 성능도 개선했습니다. 그러나 순차 계산의 근본적인 제약은 남아 있습니다.



Attention mechanisms have become an integral part of compelling sequence modeling and transduction models in various tasks, allowing modeling of dependencies without regard to their distance in the input or output sequences [2, 19]. In all but a few cases [27], however, such attention mechanisms are used in conjunction with a recurrent network
주의 메커니즘은 다양한 작업에서 강력한 시퀀스 모델링 및 변환 모델의 필수적인 부분이 되어 입력 또는 출력 시퀀스에서 거리에 관계없이 종속성을 모델링할 수 있습니다[2, 19]. 그러나 몇몇 경우를 제외하고는 [27] 이러한 주의 메커니즘이 순환 네트워크와 함께 사용됩니다.



In this work we propose the Transformer, a model architecture eschewing recurrence and instead relying entirely on an attention mechanism to draw global dependencies between input and output. The Transformer allows for significantly more parallelization and can reach a new state of the art in translation quality after being trained for as little as twelve hours on eight P100 GPUs.
이 작업에서 우리는 반복을 피하고 대신 입력과 출력 사이의 전역 종속성을 끌어내기 위해 주의 메커니즘에 전적으로 의존하는 모델 아키텍처인 Transformer를 제안합니다. Transformer는 훨씬 더 많은 병렬화를 허용하고 8개의 P100 GPU에서 12시간 동안 교육을 받은 후 번역 품질의 새로운 상태에 도달할 수 있습니다.



## 2. Background



The goal of reducing sequential computation also forms the foundation of the Extended Neural GPU [16], ByteNet [18] and ConvS2S [9], all of which use convolutional neural networks as basic building block, computing hidden representations in parallel for all input and output positions. In these models, the number of operations required to relate signals from two arbitrary input or output positions grows in the distance between positions, linearly for ConvS2S and logarithmically for ByteNet. This makes it more difficult to learn dependencies between distant positions [12]. In the Transformer this is reduced to a constant number of operations, albeit at the cost of reduced effective resolution due to averaging attention-weighted positions, an effect we counteract with Multi-Head Attention as described in section 3.2.
순차 계산을 줄이는 목표는 또한 확장 신경망 GPU[16], ByteNet[18] 및 ConvS2S[9]의 기초를 형성하며, 모두 기본 빌딩 블록으로 컨볼루션 신경망을 사용하고 모든 입력에 대해 병렬로 숨겨진 표현을 계산합니다. 출력 위치. 이러한 모델에서 두 개의 임의 입력 또는 출력 위치의 신호를 연결하는 데 필요한 작업 수는 위치 사이의 거리에서 증가합니다. ConvS2S의 경우 선형으로, ByteNet의 경우 대수적으로 증가합니다. 이것은 원거리 위치 간의 종속성을 학습하는 것을 더 어렵게 만듭니다[12]. Transformer에서 이것은 평균적인 Attention-Weighted 위치로 인해 감소된 유효 해상도를 희생하더라도 일정한 수의 작업으로 감소합니다. 섹션 3.2에 설명된 대로 Multi-Head Attention으로 상쇄하는 효과입니다.



Self-attention, sometimes called intra-attention is an attention mechanism relating different positions of a single sequence in order to compute a representation of the sequence. Self-attention has been used successfully in a variety of tasks including reading comprehension, abstractive summarization, textual entailment and learning task-independent sentence representations [4, 27, 28, 22].
때때로 인트라 어텐션(intra-attention)이라고 하는 셀프 어텐션은 시퀀스의 표현을 계산하기 위해 단일 시퀀스의 서로 다른 위치를 연결하는 어텐션 메커니즘입니다. Self-attention은 독해, 추상적 요약, 텍스트 수반 및 학습과제 독립적인 문장 표현을 포함한 다양한 과제에서 성공적으로 사용되었습니다[4, 27, 28, 22].



End-to-end memory networks are based on a recurrent attention mechanism instead of sequencealigned recurrence and have been shown to perform well on simple-language question answering and language modeling tasks [34].
종단 간 메모리 네트워크는 순차 정렬 반복 대신 반복 주의 메커니즘을 기반으로 하며 간단한 언어 질문 응답 및 언어 모델링 작업에서 잘 수행되는 것으로 나타났습니다[34].



To the best of our knowledge, however, the Transformer is the first transduction model relying entirely on self-attention to compute representations of its input and output without using sequencealigned RNNs or convolution. In the following sections, we will describe the Transformer, motivate self-attention and discuss its advantages over models such as [17, 18] and [9].
그러나 우리가 아는 한, Transformer는 sequencealigned RNN이나 convolution을 사용하지 않고 입력 및 출력의 표현을 계산하기 위해 전적으로 self-attention에 의존하는 최초의 변환 모델입니다. 다음 섹션에서는 Transformer에 대해 설명하고 자기 주의를 환기시키며 [17, 18] 및 [9]와 같은 모델에 비해 이점에 대해 논의합니다.



## 3. Model Architecture



Most competitive neural sequence transduction models have an encoder-decoder structure [5, 2, 35]. Here, the encoder maps an input sequence of symbol representations (x1, ..., xn) to a sequence of continuous representations z = (z1, ..., zn). Given z, the decoder then generates an output sequence (y1, ..., ym) of symbols one element at a time. At each step the model is auto- egressive [10], consuming the previously generated symbols as additional input when generating the next.
대부분의 경쟁적인 신경 시퀀스 변환 모델은 인코더-디코더 구조를 가지고 있습니다[5, 2, 35]. 여기서 인코더는 기호 표현의 입력 시퀀스(x1, ..., xn)를 연속 표현 z = (z1, ..., zn)의 시퀀스에 매핑합니다. z가 주어지면 디코더는 한 번에 한 요소씩 심볼의 출력 시퀀스(y1, ..., ym)를 생성합니다. 각 단계에서 모델은 자동회귀(autoegressive) [10]이며, 다음을 생성할 때 이전에 생성된 기호를 추가 입력으로 사용합니다.



The Transformer follows this overall architecture using stacked self-attention and point-wise, fully connected layers for both the encoder and decoder, shown in the left and right halves of Figure 1, respectively
Transformer는 각각 그림 1의 왼쪽과 오른쪽 절반에 표시된 인코더와 디코더 모두에 대해 스택형 self-attention 및 point-wise, fully connected 레이어를 사용하여 이 전체 아키텍처를 따릅니다.



### 3-1. Encoder and Decoder Stacks
### 3-1. 인코더 및 디코더 스택



Encoder: The encoder is composed of a stack of N = 6 identical layers. Each layer has two sub-layers. The first is a multi-head self-attention mechanism, and the second is a simple, positionwise fully connected feed-forward network. We employ a residual connection [11] around each of the two sub-layers, followed by layer normalization [1]. That is, the output of each sub-layer is LayerNorm(x + Sublayer(x)), where Sublayer(x) is the function implemented by the sub-layer itself. To facilitate these residual connections, all sub-layers in the model, as well as the embedding layers, produce outputs of dimension dmodel = 512.
인코더: 인코더는 N = 6개의 동일한 레이어 스택으로 구성됩니다. 각 레이어에는 두 개의 하위 레이어가 있습니다. 첫 번째는 multi-head self-attention 메커니즘이고 두 번째는 단순하고 위치별로 완전히 연결된 순방향 신경망입니다. 우리는 두 하위 계층 각각 주위에 잔여 연결[11]을 사용하고 계층 정규화[1]를 수행합니다. 즉, 각 하위 계층의 출력은 LayerNorm(x + Sublayer(x))이며, 여기서 Sublayer(x)는 하위 계층 자체에서 구현하는 기능입니다. 이러한 잔여 연결을 용이하게 하기 위해 모델의 모든 하위 계층과 임베딩 계층은 차원 dmodel = 512의 출력을 생성합니다.



Decoder: The decoder is also composed of a stack of N = 6 identical layers. In addition to the two sub-layers in each encoder layer, the decoder inserts a third sub-layer, which performs multi-head attention over the output of the encoder stack. Similar to the encoder, we employ residual connections around each of the sub-layers, followed by layer normalization. We also modify the self-attention sub-layer in the decoder stack to prevent positions from attending to subsequent positions. This masking, combined with fact that the output embeddings are offset by one position, ensures that the predictions for position i can depend only on the known outputs at positions less than i.
디코더: 디코더도 N = 6개의 동일한 레이어 스택으로 구성됩니다. 각 인코더 계층의 두 하위 계층 외에도 디코더는 인코더 스택의 출력에 대해 multi-head attention를 수행하는 세 번째 하위 계층을 삽입합니다. 인코더와 유사하게 각 하위 계층 주위에 잔여 연결을 사용하고 계층 정규화를 수행합니다. 위치가 후속 위치에 주의를 기울이지 않도록 디코더 스택의 self-attention 하위 계층도 수정합니다. 출력 임베딩이 한 위치만큼 오프셋된다는 사실과 결합된 이 마스킹은 위치 i에 대한 예측이 i보다 작은 위치에서 알려진 출력에만 의존할 수 있도록 합니다.



## 3-2. Attention



An attention function can be described as mapping a query and a set of key-value pairs to an output, where the query, keys, values, and output are all vectors. The output is computed as a weighted sum of the values, where the weight assigned to each value is computed by a compatibility function of the query with the corresponding key
주의 기능은 쿼리 및 키-값 쌍 집합을 출력에 매핑하는 것으로 설명할 수 있습니다. 여기서 쿼리, 키, 값 및 출력은 모두 벡터입니다. 출력은 값의 가중치 합계로 계산되며, 여기서 각 값에 할당된 가중치는 해당 키와 쿼리의 호환성 함수에 의해 계산됩니다.



### 3-2-1. Scaled Dot-Product Attention



We call our particular attention "Scaled Dot-Product Attention" (Figure 2). The input consists of queries and keys of dimension dk, and values of dimension dv. We compute the dot products of the query with all keys, divide each by √dk, and apply a softmax function to obtain the weights on the values.
우리는 우리의 특별한 주의를 "Scaled Dot-Product Attention"(그림 2)이라고 부릅니다. 입력은 쿼리 및 차원 dk의  키와 차원 dv의 값으로 구성됩니다. 모든 키로 쿼리의 내적을 계산하고 각각을 √dk로 나누고 값에 대한 가중치를 얻기 위해 softmax 함수를 적용합니다.



In practice, we compute the attention function on a set of queries simultaneously, packed together into a matrix Q. The keys and values are also packed together into matrices K and V . We compute the matrix of outputs as:
실제로, 쿼리 세트에 대한 주의 함수를 동시에 계산하고 행렬 Q로 함께 묶습니다. 키와 값도 행렬 K 및 V로 함께 묶입니다. 출력 행렬을 다음과 같이 계산합니다.



The two most commonly used attention functions are additive attention [2], and dot-product (multiplicative) attention. Dot-product attention is identical to our algorithm, except for the scaling factor of 1/√dk. Additive attention computes the compatibility function using a feed-forward network with a single hidden layer. While the two are similar in theoretical complexity, dot-product attention is much faster and more space-efficient in practice, since it can be implemented using highly optimized matrix multiplication code.
가장 일반적으로 사용되는 두 가지 주의 기능은 가산 주의[2]와 내적(승법) 주의입니다. Dot-product Attention은 1/√dk의 스케일링 인자를 제외하고 우리 알고리즘과 동일합니다. 둘은 이론적 복잡성에서 유사하지만, 내적 주의는 고도로 최적화된 행렬 곱셈 코드를 사용하여 구현할 수 있기 때문에 실제로 훨씬 빠르고 공간 효율적입니다.



While for small values of dk the two mechanisms perform similarly, additive attention outperforms dot product attention without scaling for larger values of dk [3]. We suspect that for large values of dk, the dot products grow large in magnitude, pushing the softmax function into regions where it has extremely small gradients4. To counteract this effect, we scale the dot products by 1/√dk.
작은 값의 dk에 대해 두 메커니즘이 유사하게 작동하는 반면, 가산적 주의는 더 큰 값의 dk에 대한 스케일링 없이 내적 주의를 능가합니다[3]. dk의 큰 값에 대해 내적은 크기가 크게 증가하여 softmax 함수를 매우 작은 기울기가 있는 영역으로 밀어넣는다고 생각합니다. 이 효과를 상쇄하기 위해 내적을 1/√dk만큼 스케일링합니다.

4To illustrate why the dot products get large, assume that the components of q and k are independent random variables with mean 0 and variance 1. Then their dot product, q·k = ∑dk i=1 qiki, has mean 0 and variance dk.
내적이 커지는 이유를 설명하기 위해 q와 k의 성분이 평균이 0이고 분산이 1인 독립 확률 변수라고 가정합니다. 그러면 내적 q·k = ∑dk i=1 qiki는 평균 0과 분산 dk를 갖습니다.



### 3-2-2. Multi-Head Attention



Instead of performing a single attention function with dmodel-dimensional keys, values and queries, we found it beneficial to linearly project the queries, keys and values h times with different, learned linear projections to dk, dk and dv dimensions, respectively. On each of these projected versions of queries, keys and values we then perform the attention function in parallel, yielding dv-dimensional output values. These are concatenated and once again projected, resulting in the final values, as depicted in Figure 2.
dmodel 차원 키, 값 및 쿼리로 단일 주의 기능을 수행하는 대신 각각 dk, dk 및 dv 차원에 대해 학습된 서로 다른 선형 프로젝션으로 쿼리, 키 및 값을 h번 선형 프로젝션하는 것이 유익하다는 것을 발견했습니다. 쿼리, 키 및 값의 이러한 프로젝션 버전 각각에 대해 주의 기능을 병렬로 수행하여 dv 차원 출력 값을 생성합니다. 이것들은 연결되고 다시 한 번 투영되어 그림 2와 같이 최종 값이 됩니다.



Multi-head attention allows the model to jointly attend to information from different representation subspaces at different positions. With a single attention head, averaging inhibits this.
Multi-head attention을 통해 모델은 서로 다른 위치에 있는 서로 다른 표현 부분 공간의 정보에 공동으로 주의를 기울일 수 있습니다. a single attention head의 경우 평균화는 이를 억제합니다.



In this work we employ h = 8 parallel attention layers, or heads. For each of these we use dk = dv = dmodel/h = 64. Due to the reduced dimension of each head, the total computational cost is similar to that of single-head attention with full dimensionality.
이 작업에서 우리는 h = 8개의 parallel attention 레이어 또는 헤드를 사용합니다. 이들 각각에 대해 dk = dv = dmodel/h = 64를 사용합니다. 각 헤드의 축소된 차원으로 인해 총 계산 비용은 전체 차원을 갖는 single-head attention 비용과 유사합니다.



### 3-2-3 Applications of Attention in our Model
### 3-2-3 우리 모델에서의 Attention 적용



The Transformer uses multi-head attention in three different ways:
Transformer는 다음과 같은 세 가지 방식으로 multi-head attention을 사용합니다.



- In "encoder-decoder attention" layers, the queries come from the previous decoder layer, and the memory keys and values come from the output of the encoder. This allows every position in the decoder to attend over all positions in the input sequence. This mimics the  typical encoder-decoder attention mechanisms in sequence-to-sequence models such as [38, 2, 9].
- "encoder-decoder attention" 계층에서 쿼리는 이전 디코더 계층에서 가져오고 메모리 키와 값은 인코더의 출력에서 가져옵니다. 이를 통해 디코더의 모든 위치가 입력 시퀀스의 모든 위치를 처리할 수 있습니다. 이는 [38, 2, 9]와 같은 sequence-to-sequence 모델에서 일반적인 encoder-decoder attention 메커니즘을 모방합니다.



- The encoder contains self-attention layers. In a self-attention layer all of the keys, values and queries come from the same place, in this case, the output of the previous layer in the encoder. Each position in the encoder can attend to all positions in the previous layer of the encoder.
- 인코더에는 self-attention 레이어가 포함되어 있습니다. self-attention layer에서 모든 키, 값 및 쿼리는 같은 위치에서 옵니다. 이 경우 인코더에서 이전 레이어의 출력입니다. 인코더의 각 위치는 인코더의 이전 계층에 있는 모든 위치를 처리할 수 있습니다.



- Similarly, self-attention layers in the decoder allow each position in the decoder to attend to all positions in the decoder up to and including that position. We need to prevent leftward information flow in the decoder to preserve the auto-regressive property. We implement this inside of scaled dot-product attention by masking out (setting to -∞) all values in the input of the softmax which correspond to illegal connections. See Figure 2.
- 유사하게, 디코더의 self-attention layer는 디코더의 각 위치가 디코더의 모든 위치에 해당 위치를 포함하도록 합니다. 자동 회귀 속성을 유지하려면 디코더에서 왼쪽으로 정보 흐름을 방지해야 합니다. 잘못된 연결에 해당하는 softmax 입력의 모든 값을 마스크 아웃(-∞로 설정)하여 scaled dot-product attention 내부에서 이것을 구현합니다. 그림 2를 참조하십시오.



## 3-3 Position-wise Feed-Forward Networks
## 3-3 위치별 순방향 신경망



