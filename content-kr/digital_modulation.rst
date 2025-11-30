.. _modulation-chapter:

###################
Digital Modulation
###################

이 장에서는 디지털 변조(digital modulation)와 무선 심볼(wireless symbols)을 이용해 **실제로 데이터를 송신하는 방법**을 다루겠습니다! ASK, PSK, QAM, FSK 같은 변조 방식을 사용해 1과 0 같은 "정보"를 전달하는 신호를 설계할 것입니다. 또한 IQ 플롯과 별자리도(constellation)에 대해서도 살펴보고, 마지막에는 Python 예제를 통해 마무리하겠습니다.

변조의 주요 목표는 가능한 한 적은 스펙트럼(주파수 대역폭)을 사용하면서 최대한 많은 데이터를 전송하는 것입니다. 기술적으로 말하면, 비트/초/Hz 단위의 "스펙트럼 효율(spectral efficiency)"을 극대화하는 것입니다. 1과 0을 더 빠르게 송신하면 신호의 대역폭이 증가하며(푸리에 성질을 떠올리세요), 그만큼 더 넓은 스펙트럼을 사용하게 됩니다. 우리는 단순히 빠르게 송신하는 것 외에도 다양한 기법을 살펴볼 것입니다. 변조 방식을 결정할 때는 많은 절충점(trade-off)이 존재하지만, 동시에 창의적으로 접근할 수 있는 여지도 많습니다.


*******************
Symbols
*******************
새로운 용어 등장! 우리가 송신하는 신호는 "심볼(symbol)"로 이루어져 있습니다. 각 심볼은 일정한 수의 비트를 담고 있으며, 이 심볼들을 연속적으로—수천 개, 수백만 개까지—송신하게 됩니다.

간단한 예로, 전선을 통해 1과 0을 각각 높은 전압과 낮은 전압으로 송신한다고 합시다. 이때 심볼은 1 또는 0 하나를 의미합니다:

.. image:: ../_images/symbols.png
   :scale: 60 %
   :align: center
   :alt: 정보 비트를 담은 디지털 심볼 개념을 보여주는 펄스 열

위의 예에서 각 심볼은 1비트를 나타냅니다. 그렇다면 심볼당 1비트 이상을 표현하려면 어떻게 할까요? Ethernet 케이블을 통해 전송되는 신호를 살펴보겠습니다. IEEE 802.3 1000BASE-T라는 표준에 정의된 방식입니다. 일반적인 Ethernet 동작 모드는 4레벨 진폭 변조(심볼당 2비트)를 사용하며, 심볼 길이는 8 ns입니다.

.. image:: ../_images/ethernet.svg
   :align: center
   :target: ../_images/ethernet.svg
   :alt: IEEE 802.3 1000BASE-T Ethernet 전압 신호 플롯 – 4레벨 진폭편이키잉(ASK)

잠시 시간을 내어 아래 질문에 답해보세요:

1. 위 예에서 초당 몇 비트가 전송될까요?
2. 1 Gbps를 전송하려면 몇 쌍의 데이터 전선이 필요할까요?
3. 변조 방식이 16개 레벨을 가진다면, 심볼당 몇 비트를 전송할 수 있을까요?
4. 16개 레벨과 8 ns 심볼을 사용한다면 초당 몇 비트를 전송할 수 있을까요?

.. raw:: html

   <details>
   <summary>정답</summary>

1. 250 Mbps – (1/8e-9)*2
2. 네 쌍 (Ethernet 케이블이 실제로 그렇습니다)
3. 심볼당 4비트 – log₂(16)
4. 0.5 Gbps – (1/8e-9)*4

.. raw:: html

   </details>


*******************
Wireless Symbols
*******************
질문: 왜 위 그림의 Ethernet 신호를 무선으로 직접 송신할 수 없을까요? 여러 이유가 있지만, 가장 큰 두 가지는 다음과 같습니다:

1. 낮은 주파수는 *매우 큰* 안테나가 필요하며, 위 신호에는 DC(0 Hz)까지 포함되어 있습니다. DC는 전송할 수 없습니다.
2. 사각파(square wave)는 비트율 대비 지나치게 넓은 스펙트럼을 차지합니다. :ref:`freq-domain-chapter` 장에서 배웠듯, 시간 영역에서의 급격한 변화는 매우 넓은 대역폭/스펙트럼을 필요로 합니다:

