

# Overview



## 1. typical system

### 1-1. Statistical ASR

WAVEFORM -> DISCRETE REP -> ACOUSTIC MODEL -> ...

1. SPEECH SIGNAL ANALYSIS

   * discretize samples : 각 샘플마다 10~25ms정도의 시간간격있음
     * frame
   * extract acoustic feature
   * assumption : stationary (within frame) - 그러지않으면 특성추출이 불가능?!
   * motivate : 귀가 어떻게 작동하는가? ex, filterbank

2. Acoustic model

   * phoneme : word 구분하기 위한 기본단위

     * 알파벳과 비슷한 범주임
     * language dependent characterization

   * 단어 -> sequence of phones : expert의 역할(CMU dict)

   * word를 기본단위로 안 쓰는 이유

     : unseen word때문에, 만약 phoneme을 기본단위로 쓰면 안쓰는 단어여도 phoneme 단위로 분리가능

     (물론 충분한 리소스가 있다면 word단위를 쓰는것도 불가능하지는 않음)

   * HMM : acoustic feature -> phonemes

     * 다음에 어떤 소리가 나올 지 모르기 때문에, 모든 next phoneme에 대한 경우의수를 고려해야함.

     * state -> probability : generates certain probability 

       ex) GMM, DNN

   * DNN-HMM

     * input : fixed window of frames 
     * output : Posterior probability of phones (given speech frame)

3. Pronunciation dictionary

   * 학습되지 않음 (전문가영역이므로)

   * phonetically transcribed : 전문가가 번역

   * 하지만, 실제로는 다양한 발음이 나올 수 있음

     * 책을 읽듯이 읽으면 정교한 표준발음처럼 나올 가능성이 높지만, 상호간의 대화에서는 속도가 매우 빠른편이다.

       => 다양한 발음이 나올 수 있음

     * 보통 4~5개의 발음종류가 있음

       ![12](/home/dongchans/Desktop/12.png)

   * 그러한 Speech variation을 모델링하는 방법?

     : articulatory feature-based pronunciation models

     * vocal track variables