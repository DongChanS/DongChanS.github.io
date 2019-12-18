# WFST



## 1) Abstract

* HMM, Context-dependency model, pronunciation dictionary, grammars, word&phone lattice

  와 같은 **여러 모델을 표현할 수 있는 구조**

* 여러 알고리즘을 통해서 최적화를 할 수 있음

  * Composition
  * Determinization
  * Epsilon removal
  * Minimization
  * Weight pushing algorithm 

* Real-time decoder : 여러 HMM들과, cross-word triphone, lexicon, trigram grammar를 결합하여 만들 수 있음.

* Second-pass recognition



## 2) Introduction

### 2-1. Finite state transducer

* State transition은 input & output symbol로 라벨링이 되는 Finite state automaton

* Weight라는 개념을 추가하여 각 transition마다 weight를 매길 수 있음

  * 각 transition을 누적하여 path를 만들기 때문에 weight는 누적할 수 있는 어떠한 값 상관없음.

    ex) 확률

### 2-2. WFST in Speech recognition

* WFST 내에서 정의되는 효과적인 연산(Operation)을 이용하여 효율적인 그래프를 만들 수 있음

* 각 음성인식의 모듈들을 WFST로 취급함으로써 음성인식의 효율을 증대시킬 수 있음

  

## 3) Overview

### 3-1. Weighted Acceptors (Automaton)

1. 구성요소

   * Set of states
   * Set of initial states
   * Set of final states
   * Set of labels
   * Initial weight function
   * Final weight function
   * Set of transitions between states : `(source state, destination state, input label, weight)`

2. Run-time decoder의 구성요소

   * Lexicon : word pronunciation을 탐색
   * Grammar
   * Phonetic tree : 낭비되는 path 제거, 검색 효율성 증가
     * 결론적으로, 비슷한 Phone마다 같은 Context-dependent model을 배정하여, HMM-level transducer를 만드는 역할을 함.
   * Tied model topologies : 검색 효율성을 위해서 몇 가지 조건들을 제한함.
     * Triphone model
     * Trigram grammar
     * Alternative pronunciations might have to be enumerated in the lexicon (??)

   

### 3-2. Weighted Transducers

1. 음성인식 모듈을 표현가능

   * N-gram grammar
   * Pronunciation lexicon
   * Context-dependency specifications
   * HMM topology, segmentation (??)
   * Lattices : A compact representations of search space (최종적으로 음성인식 결과가 도출되는 그래프인 것 같습니다)
     * 참고 : https://www.cs.brandeis.edu/~cs136a/CS136a_Slides/zre_lecture_asr_wfst.pdf

2.  Weighted acceptor와의 차이점 : Input & Output label

   * Two levels of representations

3. 예시 - lexicon : 각 단어로부터 가능한 발음의 경우들을 나타내는 WFST

   * Input label : phoneme, Output label : word

   * 더 큰 단어로 합치기 때문에 epsilon이 등장하는 경우가 많음.
   * Acceptor로 Lexicon을 표현하면 Output label이 없기 때문에, 서로 다른 Lexicon을 결합할 때 Word identity를 잃게 되는데, Transducer로 표현하면 그렇지 않음.

4. Relation에 가까운 형태

   * 함수는 하나의 Input label에 대해서 고정된 Output label을 반환하지만,

     WFST는 하나의 Input label이어도 Path에 따라서 다른 Output label을 반환할 수 있기 때문에 Relation이라고 할 수 있습니다.

5. Operations

   * Rational power series에 근거함.
   * Unweighted acceptor에서 확장된 형태
   * 기본적인 연산의 종류
     * Union : 병렬로 같은 형태의 WFST를 결합
     * Concatenate : Sequential하게 같은 형태의 WFST를 결합
     * Kleene closure : 동일한 WFST의 반복 
     * projection
     * find best N path
     * Remove unreachable path
     * Sort acyclic topologies (??)

6. Lazy implementation

   * 일부 WFST의 연산은 부분적으로도 수행할 수 있기 때문에,

     On-demand한 형태로 필요할 부분에 대해서만 연산을 수행하는 방식이 효율적임.



### 3-3. Composition

For combining different levels of representaitons

ex) Pronunciation lexicon + Word-level grammar

1. Composition 연산
2. 예시
3. Local computation



### 3-4. Determinization

1. Deterministic automaton

   * 해당 State에서 시작하는 Input label이 동일한 path는 유일하다. 
   * Epsilon input label을 가지지 않음.

2. Advantages : 검색 효율성, 주어진 Input label에 대해서 검색할 수 있는 path가 유일하기 때문이다.

   * 특히 Pronunciation lexicon의 redundancy가 심하기 때문에 이를 꼭 해결해야함.

3. Determinization

   * Non-deterministic automaton -> Deterministic equivalent automaton

   * Equivalent

     * 각 주어진 Input string에 대해서 발생되는 **Path들의 총 weight는 동일**함.

       (물론, 각 path들의 weight의 분배는 달라도 상관없음) 

   * Eliminate redundant paths

     핵심적으로 가장 중요한 것은 Redundant path를 없애는 것이다.

     * Viterbi approximation : 같은 Input label에 대하여 **가장 가능성 있는 Path를 보존**하는 방법 중 하나

4. Tree-shaped automaton

   * Union of single pronunciation lexicon들에 대해서 Determinization을 결합하면 Tree-shape가 되기 때문에 매우 효율적으로 검색이 가능하다.
   * 일반적으로는 Determinization의 결과는 compact graph의 형태를 띈다.

5. Automaton에 대한 Deterministic

   * For each state `s`
     * For each input label `a` with leaving a state `s`
       * new transition `t` 를 생성함
         1. Weight = `w + l`
            * `w` : `s` 's of leftover weight (Weight - minimum weight)
            * `l` : weight of an `a`-transition leaving a state `s` in S 
         2. Input label = `a`
         3. Destination state = 집함
               *  집합의 원소 : `(q', w')`
            * `q'` : 기존 WFST의 destination state
            * `w'` : `q'`에 대한 leftover weight

6. Determinization 또한 locally하게 작동할 수 있다.

![1](WFST/WFST-1.png)

### 2.5 Minimization

deterministic weight automaton은 항상 적용이 가능하다.

1. 일반적인 적용 순서
     * pushes weight among transitions
     * classical minimum algorithm
2. weight pushing algorithm이 먼저 이용되며, 이는 minimization 뿐만 아니라 검색 효율성을 높이는 장점이 있음
3. Pushing algorithm
     * Potential function 
4. 