.. image:: ../_images/square-wave.svg
   :align: center
   :target: ../_images/square-wave.svg
   :alt: 시간 및 주파수 영역에서 사각파가 얼마나 큰 대역폭을 차지하는지 보여주는 그림

무선 신호의 경우, 우리는 먼저 반송파(carrier)라는 사인파로 시작합니다. 예를 들어 FM 라디오는 101.1 MHz나 100.3 MHz 같은 반송파를 사용합니다. 이 반송파를 어떤 방식으로든 변조(modulation)합니다. FM 라디오는 디지털이 아닌 아날로그 변조지만, 디지털 변조와 동일한 개념을 기반으로 합니다.

그렇다면 반송파를 어떻게 변조할 수 있을까요? 같은 질문을 다른 식으로 표현하면, 사인파가 가질 수 있는 서로 다른 속성은 무엇일까요?

1. 진폭(Amplitude)
2. 위상(Phase)
3. 주파수(Frequency)

우리는 이 세 가지 중 하나(또는 그 이상)를 바꿔 데이터를 반송파에 실어 보낼 수 있습니다.


****************************
Amplitude Shift Keying (ASK)
****************************

**진폭 편이 키잉(ASK, Amplitude Shift Keying)**은 디지털 변조 방식 중 가장 먼저 다룰 주제입니다. 세 가지 사인파 속성(진폭, 위상, 주파수) 중 진폭을 변조하는 방식이며, 가장 직관적으로 시각화할 수 있습니다. 즉, 반송파의 **진폭**을 직접 변조하는 것입니다. 아래 그림은 2레벨 ASK(2-ASK)의 예시입니다:

.. image:: ../_images/ASK.svg
   :align: center
   :target: ../_images/ASK.svg
   :alt: 시간 영역에서의 진폭 편이 키잉(ASK) 예시, 2-ASK

평균값이 0이 되는 것을 주목하세요. 가능하다면 항상 이런 형태를 선호합니다.

두 개 이상의 레벨을 사용할 수도 있는데, 이 경우 심볼당 더 많은 비트를 표현할 수 있습니다. 아래는 4-ASK의 예시로, 각 심볼은 2비트의 정보를 담습니다:

.. image:: ../_images/ask2.svg
   :align: center
   :target: ../_images/ask2.svg
   :alt: 시간 영역에서의 진폭 편이 키잉(ASK) 예시, 4-ASK

질문: 위 신호 조각에 나타난 심볼은 몇 개이며, 총 몇 비트를 표현할까요?

.. raw:: html

   <details>
   <summary>정답</summary>

20개의 심볼, 따라서 총 40비트의 정보

.. raw:: html

   </details>

그렇다면 이 신호를 실제로 디지털 코드로 어떻게 생성할까요? 간단합니다. 심볼당 N개의 샘플을 가진 벡터를 만든 뒤, 그 벡터를 사인파와 곱하면 됩니다. 사인파가 반송파 역할을 하며, 신호를 반송파에 변조하는 것이죠. 아래 예시는 심볼당 10개 샘플을 사용한 2-ASK를 보여줍니다:

.. image:: ../_images/ask3.svg
   :align: center
   :target: ../_images/ask3.svg
   :alt: 시간 영역에서 심볼당 10개 샘플(sps)을 가진 2-ASK 예시

위쪽 플롯은 빨간 점으로 표시된 이산 샘플, 즉 디지털 신호를 나타냅니다. 아래쪽 플롯은 이를 변조한 후 실제 공중으로 송신할 수 있는 신호 모양입니다. 실제 시스템에서는 반송파 주파수가 심볼 변화율보다 훨씬 더 높습니다. 이 예시에서는 각 심볼에 사인파가 세 번만 반복되지만, 실제 무선 시스템에서는 송신 주파수 대역에 따라 수천 번의 주기가 들어가기도 합니다.


************************
위상 편이 변조 (Phase Shift Keying, PSK)
************************

이번에는 진폭을 변조했던 방식과 비슷하게, 위상(phase)을 변조하는 방법을 살펴보겠습니다.
가장 단순한 형태는 Binary PSK(BPSK)로, 두 가지 위상 상태만을 사용합니다:

1. 위상 변화 없음 (No phase change)
2. 180도 위상 변화 (180-degree phase change)

BPSK 예시 (위상이 바뀌는 지점을 주목하세요):

