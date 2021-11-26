# Recursive Context-Aware Lexical Simplification
# 재귀적 컨텍스트 인식 어휘 단순화


## Abstract

This paper presents a novel architecture for recursive context-aware lexical simplification, REC-LS, that is capable of (1) making use of the wider context when detecting the words in need of simplification and suggesting alternatives, and (2) taking previous simplification steps into account. We show that our system outputs lexical simplifications that are grammatically correct and semantically appropriate, and outperforms the current state-of-theart systems in lexical simplification.
이 논문은 (1) 단순화가 필요한 단어를 감지하고 대안을 제안할 때 더 넓은 컨텍스트를 활용하고, (2) 이전 단순화를 취할 수 있는 재귀적 컨텍스트 인식 어휘 단순화를 위한 새로운 아키텍처를 제시합니다. 단계를 고려합니다. 우리는 우리 시스템이 문법적으로 정확하고 의미적으로 적절한 어휘 단순화를 출력하고 어휘 단순화에서 현재의 최신 시스템을 능가한다는 것을 보여줍니다.



## 1. Introduction

Text simplification (TS) is aimed at reducing the reading and grammatical complexity of text while retaining the meaning and grammaticality (Chandrasekar and Bangalore, 1997). This is usually achieved by a series of transformations at the lexical and syntactic level. A number of systems in the recent years have approached this task in an integral manner (Zhu et al., 2010; Kauchak, 2013; Zhang and Lapata, 2017). Such comprehensive systems can perform a number of simplification operations at once, but the results are sometimes ungrammatical and meaning can be changed, arguably making the original text less clear and more complex (Siddharthan, 2014).
텍스트 단순화(TS)는 의미와 문법을 유지하면서 텍스트의 읽기 및 문법적 복잡성을 줄이는 것을 목표로 합니다(Chandrasekar and Bangalore, 1997). 이것은 일반적으로 어휘 및 구문 수준에서 일련의 변환에 의해 달성됩니다. 최근 몇 년 동안 많은 시스템이 통합 방식으로 이 작업에 접근했습니다(Zhu et al., 2010; Kauchak, 2013; Zhang and Lapata, 2017). 이러한 포괄적인 시스템은 여러 단순화 작업을 한 번에 수행할 수 있지만 결과는 때때로 비문법적이며 의미가 변경될 수 있으므로 틀림없이 원본 텍스트가 덜 명확하고 더 복잡해질 수 있습니다(Siddharthan, 2014).