.. image:: ../_images/bpsk.svg
   :align: center
   :target: ../_images/bpsk.svg
   :alt: Simple example of binary phase shift keying (BPSK) in the time domain, showing a modulated carrier

하지만 이런 그래프는 보는 재미가 없습니다:

.. image:: ../_images/bpsk2.svg
   :align: center
   :target: ../_images/bpsk2.svg
   :alt: Phase shift keying like BPSK in the time domain is difficult to read, so we tend to use a constellation plot or complex plane

대신, 일반적으로 위상을 복소수 평면(complex plane) 상에 표현합니다.

***********************
IQ 플롯 / 성상도 (Constellations)
***********************

IQ 플롯은 이전에 :ref:`sampling-chapter` 챕터의 복소수 부분에서 본 적이 있지만, 이번에는 이를 더 흥미로운 방식으로 사용해보겠습니다.  특정 심볼(symbol)에 대해, IQ 플롯에서 진폭(amplitude)과 위상(phase)을 함께 표현할 수 있습니다.  BPSK 예제에서 우리는 위상이 0도와 180도인 두 가지 경우가 있다고 했습니다.  이 두 점을 IQ 플롯에 표시해 보겠습니다.  우리는 크기(magnitude)를 1이라고 가정할 것입니다.  실제로는 어떤 크기를 쓰든 큰 차이는 없습니다.  크기가 크다는 것은 높은 전력의 신호를 의미하지만, 단순히 증폭기 이득(amplifier gain)을 높여도 되기 때문입니다.

.. image:: ../_images/bpsk_iq.png
   :scale: 80 %
   :align: center
   :alt: IQ plot or constellation plot of BPSK

위의 IQ 플롯은 우리가 송신할 심볼들의 집합(즉, 선택 가능한 심볼들)을 보여줍니다.  반송파(carrier)는 표시되지 않으므로, 이 플롯은 심볼이 기저대역(baseband)에 있다고 생각할 수 있습니다.  특정 변조 방식에서 가능한 심볼의 집합을 나타낼 때 이를 “성상도(constellation)”라고 부릅니다.  많은 변조 방식들이 이 성상도로 정의될 수 있습니다.

BPSK를 수신하고 디코딩하기 위해서는 지난 장에서 배웠던 것처럼 IQ 샘플링을 사용하고, IQ 플롯 상에서 수신된 점들이 어디로 나타나는지 확인하면 됩니다.  다만, 무선 채널을 통해 신호가 전파될 때 어떤 임의의 지연(random delay)이 발생하므로 위상이 무작위로 회전합니다.  이 무작위 위상 회전은 이후에 배우게 될 다양한 방법으로 보정할 수 있습니다.  아래는 잡음(noise)을 제외하고, 수신단에서 BPSK 신호가 나타날 수 있는 다양한 예시입니다:

.. image:: ../_images/bpsk3.png
   :scale: 60 %
   :align: center
   :alt: A random phase rotation of BPSK occurs as the wireless signal travels through the air

PSK로 돌아옵시다.  만약 네 가지 위상 레벨, 즉 0도, 90도, 180도, 270도를 사용하고 싶다면 어떻게 될까요?  이 경우는 IQ 플롯에서 아래와 같이 표현되며, 이를 직교 위상 편이 변조(Quadrature Phase Shift Keying, QPSK)라고 부르는 변조 방식이 됩니다:

.. image:: ../_images/qpsk.png
   :scale: 60 %
   :align: center
   :alt: Example of Quadrature Phase Shift Keying (QPSK) in the IQ plot or constellation plot

PSK에서는 항상 N개의 위상이 있으며, 이상적인 경우 이 위상들은 360도를 균일하게 나누어 배치됩니다.  모든 점들이 동일한 크기를 가진다는 점을 강조하기 위해 종종 단위원(unit circle)을 함께 표시합니다:

.. image:: ../_images/psk_set.png
   :scale: 60 %
   :align: center
   :alt: Phase shift keying uses equally spaced constellation points on the IQ plot

질문: 아래 그림과 같은 PSK 방식을 사용하는 데에는 어떤 문제가 있을까요?  이것도 유효한 PSK 변조 방식일까요?

.. image:: ../_images/weird_psk.png
   :scale: 60 %
   :align: center
   :alt: Example of non-uniformly spaced PSK constellation plot

.. raw:: html

   <details>
   <summary>Answer</summary>

이 PSK 방식은 잘못된 것이 아닙니다.  분명 사용할 수는 있습니다.  하지만 점들이 균일하게 배치되어 있지 않기 때문에, 가능한 만큼 효율적이지 않습니다.  이 효율 문제는 뒤에서 잡음이 심볼에 어떤 영향을 미치는지를 논할 때 더 명확해질 것입니다.  간단히 말하면, 심볼들 사이에 가능한 한 많은 여유 공간을 두어야 수신단에서 잡음 때문에 다른 심볼로 잘못 해석되는 것을 막을 수 있습니다.  우리는 0을 보냈는데 1로 수신되는 일을 원하지 않습니다.

.. raw:: html

   </details>

ASK로 잠시 돌아가 봅시다.  PSK와 마찬가지로 ASK도 IQ 플롯에서 표현할 수 있습니다.  아래는 양극성(bipolar) 구성의 2-ASK, 4-ASK, 8-ASK, 그리고 단극성(unipolar) 구성의 2-ASK 및 4-ASK의 IQ 플롯입니다.

.. image:: ../_images/ask_set.png
   :scale: 50 %
   :align: center
   :alt: Bipolar and unipolar amplitude shift keying (ASK) constellation or IQ plots

눈치챘을 수도 있지만, 양극성 2-ASK는 BPSK와 동일합니다.  180도 위상 변화는 사인파에 -1을 곱하는 것과 동일합니다.  우리가 이를 BPSK라고 부르는 이유는, ASK보다 PSK가 훨씬 더 널리 쓰이기 때문일 것입니다.


**************************************
직교 진폭 변조 (Quadrature Amplitude Modulation, QAM)
**************************************
만약 ASK와 PSK를 결합하면 어떻게 될까요?  이러한 변조 방식을 직교 진폭 변조(Quadrature Amplitude Modulation, QAM)라고 부릅니다.  QAM은 일반적으로 아래와 같은 형태를 가집니다:

.. image:: ../_images/64qam.png
   :scale: 90 %
   :align: center
   :alt: Example of Quadrature Amplitude Modulation (QAM) on the IQ or constellation plot

다음은 QAM의 다른 예시들입니다:

.. image:: ../_images/qam.png
   :scale: 50 %
   :align: center
   :alt: Example of 16QAM, 32QAM, 64QAM, and 256QAM on the IQ or constellation plot

QAM 변조 방식에서는 위상 *및* 진폭 모두를 변조하기 때문에, 이론적으로는 IQ 플롯 상에서 점들을 원하는 위치에 배치할 수 있습니다.  특정 QAM 방식의 “파라미터(parameters)”는 그 변조 방식의 성상도(constellation)를 통해 가장 잘 정의됩니다.  또는 아래 QPSK 예시처럼, 각 점의 I와 Q 값을 표로 나타낼 수도 있습니다:

.. image:: ../_images/qpsk_list.png
   :scale: 80 %
   :align: center
   :alt: Constellation or IQ plots can also be represented using a table of symbols

대부분의 변조 방식은, 다양한 ASK들과 BPSK를 제외하면, 시간 영역(time domain)에서 시각적으로 이해하기가 꽤 어렵습니다.  이를 보여주기 위해, 다음은 시간 영역에서 본 QAM의 예시입니다.  아래 그림에서 각 심볼의 위상을 구분할 수 있을까요? 꽤 어렵습니다.

.. image:: ../_images/qam_time_domain.png
   :scale: 50 %
   :align: center
   :alt: Looking at QAM in the time domain is difficult which is why we use constellation or IQ plots

시간 영역에서 변조 방식을 구별하기 어렵기 때문에, 우리는 시간 영역 파형보다는 IQ 플롯을 사용하는 것을 선호합니다.  다만, 패킷 구조나 심볼 순서가 중요할 때는 시간 영역 신호를 함께 표시하기도 합니다.

****************************
주파수 편이 변조 (Frequency Shift Keying, FSK)
****************************

마지막으로 살펴볼 변조 방식은 주파수 편이 변조(Frequency Shift Keying, FSK)입니다.  FSK는 비교적 이해하기 쉽습니다.  각각의 주파수가 하나의 심볼을 나타내도록, N개의 주파수 사이를 전환하기만 하면 됩니다.  다만 실제로는 반송파(carrier)를 변조하므로, 반송파 주파수에서 +/- N개의 주파수로 이동하는 방식입니다. 예를 들어, 반송파가 1.2 GHz일 때 다음 네 개의 주파수 사이를 전환할 수 있습니다:

1. 1.2001 GHz
2. 1.2003 GHz
3. 1.1999 GHz
4. 1.1997 GHz

위 예시는 4-FSK이며, 따라서 한 심볼(symbol)당 2비트에 해당합니다.  주파수 간 간격은 200 kHz이고, 전체 신호 대역폭은 약 600 kHz 이상을 차지합니다.  이 4-FSK 신호를 기저대역(baseband)에서 여러 심볼에 대해 FFT로 보면 다음과 같은 모습일 것입니다:

.. image:: ../_images/fsk.svg
   :align: center
   :target: ../_images/fsk.svg
   :alt: Example of Frequency Shift Keying (FSK), specifically 4FSK

FSK를 사용할 때 반드시 던져야 할 중요한 질문이 있습니다: 주파수 간 간격은 얼마나 되어야 하는가?  이 간격을 보통 :math:`\Delta f` (Hz 단위)로 나타냅니다.  수신기가 각 심볼이 어떤 주파수였는지를 정확히 구분할 수 있도록 주파수 영역에서 겹치지 않아야 하므로, :math:`\Delta f`는 충분히 커야 합니다.

각 반송파의 주파수 폭은 심볼 속도(symbol rate)와 사용된 펄스 성형 필터(pulse shaping filter)의 함수입니다.  초당 더 많은 심볼을 전송할수록 심볼 길이는 짧아지고, 이는 더 넓은 대역폭을 의미합니다.  즉, 심볼 속도가 증가할수록 각 반송파는 더 넓게 퍼지고, 반송파들 간의 겹침을 피하기 위해 :math:`\Delta f`도 더 크게 설정해야 합니다.

IQ 플롯은 서로 다른 주파수를 보여줄 수 없습니다.  IQ 플롯은 크기(magnitude)와 위상(phase)을 표현하는 도구이기 때문입니다.  시간 영역(time domain)에서 FSK를 표시할 수는 있지만, 2개 이상의 주파수를 사용하면 어떤 심볼인지 구분하기가 어려워집니다:

.. image:: ../_images/fsk2.svg
   :align: center
   :target: ../_images/fsk2.svg
   :alt: Frequency Shift Keying (FSK) or 2FSK in the time domain

참고로 FM 라디오는 FSK의 아날로그 버전이라고 볼 수 있는 주파수 변조(Frequency Modulation, FM)를 사용합니다.  FSK가 이산적인 주파수들 간을 이동하는 반면, FM 라디오는 연속적인 오디오 신호를 사용해 반송파의 주파수를 연속적으로 변조합니다.  아래는 AM과 FM 변조 예시입니다. 최상단 신호는 반송파에 실릴 오디오 신호입니다.

.. image:: ../_images/am_fm_animation.gif
   :align: center
   :scale: 75 %
   :target: ../_images/am_fm_animation.gif
   :alt: Animation of a carrier, amplitude modulation (AM), and frequency modulation (FM) in the time domain

이 교재에서는 주로 디지털 변조 방식에 대해 다룰 예정입니다.


*******************
차분 부호화 (Differential Coding)
*******************

PSK 또는 QAM 기반의 많은 무선(그리고 유선) 통신 프로토콜에서는, 비트가 변조되기 직전(또는 복조 직후)에 **차분 부호화(differential coding)** 라는 단계를 거치게 됩니다.  이 기법의 유용성을 이해하기 위해 BPSK 수신 상황을 생각해 봅시다.  신호는 공기를 통해 전파되는 동안 송신기와 수신기 사이에 임의의 지연(random delay)을 겪게 되고, 앞서 언급했듯 성상도(constellation)에서 임의의 위상 회전이 발생합니다.  수신기가 이를 동기화하여 I(실수) 축과 정렬하더라도, BPSK 성상도는 대칭적이기 때문에 그 신호가 180도 뒤집혀 있는지 아닌지 구분할 수 없습니다.

한 가지 해결책으로는, 수신기가 미리 값을 알고 있는 심볼을 정보에 섞어서 전송하는 방법이 있습니다. 이를 **파일럿 심볼(pilot symbols)** 이라고 부르며, 이 심볼들을 통해 수신기는 어떤 클러스터가 1인지 0인지 구분할 수 있습니다(특히 BPSK의 경우).  그러나 파일럿 심볼은 무선 채널이 변화하는 속도에 맞춰 주기적으로 삽입해야 하므로, 결국 데이터 전송률이 감소하게 됩니다.  이러한 문제를 피하기 위해 우리는 차분 부호화를 사용할 수 있습니다.

가장 간단한 차분 부호화는 BPSK와 함께 사용할 때입니다(심볼당 1비트).  단순히 1 비트에 1을, 0 비트에 -1을 전송하는 대신, 차분 BPSK에서는 **이전 출력 비트**(이전 입력 비트가 아님!)와 같으면 0을, 다르면 1을 전송합니다.  시작을 위해 한 개의 임의 심볼을 추가해야 하지만(이는 반드시 송신해야 함), 이제 180도 위상 모호성 문제를 걱정할 필요가 없습니다.

이 부호화 방식은 다음 수식으로 표현됩니다. 여기서 :math:`x` 는 입력 비트, :math:`y` 는 BPSK로 변조될 출력 비트입니다:

.. math::
  y_i = y_{i-1} \oplus x_i

출력은 이전 출력에 기반하므로, 출력의 시작은 임의의 1 또는 0으로 두어야 하며, 디코딩 시에 어떤 값을 사용해도 상관없습니다(단, 이 시작 심볼은 반드시 전송해야 합니다!).

아래 그림은 차분 부호화 과정을 지연 블록(delay-by-1)을 통해 나타낸 것입니다:

.. image:: ../_images/differential_coding2.svg
   :align: center
   :target: ../_images/differential_coding2.svg
   :alt: Differential coding block diagram

예제로 10비트 [1, 1, 0, 0, 1, 1, 1, 1, 1, 0] 를 BPSK로 전송한다고 해 봅시다. 출력의 시작을 1로 둔다고 가정하면(0이어도 좋음), 다음과 같이 표기할 수 있습니다:

.. code-block::

 Input:     1 1 0 0 1 1 1 1 1 0
 Output:  1

다음 출력 비트는 입력 비트와 **이전 출력 비트**를 비교한 후 XOR 결과입니다. 1과 1이 같으므로 다음 출력은 0입니다:

.. code-block::

 Input:     1 1 0 0 1 1 1 1 1 0
 Output:  1 0

이를 전체 입력에 대해 반복하면:

.. code-block::

 Input:     1 1 0 0 1 1 1 1 1 0
 Output:  1 0 1 1 1 0 1 0 1 0 0

즉, 차분 부호화 후 전송될 비트는 [1, 0, 1, 1, 1, 0, 1, 0, 1, 0, 0] 입니다.
1과 0은 여전히 앞서 논의한 양/음 BPSK 심볼에 매핑됩니다.

수신기에서는 다음과 같은 간단한 방식으로 디코딩합니다:

.. math::
  x_i = y_i \oplus y_{i-1}

예를 들어 수신된 BPSK 심볼이 [1, 0, 1, 1, 1, 0, 1, 0, 1, 0, 0] 라면, 처음 두 비트를 비교합니다. 서로 다르므로 첫 입력 비트는 1입니다. 이를 반복하면 원래의 입력 [1, 1, 0, 0, 1, 1, 1, 1, 1, 0] 을 얻게 됩니다.
여기서 중요한 점은, 시작 심볼을 1로 두든 0으로 두든 최종 결과는 동일하다는 것입니다.

다음 도식은 인코딩과 디코딩 과정을 요약합니다:

.. image:: ../_images/differential_coding.svg
   :align: center
   :target: ../_images/differential_coding.svg
   :alt: Demonstration of differential coding using sequence of encoded and decoded bits

차분 부호화의 큰 단점은, **1비트 오류가 발생하면 2비트 오류로 이어질 수 있다는 점**입니다.  BPSK에서 차분 부호화를 사용하지 않는 대안은 앞서 설명한 파일럿 심볼을 사용하여 주기적으로 채널의 위상 뒤집힘을 보정하는 것입니다.  그러나 수신기 또는 송신기가 움직이는 상황에서는 무선 채널이 매우 빠르게 변할 수 있으므로(수십~수백 심볼 수준), 파일럿을 자주 넣어야 합니다.  따라서 수신기 복잡도를 줄이는 것이 중요한 프로토콜(예: 본문의 :ref:`rds-chapter` 에서 다루는 RDS)은 차분 부호화를 선택할 수 있습니다.

위의 차분 부호화 예제는 BPSK에 국한된 것이었습니다. 차분 부호화는 심볼 단위로 적용되므로, QPSK에는 2비트씩 묶어서 적용하며, 더 높은 차수의 QAM에서도 동일한 방식으로 확장됩니다.  차분 QPSK는 흔히 DQPSK라고 부릅니다.


*******************
Python 예제
*******************

간단한 Python 예제로, 기저대역(baseband)에서 QPSK를 생성하고 성상도를 그려보겠습니다.

복소수 심볼을 직접 생성할 수도 있지만, 우선 QPSK가 단위원(unit circle) 상에서 90도 간격으로 네 개의 심볼을 가진다는 점을 이용하겠습니다. 여기서는 45, 135, 225, 315도를 사용합니다. 먼저 0~3 사이의 난수를 만들고, 이를 각도로 변환한 뒤 다시 라디안으로 바꿉니다.

.. code-block:: python

 import numpy as np
 import matplotlib.pyplot as plt

 num_symbols = 1000

 x_int = np.random.randint(0, 4, num_symbols) # 0 to 3
 x_degrees = x_int*360/4.0 + 45 # 45, 135, 225, 315 degrees
 x_radians = x_degrees*np.pi/180.0 # sin() and cos() takes in radians
 x_symbols = np.cos(x_radians) + 1j*np.sin(x_radians) # this produces our QPSK complex symbols
 plt.plot(np.real(x_symbols), np.imag(x_symbols), '.')
 plt.grid(True)
 plt.show()

.. image:: ../_images/qpsk_python.svg
   :align: center
   :target: ../_images/qpsk_python.svg
   :alt: QPSK generated or simulated in Python

생성된 심볼들이 정확히 겹쳐 보이는 것을 확인할 수 있습니다. 잡음이 없기 때문에 모든 심볼이 동일한 값을 갖습니다. 이번에는 잡음을 추가해 보겠습니다:

.. code-block:: python

 n = (np.random.randn(num_symbols) + 1j*np.random.randn(num_symbols))/np.sqrt(2) # AWGN with unity power
 noise_power = 0.01
 r = x_symbols + n * np.sqrt(noise_power)
 plt.plot(np.real(r), np.imag(r), '.')
 plt.grid(True)
 plt.show()

.. image:: ../_images/qpsk_python2.svg
   :align: center
   :target: ../_images/qpsk_python2.svg
   :alt: QPSK with AWGN noise generated or simulated in Python

가산 백색 가우시안 잡음(AWGN)이 성상도에서 각 포인트 주변을 균일하게 퍼뜨리는 모습을 볼 수 있습니다. 잡음이 너무 크면 심볼이 사분면을 넘어가 수신기에서 잘못된 심볼로 해석됩니다. :code:`noise_power` 값을 점차 증가시키며 관찰해 보세요.

위상 잡음(phase noise)을 시뮬레이션하려면, 국부 발진기(LO)의 위상 지터를 가정하여 아래처럼 :code:`r`을 대체할 수 있습니다:

.. code-block:: python

 phase_noise = np.random.randn(len(x_symbols)) * 0.1 # adjust multiplier for "strength" of phase noise
 r = x_symbols * np.exp(1j*phase_noise)

.. image:: ../_images/phase_jitter.svg
   :align: center
   :target: ../_images/phase_jitter.svg
   :alt: QPSK with phase jitter generated or simulated in Python

AWGN과 위상 잡음을 함께 적용할 수도 있습니다:

.. image:: ../_images/phase_jitter_awgn.svg
   :align: center
   :target: ../_images/phase_jitter_awgn.svg
   :alt: QPSK with AWGN noise and phase jitter generated or simulated in Python

여기서 마무리하겠습니다. 만약 QPSK 신호를 시간 영역에서 보고자 한다면, 심볼당 여러 개의 샘플을 생성해야 합니다(이 예제에서는 심볼당 1개의 샘플만 생성했습니다). 왜 심볼당 여러 샘플을 생성해야 하는지는 펄스 성형(pulse shaping)을 다룰 때 배우게 됩니다. :ref:`pulse-shaping-chapter` 챕터의 Python 실습에서 여기서 이어서 진행할 예정입니다.


*******************
Further Reading
*******************

#. https://en.wikipedia.org/wiki/Differential_coding
