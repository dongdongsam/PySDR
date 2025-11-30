.. _freq-domain-chapter:

#####################
Frequency Domain
#####################

이 챕터에서는 주파수 영역/푸리에 급수/푸리에 변환/Widowing/스펙트럼에 대해서 Python 예시 코드와 함께 다루도록 하겠습니다.

DSP와 무선통신에 대해 배울 때 멋진 점 중에 하나는, 바로 여러분들이 주파수 영역을 다루게 된다는 점 입니다. 대게는 주파수 영역이 다룰 일이 차량의 라디오 다이얼을 돌리는 일이 대부분이긴 하고, 이는 다시 말해서 아래의 오디오 이퀄라이저 화면을 보는게 전부라는 소리입니다.:

.. image:: ../_images/audio_equalizer.webp
   :align: center
   
이 챕터를 마치면 주파수 도메인이 무엇을 의미하는지, 그리고 주파수 도메인에서 시간 도메인으로 어떻게 변환하는지 (거기에 더해서 그럴때 뭔 일이 일어나는지)를 배울겁니다. 그리고 SDR과 DSP를 배우면서 적용되는 신기한 원리들에 대해서도 다룰 예정입니다. 그리고 이 책의 끝에서는, 제가 장담하건데 - 여러분들은 주파수 도메인의 전문가가 되어 있을 것입니다!

그래서 일단, 왜 우리는 신호를 주파수 도메인에서 보는 것을 좋아할까요? 그래서 아래에 두 신호를, 각각 주파수 영역과 시간 영역에 표시한 것을 보여드리겠습니다.

.. image:: ../_images/time_and_freq_domain_example_signals.png
   :scale: 40 %
   :align: center
   :alt: 시간 도메인에서의 신호는 그냥 노이즈 처럼 보이지만, 주파수 도메인에서 보면 신기한 특징이 보입니다.

보다시피, 시간 영역에서는 이상한 노이즈로 보이지만 주파수 영역에서는 도드라지는 특징이 나타납니다.  여러분이 신호를 받아 들이면, 시간 도메인의 모든 것은 대게 자연적인 형태입니다. 왜냐하면 여러분들은 신호를 **직접** 주파수 도메인으로 샘플링 할수 없거든요. 그렇지만 재미있는건, 바로 주파수 도메인입니다. 그러면 어떻게 해야 할까요?

***************
푸리에 급수
***************

주파수 도메인에 대해서 이해하기 위해서는, 먼저 모든 신호는 사인파의 합으로 표현 가능하다는 걸 알아야 합니다. 우리는 어떤 신호를 사인파의 합으로 표현할 때, 이 방법을 푸리에 급수(Series)라고 합니다.. 예로, 아래에 두개의 사인파의 합으로 이루어진 파형의 그래프가 있습니다.

.. image:: ../_images/summing_sinusoids.svg
   :align: center
   :target: ../_images/summing_sinusoids.svg
   :alt: 푸리에 급수를 증명하는 - 어떻게 두 사인파의 합이 파형을 형성하는지 보여주는 예입니다.
   
그리고 다른 예시 또한 있습니다; 아래의 빨간색 곡선은 최대 10개의 사인파를 합산해서, 톱니와 같은 파형을 나타냅니다. 우리는 이것이 완벽한 재구성이 아님을 알 수 있는데 - 아마도 톱니파를 제현 하려면 무한한 사인파의 합이 필요하겠죠.

.. image:: ../_images/fourier_series_triangle.gif
   :scale: 70 %   
   :align: center
   :alt: 삼각파의 구성으로 된 푸리의 급수의 애니메이션(톱니 파형)
   
어떤 신호는 다른 신호보다 더 많은 사인파가 필요하고, 어떤 신호는 항상 무한의 합으로 표현 됨에도 유한의 수로 근사할 수 있습니다.

.. image:: ../_images/fourier_series_arbitrary_function.gif
   :scale: 70 %   
   :align: center  
   :alt: 정사각형 펄스로 구성된 임의의 함수의 푸리에 급수 분해 애니메이션

신호를 삼각 함수의 파형으로 분해하는 방법을 이해하려면, 먼저 사인파의 세가지 속성에 대해서 알아 봐야 합니다.
#. 진폭 (Amplitude)
#. 진동수 (Frequency, 주파수)
#. 위상 (Phase)

**진폭**은 파형의 세기를 나타내는 반면, **진동수**는 파형이 1초에 몇번 흔들리는 지를 나타냅니다. **위상** 은 사인파가 얼마나 지나갔는지 나타내기 위해 사용되는데, 그 범위는 0도에서 360도입니다. (or 0 to :math:`2\pi`), 하지만 동일한 주파수를 가진 두 신호가 서로 30도 정도 위상이 어긋 나는 등등, 각 속성은 어떠한 의미를 "상대적으로" 가지게 됩니다.

.. image:: ../_images/amplitude_phase_period.svg
   :align: center
   :target: ../_images/amplitude_phase_period.svg
   :alt: 사인파(일명 Sinusoid)의 진폭, 위상 및 주파수에 대한 참조 다이어그램
   
이 시점에서 여러분들은 "신호"는 본질적으로 함수일 뿐이며 일반적으로 "시간 경과에 따라"(즉, x축)로 표현된다는 것을 깨달았을지도 모릅니다. 기억하기 쉬운 또 다른 속성은 **주기**로, 이는 **주파수**의 역수입니다. 정현파(깔끔한 모양의 사인파) 의 **주기**는 파동이 한 사이클을 완료하는 데 걸리는 시간을 초 단위로 나타냅니다. 따라서 주파수의 단위는 1/초 또는 Hz입니다.
   
신호를 사인파의 합으로 분해하면 각 사인파는 특정 **진폭**, **위상**, **주파수**를 갖게 됩니다. 각 사인파의 **진폭**은 원래 신호에 **주파수**가 얼마나 강하게 존재했는지 알려줍니다. 현재로서는 **위상**에 대해 너무 걱정하지 마세요. Sin()와 Cos()의 유일한 차이점은 위상 이동(시간 이동)뿐이라는 사실을 깨닫는 것 외에는 말이죠. (주석 : sin(x+(pi/2)) == cos(x))

푸리에 급수를 이해하기 위해서는 실제 방정식 보다는 기본 개념을 이해하는 것이 더 중요하긴 합니다만, 만약 산술적인 흥미가 들어 방정식에 관심이 있으시다면 - 아래 Wolfram의 간결한 살명을 참고해 주세요.: https://mathworld.wolfram.com/FourierSeries.html.

********************
시간-주파수 쌍
********************

우리는 신호를 여러 속성을 가진 사인파로 표현할 수 있다는 것을 확인했습니다. 이제 주파수 영역에서 신호를 플롯하는(나타내는) 방법을 배워봅시다. 시간 영역은 대게 신호가 시간에 따라 어떻게 변하는지 보여주는 반면, 주파수 영역은 신호가 얼마나 어느 주파수에 있는지 표시합니다. X축이 시간이 되는 대신 주파수가 됩니다. 주어진 신호를 시간 *주파수와 *주파수 모두에 플롯할 수 있습니다. 먼저 몇 가지 간단한 예를 살펴보겠습니다.

아래는 진동수 f의 사인파가 시간 도메인, 그리고 주파수 도메인에서 나타나는 모습입니다.:

.. image:: ../_images/sine-wave.png
   :scale: 70 % 
   :align: center
   :alt: 주파수 도메인에서는 한 펄스로 나타나는, 사인파의 시간-주파수 쌍

시간 영역은 매우 친숙해 보여야 합니다. 늘 그렇 듯이, 진동하는 함수입니다. 주기의 어느 지점에서 시작되는지 또는 얼마나 오래 지속되는지에 대해서는 걱정하지 마세요. 신호에는 **단일 주파수**가 있으므로 주파수 영역에서 단일 스파이크/피크가 표시됩니다. 그리고 사인파가 진동하는 주파수가, 주파수 영역에서 임펄스의 형태로 나타나는 것을 볼 수 있습니다. 이와 같은 스파이크의 수학적 이름을 "임펄스"라고 합니다.

이제 만약, 시간 영역에서 임펄스가 난다면 어떨까요? 그러니까, 누군가가 손뼉을 치거나 망치로 못을 내리 치는 소리 말입니다. 이때 이 시간-주파수 쌍은 직관성이 조금 떨어집니다.

.. image:: ../_images/impulse.png
   :scale: 70 % 
   :align: center  
   :alt: 시간 도메인에서의 임펄스에 대한 푸리에 시간-주파수 쌍, 이때 가로축은 주파수 성분입니다.

보시다시피 시간 영역의 스파이크/임펄스는 주파수 영역에서 평평하며 이론적으로 모든 주파수를 포함합니다. 시간 영역에서는 이론적으로 완벽한 임펄스(디렉-델타 함수와 비슷한 것)은 없습니다. 그리고 사인파와 마찬가지로 시간 영역에서 임펄스가 어디에서 발생하는지는 중요하지 않습니다. 여기서 중요한 점은 시간 영역의 빠른 변화로 인해 많은 주파수가 발생한다는 것입니다.

다음으로는, 사각형 파동의 시간 및 주파수 영역 그래프를 살펴보겠습니다:

.. image:: ../_images/square-wave.svg
   :align: center 
   :target: ../_images/square-wave.svg
   :alt: 사각파의 시간-주파수 푸리에 쌍은 주파수 영역에서 싱크 함수(sinc, 즉 sin(x)/x 함수)입니다.

이것도 직관적으로 이해하기는 어렵지만, 주파수 영역을 보면 강한 스파이크가 나타나는 것을 볼 수 있습니다. 이 스파이크는 사각파의 주파수 위치에 있지만, 더 높은 주파수로 갈수록 추가적인 스파이크들이 나타납니다. 이는 앞선 예시처럼 시간 영역에서의 급격한 변화 때문입니다. 하지만 주파수 영역이 평평한(flat) 것은 아니며, 일정한 간격으로 스파이크가 존재하고 그 크기는 서서히 감소합니다 (비록 무한히 이어지긴 하지만요). 시간 영역의 사각파는 주파수 영역에서 sin(x)/x 형태의 패턴, 즉 싱크 함수(sinc function)를 갖습니다.

이젠 시간 영역에서 일정한 신호를 가지면 어떻게 될까요? 정답은, 일정한 신호는 주파수가 없다는 겁니다..   아래를 봅시다:

.. image:: ../_images/dc-signal.png
   :scale: 80 % 
   :align: center 
   :alt: DC 신호의 시간-주파수 푸리에 쌍은 주파수 영역에서 0 Hz에 위치한 임펄스(Impulse)입니다.

주파수가 없기 때문에 주파수 영역에서는 0 Hz에서 스파이크가 나타납니다. 곰곰히 생각해보면 당연한 일입니다. 주파수 영역이 "비어 있다(empty)"는 것은 신호 자체가 없는 경우(즉, 시간 영역이 0으로 가득 찼을 때)만 해당하는 일이니까요. 우리는 주파수 영역의 0 Hz를 "DC"라고 부르는데, 이는 시간 영역에서 DC 신호(변하지 않는 일정한 신호)가 원인이기 때문입니다. 또한, 시간 영역에서 DC 신호의 진폭을 키우면 주파수 영역의 0 Hz 스파이크 역시 커진다는 점도 주목해야 합니다.

나중에 주파수 영역 플롯에서 y축이 정확히 무엇을 의미하는지 배우겠지만, 지금은 그것을 시간 영역 신호에 해당 주파수가 얼마나 포함되어 있는지를 나타내는 일종의 진폭(amplitude)이라고 생각하면 됩니다.
   
*****************
푸리에 변환
*****************

수학적으로, 시간 영역에서 주파수 영역으로, 또는 그 반대로 변환할 때 사용하는 "변환"을 푸리에 변환(Fourier Transform)이라고 합니다. 푸리에 변환은 다음과 같이 정의됩니다:

.. math::
   X(f) = \int x(t) e^{-j2\pi ft} dt

어떤 신호 :math:`x(t)` 에 대해 우리는 주파수 영역 버전, :math:`X(f)`버전을 이 공식을 통해 얻을 수 있습니다.  우리는 시간 버전의 함수를 :math:`x(t)` 또는 :math:`y(t)`로 표현할 거고, 거기에 상응하는 주파수 버전은 :math:`X(f)` 그리고 :math:`Y(f)`으로 표현할 겁니다. 아, :math:`t` 는 시간을 나타내고, :math:`f`는 주파수를 나타낸다는걸 기억해 주세요. :math:`j` 는 단순히 복소수의 단위이고, 아마도 고등학교 수학 시간때 :math:`i` 라고 표현했을 겁니다.  공학과 컴퓨터 과학에서는 "i"가 전류(current)를 의미하는 경우가 많고, 프로그래밍에서는 반복자(iterator)로 자주 사용되기 때문에 "j"를 허수 단위로 사용합니다.

더불어 주파수 영역에서 시간 영역으로 되돌아가는 변환은 부호만 반대일 뿐 거의 동일합니다:

.. math::
   x(t) = \int X(f) e^{j2\pi ft} df

알아 두셔야 할게, 다른 책이나 교과서에서는 :math:`w`를 :math:`2\pi f`를 대체헤서 사용합니다. 이때  :math:`w` 는 rad/s의 각속도 이고, 반면 :math:`f` 는 hz로 표기되는 진동수 입니다. 그러니까 결론은, 여러분들이 알아여 할건 아래와 같습니다.

.. math::
   \omega = 2 \pi f

비록 이 공식은 :math:`2 \pi`를 많은 공식에 집어넣긴 합니다만, 대부분의 SDR 및 RF 신호 처리에서는 Hz(헤르츠)를 사용하기 때문에, 주파수를 Hz 단위로 유지하는 것이 더 쉽고 편리합니다.

위의 푸리에 변환 식은 연속(continuous) 형태로, 주로 수학 문제에서만 보게 됩니다. 실제 코드에서 구현되는 것은 이산(discrete) 형태이며, 이산 형태가 훨씬 더 현실적인 구현(Discrete Fourier Transform)에 가깝습니다:

.. math::
   X_k = \sum_{n=0}^{N-1} x_n e^{-\frac{j2\pi}{N}kn}
   
주요 차이점은 적분 기호를 합산 기호(Σ)로 바꾼 것이라는 점에 주목하세요. 이때 인덱스스 :math:`k'는 0부터 N-1 까지입니다.

이런 식들이 딱히 의미 있게 느껴지지 않아도 괜찮습니다. 사실 우리는 DSP와 SDR로 멋진 일을 하기 위해 이런 식들을 직접 사용할 필요는 없습니다!

*************************
시간-주파수의 성질
*************************

앞서 우리는 신호가 시간 영역과 주파수 영역에서 어떻게 나타나는지 예제를 통해 살펴보았습니다. 이제 우리는 다섯 가지 중요한 "푸리에 성질(Fourier properties)"을 다룰 것입니다. 이 성질들은 시간 영역 신호에 대해 ___를 하면, 주파수 영역 신호에서는 ___가 일어난다는 것을 알려줍니다. 이러한 성질들은 실제로 시간 영역 신호를 처리할 때 어떤 종류의 디지털 신호 처리(DSP)를 하게 될지를 이해하는 데 중요한 통찰을 줄 것입니다.

1. 선형성(Linearity) 성질:

.. math::
   a x(t) + b y(t) \leftrightarrow a X(f) + b Y(f)

이 성질은 아마 가장 이해하기 쉬운 성질일 것입니다. 시간 영역에서 두 신호를 더하면, 주파수 영역에서도 두 주파수 신호가 더해진 형태로 나타납니다. 또한, 시간 영역이나 주파수 영역 중 어느 한쪽에서 신호에 스케일링 계수를 곱하면, 다른 영역에서도 동일한 계수로 스케일이 조정됩니다. 이 성질의 유용함은 나중에 여러 신호를 합성(add)할 때 더욱 분명해질 것입니다.

2. 주파수 이동(Shift) 성질:

.. math::
   e^{2 \pi j f_0 t}x(t) \leftrightarrow X(f-f_0)

x(t)의 왼쪽에 있는 항은 우리가 "복소 사인파(complex sinusoid)" 또는 "복소 지수 함수(complex exponential)"라고 부르는 것 입니다. 지금 단계에서는, 이것이 본질적으로 주파수 :math:`f_0`에서의 사인파라고만 이해하면 충분합니다.  그리고 이 성질이 시사하는 바는, 만약 우리가 신호 :math:`x(t)` 를 가져다가 사인파로 곱하면은, 주파수 도메인에서는 특정 주파수 f_0에 의해 옮겨진 것을 반영한 :math:`X(f)'를 얻는다는 것 입니다. 그리고 이러한 현상은 아래와 같이 시각화 하기에 상당히 좋습니다:

.. image:: ../_images/freq-shift.svg
   :align: center 
   :target: ../_images/freq-shift.svg
   :alt: 주파수 영역에서 신호의 주파수 이동을 나타내면 다음과 같은 모습입니다.

주파수 이동(Frequency shift)은 DSP에서 매우 중요한 개념입니다. 왜냐하면 우리는 다양한 이유로 신호를 주파수 상에서 위아래로 이동시키고 싶기 때문입니다. 이 성질은 신호를 사인파와 곱함으로써 주파수를 이동시키는 방법을 알려줍니다.

이 성질을 시각적으로 이해하는 또 다른 방법은 다음과 같습니다:

.. image:: ../_images/freq-shift-diagram.svg
   :align: center
   :target: ../_images/freq-shift-diagram.svg
   :alt: 사인파(또는 사인 곡선)와 곱하여 주파수를 이동시키는 과정의 시각적 표현
   
3. 시간 영역에서의 배율화(스케일링):

.. math::
   x(at) \leftrightarrow X\left(\frac{f}{a}\right)

식의 왼쪽을 보면 시간 영역에서 신호 :math:x(t)를 스케일링(늘리거나 줄이는)하고 있는 것을 볼 수 있습니다. 아래 예시는 시간 영역에서 신호를 스케일링했을 때, 주파수 영역에서는 각각 어떻게 변하는지를 보여줍니다.

.. image:: ../_images/time-scaling.svg
   :align: center
   :target: ../_images/time-scaling.svg
   :alt: 시간 스케일링에 따른 푸리에 변환 성질을 시간 영역과 주파수 영역에서 나타낸 그림

시간 영역에서 스케일링을 한다는 것은 신호를 x축(시간 축) 방향으로 압축하거나 늘리는 것을 의미합니다. 이 성질이 말해주는 것은, 시간 영역에서 신호를 스케일링하면 주파수 영역에서는 그 반대 방향으로 스케일링이 일어난다는 점입니다.

예를 들어, 데이터를 더 빠르게 전송하면 더 많은 대역폭을 사용해야 합니다. 이 성질은 왜 데이터 전송 속도가 높아질수록 더 많은 대역폭(스펙트럼)이 필요한지를 설명해줍니다. 만약 시간-주파수 스케일링이 반비례가 아니라 비례 관계였다면, 이동통신사들은 원하는 만큼 초당 비트를 전송하면서도 수조 원을 주파수 경매에 쓰지 않아도 됐을 겁니다. 하지만 현실은 그렇지 않습니다.

이 성질에 익숙한 사람이라면 식에서 스케일링 계수가 빠져 있다는 것을 눈치챌 수 있습니다. 하지만 설명의 단순함을 위해 일부러 생략한 것이며, 실제로는 큰 차이를 만들지 않습니다.

(역주 : 정보를 단시간에 보내기 위해 X축 스케일링을 하면, 주파수 도메인에서는 반대로 스케일링이 늘어나 대역폭이 늘어진다는 소리입니다. 신호를 파동에 실어 보내기 위해서는 진폭을 변조하거나(AM) 아니면 주파수 자체를 옮겨 다니거나(FM) 위상을 변조해야 하는데(QM or QAM) 이 과정에서 주파수 영역이 변동되기에 대역폭이 늘어집니다.)

4. 시간 영역에서의 컨볼루션(대합):

.. math::
   \int x(\tau) y(t-\tau) d\tau  \leftrightarrow X(f)Y(f)

이것을 **컨볼루션 성질(convolution property)** 이라고 부릅니다. 시간 영역(time domain)에서 우리는 x(t)와 y(t)를 컨볼루션(convolution)하고 있기 때문입니다. 아직 컨볼루션 연산에 대해 잘 모를 수도 있으니, 지금은 일단 교차상관(cross-correlation)과 비슷하다고 생각해도 됩니다. (:ref:`this section <convolution-section>` 에서 컨볼루션을 더 깊이 다루게 됩니다.)

시간 영역에서 신호를 컨볼루션하면, 이는 두 신호의 주파수 영역(frequency domain) 표현을 곱하는 것과 같습니다. 이것은 단순히 두 신호를 더하는 것과는 매우 다릅니다. 신호를 더하면, 앞서 본 것처럼 그냥 주파수 영역에서도 단순히 더해질 뿐 특별한 일이 일어나지 않습니다. 하지만 두 신호를 컨볼루션하면, 마치 새로운 세 번째 신호를 만들어내는 것과 같습니다.

컨볼루션은 DSP(디지털 신호 처리)에서 가장 중요한 기법이지만, 이를 완전히 이해하기 위해서는 먼저 필터(filter)가 어떻게 동작하는지 알아야 합니다.

넘어가기 전에, 이 성질이 왜 그렇게 중요한지 간단히 설명해보겠습니다. 만약 당신이 수신하고 싶은 하나의 신호가 있고, 그 옆에 간섭 신호가 있다고 가정해봅시

.. image:: ../_images/two-signals.svg
   :align: center
   :target: ../_images/two-signals.svg
   
프로그래밍에서 **마스킹(masking)** 이라는 개념은 자주 사용됩니다. 여기에서도 그 개념을 적용해 봅시다. 만약 아래와 같은 마스크를 만들고, 그것을 위의 신호에 곱해서 우리가 원하지 않는 신호를 가려낼 수 있다면 어떨까요?


.. image:: ../_images/masking.svg
   :align: center
   :target: ../_images/masking.svg

우리는 보통 시간 영역(time domain)에서 DSP 연산을 수행합니다. 따라서 컨볼루션 성질(convolution property)을 활용해, 시간 영역에서 마스킹(masking)을 어떻게 구현할 수 있는지 살펴봅시다.

x(t)를 우리가 수신한 신호라고 합시다. 주파수 영역(frequency domain)에서 적용하고자 하는 마스크를 Y(f)라고 하면, y(t)는 그 마스크의 시간 영역 표현이 됩니다. 그리고 x(t)와 y(t)를 컨볼루션하면, 우리가 원하지 않는 신호를 **걸러낼(filter out)** 수 있습니다.


.. tikz:: [font=\Large\bfseries\sffamily]
   \definecolor{babyblueeyes}{rgb}{0.36, 0.61, 0.83}
   \draw (0,0) node[align=center,babyblueeyes]           {E.g., our received signal};
   \draw (0,-4) node[below, align=center,babyblueeyes]   {E.g., the mask}; 
   \draw (0,-2) node[align=center,scale=2]{$\int x(\tau)y(t-\tau)d\tau \leftrightarrow X(f)Y(f)$};   
   \draw[->,babyblueeyes,thick] (-4,0) -- (-5.5,-1.2);
   \draw[->,babyblueeyes,thick] (2.5,-0.5) -- (3,-1.3);
   \draw[->,babyblueeyes,thick] (-2.5,-4) -- (-3.8,-2.8);
   \draw[->,babyblueeyes,thick] (3,-4) -- (5.2,-2.8);
   :xscale: 70

그리고 아마 필터링에 대해 더 얘기하면, Convolution에 대해서 더 이해가 잘 되실 겁니다.

5. 주파수 영역에서의 컨볼루션 성질:

마지막으로, 컨볼루션 성질은 반대 방향으로도 성립한다는 점을 짚고 넘어가고 싶습니다. 다만 시간 영역에서의 컨볼루션만큼 자주 사용되지는 않습니다:

.. math::
   x(t)y(t)  \leftrightarrow  \int X(g) Y(f-g) dg

이 외에도 여러 성질이 있지만, 위에서 다룬 다섯 가지가 가장 핵심적이라고 생각합니다. 각 성질에 대한 증명을 일일이 따라가 보진 않았지만, 중요한 점은 이러한 수학적 성질들을 통해 실제 신호를 분석하고 처리할 때 어떤 일이 일어나는지 통찰을 얻을 수 있다는 것입니다.

방정식 자체에 집착하지 말고, 각 성질이 설명하는 바를 이해하는 데 집중하세요.

******************************
고속 푸리에 변환 (Fast Fourier Transform, FFT)
******************************

이제 다시 푸리에 변환(Fourier Transform)으로 돌아가 봅시다. 앞에서 이산 푸리에 변환(Discrete Fourier Transform, DFT)의 식을 보여드렸지만, 실제로 코드를 작성할 때 99.9%의 경우에는 **FFT 함수, `fft()`** 를 사용하게 될 것입니다.

빠른 푸리에 변환(Fast Fourier Transform, FFT)은 단순히 이산 푸리에 변환을 계산하기 위한 알고리즘일 뿐입니다. 수십 년 전에 개발되었고, 구현 방식에는 여러 변형이 있지만 여전히 이산 푸리에 변환을 계산하는 데 있어 사실상 표준으로 자리 잡고 있습니다. 이름에 "Fast"를 붙인 걸 고려하면 다행이죠.

FFT는 입력 하나와 출력 하나를 가지는 함수입니다. 이 함수는 신호를 시간 영역(time domain)에서 주파수 영역(frequency domain)으로 변환합니다:


.. image:: ../_images/fft-block-diagram.svg
   :align: center
   :target: ../_images/fft-block-diagram.svg
   :alt: FFT는 시간을 입력으로, 주파수 영역을 1:1로 뱉어주는 함수입니다.
   
이 교재에서는 1차원 FFT만 다룰 것입니다. (2차원 FFT는 영상 처리(image processing)나 기타 응용 분야에서 사용됩니다.)

우리의 목적에 맞게, FFT 함수를 다음과 같이 생각하면 됩니다: **입력은 샘플 벡터 하나**, **출력은 그 샘플 벡터의 주파수 영역 표현 하나다**. 그리고 출력의 크기는 항상 입력의 크기와 동일합니다. 예를 들어 1,024개의 샘플을 FFT에 넣으면, 출력도 1,024개가 나옵니다.

여기서 헷갈리기 쉬운 부분은 출력이 항상 주파수 영역에 있다는 점입니다. 따라서 이것을 그래프로 그린다고 했을 때, x축의 "범위(span)"는 시간 영역 입력 샘플 수에 따라 달라지지 않습니다.

이제 입력 배열과 출력 배열을, 그리고 각 인덱스가 가지는 단위를 함께 살펴보며 이를 시각적으로 이해해봅시다:


.. image:: ../_images/fft-io.svg
   :align: center
   :target: ../_images/fft-io.svg
   :alt: FFT 함수의 입력(초, seconds)과 출력(대역폭, bandwidth) 형식을 보여주는 참조 다이어그램

출력이 주파수 영역(frequency domain)에 있기 때문에, x축의 범위(span)는 샘플레이트(sample rate)에 의해 결정됩니다. 샘플레이트에 대해서는 다음 장에서 다룰 것입니다.

입력 벡터에 더 많은 샘플을 사용할수록 주파수 영역에서의 해상도(resolution)가 좋아집니다. (물론 동시에 더 많은 샘플을 처리하게 되기도 합니다.) 그러나 입력을 크게 한다고 해서 실제로 더 많은 주파수를 "보게 되는" 것은 아닙니다. 더 많은 주파수를 보기 위한 유일한 방법은 샘플레이트를 늘리는 것, 즉 샘플 주기(:math:`\Delta t`)를 줄이는 것입니다.

그렇다면 이 출력을 실제로 어떻게 그릴 수 있을까요? 예를 들어 샘플레이트가 1초당 100만 샘플(1 MHz)이라고 해봅시다. 다음 장에서 배우겠지만, 이 경우 FFT에 몇 개의 샘플을 넣든지 간에 최대 0.5 MHz까지만 볼 수 있습니다. FFT의 출력이 표현되는 방식은 다음과 같습니다:


.. image:: ../_images/negative-frequencies.svg
   :align: center
   :target: ../_images/negative-frequencies.svg
   :alt: 음의 주파수?

항상 동일한 규칙이 적용됩니다. FFT의 출력은 언제나 :math:`\text{-} f_s/2` 에서 :math:`f_s/2` 까지를 보여주며, 여기서 :math:`f_s` 는 샘플레이트(sample rate)입니다. 즉, 출력에는 항상 음수 부분과 양수 부분이 존재합니다. 입력 신호가 복소수(complex)라면 음수 부분과 양수 부분이 서로 다르지만, 입력이 실수(real)라면 두 부분은 동일합니다.

주파수 간격(frequency interval)에 대해서는, 각 빈(bin)이 :math:`f_s/N` Hz에 해당합니다. 따라서 FFT에 더 많은 샘플을 넣으면 출력에서 더 세밀한 해상도(granular resolution)를 얻을 수 있습니다.

입문자라면 무시해도 되는 아주 작은 세부 사항이 하나 있습니다. 수학적으로는 마지막 인덱스가 *정확히* :math:`f_s/2` 에 대응하지 않고, 실제로는 :math:`f_s/2 - f_s/N` 에 해당합니다. 하지만 :math:`N` 이 충분히 크면 사실상 :math:`f_s/2` 와 같다고 볼 수 있습니다.


********************
음의 주파수 (Negative Frequencies)
********************

도대체 **음의 주파수(negative frequency)** 란 무엇일까요? 우선, 이는 복소수(complex number, 즉 허수 imaginary numbers)를 사용할 때 등장하는 개념이라는 점만 알아두면 됩니다. 실제로 RF 신호를 송수신(transmit/receive)할 때 "음의 주파수"라는 것이 존재하는 것은 아니고, 단지 우리가 쓰는 **표현 방식(representation)**일 뿐입니다. (역주 : 반송파(Carrier Wave, 중심 주파수)를 기준으로 상대적으로 낮은 쪽을 "음의 주파수"라고 부릅니다. 실제로 -5khz 이런 주파수가 존재하지는 않습니다.)

이를 직관적으로 이해할 수 있는 방법을 하나 들어보겠습니다. SDR에 100 MHz(FM 라디오 대역)에 맞추도록 설정하고, 샘플레이트를 10 MHz로 한다고 합시다. 즉, 스펙트럼은 95 MHz부터 105 MHz까지 보게 되는 셈입니다. 여기에 세 개의 신호가 존재한다고 가정해봅시다:


.. image:: ../_images/negative-frequencies2.svg
   :align: center
   :target: ../_images/negative-frequencies2.svg
   
그래서 이제, SDR이 샘플을 보여 준다면 아래와 같이 됩니다:

.. image:: ../_images/negative-frequencies3.svg
   :align: center
   :target: ../_images/negative-frequencies3.svg
   :alt: 음의 주파수(negative frequencies)란, 라디오가 맞춘 중심 주파수(센터 주파수, center frequency 혹은 반송파 주파수, carrier frequency)보다 낮은 쪽에 위치한 주파수를 가리킬 뿐입니다.

앞서 SDR을 100 MHz에 맞췄다는 것을 기억해 둡시다. 그러면 약 97.5 MHz에 있던 신호는 디지털 표현에서는 -2.5 MHz로 나타나게 되는데, 이것이 기술적으로는 **음의 주파수(negative frequency)** 입니다. 실제로는 단지 중심 주파수보다 낮은 주파수일 뿐입니다. 이 부분은 샘플링(sampling)에 대해 더 배우고, 직접 SDR을 사용해보면서 경험을 쌓으면 훨씬 더 잘 이해될 것입니다.


수학적인 관점에서 **음의 주파수(negative frequency)** 는 복소 지수 함수 :math:`e^{2 j \pi f t}` 를 통해 살펴볼 수 있습니다. 주파수가 음수라면, 그것은 반대 방향으로 회전하는 복소 사인파(complex sinusoid)가 됩니다.


.. math::
   e^{2j \pi f t} = \cos(2 \pi f t) + j \sin(2 \pi f t) \quad \mathrm{\textcolor{blue}{blue}}

.. math::
   e^{2j \pi (-f) t} = \cos(2 \pi f t) - j \sin(2 \pi f t) \quad \mathrm{\textcolor{red}{red}}

.. image:: ../_images/negative_freq_animation.gif
   :align: center
   :scale: 75 %
   :target: ../_images/negative_freq_animation.gif
   :alt: 복소수에서 표현되는 양의 주파수와 음의 주파수에 대한 애니메이션

위에서 복소 지수(complex exponential)를 사용한 이유는, 단순히 :math:`\cos()` 나 :math:`\sin()` 함수만 쓰면 그 안에는 양의 주파수와 음의 주파수가 모두 섞여 있기 때문입니다. 이는 주파수 :math:`f`와 시간 :math:`t`에 대한 사인파(sinusoid)에 오일러 공식(Euler's formula)을 적용해보면 알 수 있습니다:


.. math::
   \cos(2 \pi f t) = \underbrace{\frac{1}{2} e^{2j \pi f t}}_\text{positive} + \underbrace{\frac{1}{2} e^{-2j \pi f t}}_\text{negative}

.. math::
   \sin(2 \pi f t) = \underbrace{\frac{1}{2j} e^{2j \pi f t}}_\text{positive} - \underbrace{\frac{1}{2j} e^{-2j \pi f t}}_\text{negative}

그래서 RF 신호 처리에서는 코사인/사인의 합보다 자연 로그의 밑을 가지고 표현하는 경우가 많습니다.

****************************
시간 순서는 중요하지 않다 (Order in Time Doesn't Matter)
****************************

FFT는 한 번에 많은 샘플을 가지고 수행된다는 점을 떠올려 봅시다. 즉, 한 시점의 단일 샘플만으로는 주파수 영역을 관찰할 수 없고, 일정 시간 구간(span)에 걸친 다수의 샘플이 필요합니다.

FFT 함수는 입력 신호를 일종의 “재배열(mix around)” 하여 출력으로 변환하는데, 이때 출력은 시간 영역이 아니라 다른 척도(scale)와 단위를 갖는 주파수 영역 신호가 됩니다.

이 두 영역의 차이를 체득하기 좋은 방법은, **시간 영역에서 사건이 일어나는 순서를 바꿔도 신호의 주파수 성분은 변하지 않는다**는 사실을 이해하는 것입니다. 예를 들어, 다음 두 신호에 대해 **단일 FFT**를 수행하면, 두 신호 모두에서 주파수 영역에는 같은 두 개의 스파이크(spikes)가 나타납니다. 왜냐하면 이 신호는 단지 서로 다른 두 주파수를 가진 사인파 두 개가 합쳐진 것이기 때문입니다.

사인파가 발생하는 순서를 바꿔도, 그것이 “서로 다른 두 주파수의 사인파”라는 사실은 변하지 않습니다. 다만 이는 두 사인파가 FFT에 입력되는 동일한 시간 구간 안에 존재한다는 가정하에서 성립합니다. 만약 FFT 크기를 줄이고 여러 번 FFT를 수행한다면(이는 스펙트로그램(Spectrogram) 부분에서 다루게 될 것입니다), 두 사인파가 시간에 따라 구분되어 나타나는 것을 확인할 수 있습니다.


.. image:: ../_images/fft_signal_order.png
   :scale: 50 % 
   :align: center
   :alt: 일정한 샘플 집합에 대해 FFT를 수행할 때, 그 안에서 서로 다른 주파수들이 시간적으로 어떤 순서로 나타났는지는 최종 FFT 출력에 영향을 주지 않습니다.


기술적으로는, 사인파가 시간적으로 이동(time-shift)하면 FFT 값의 위상(phase)이 변하게 됩니다. 그러나 이 교재의 앞부분 몇 장에서는 주로 FFT의 크기(magnitude)에만 집중할 것입니다.


*******************
파이썬에서의 FFT (FFT in Python)
*******************

이제 FFT가 무엇인지, 그리고 그 출력이 어떻게 표현되는지 배웠으니 실제로 파이썬 코드를 살펴보며 Numpy의 FFT 함수 `np.fft.fft()`를 사용해봅시다.

가능하다면 여러분의 컴퓨터에서 전체 파이썬 콘솔이나 IDE를 사용하는 것을 권장합니다. 하지만 급할 때는 왼쪽 네비게이션 바 하단에 링크된 온라인 웹 기반 파이썬 콘솔을 이용해도 됩니다.

먼저 시간 영역(time domain)에서 신호를 하나 만들어야 합니다. 여러분도 직접 파이썬 콘솔에서 따라 해 보세요. 단순화를 위해 주파수 0.15 Hz의 간단한 사인파(sine wave)를 만들겠습니다. 샘플레이트(sample rate)는 1 Hz로 하여, 시간축에서는 0초, 1초, 2초, 3초 등과 같이 샘플링하게 됩니다.


.. code-block:: python

 import numpy as np
 t = np.arange(100)
 s = np.sin(0.15*2*np.pi*t)

이때 :code:`s`를 표현하면 이처럼 보일껍니다:

.. image:: ../_images/fft-python1.svg
   :target: ../_images/fft-python1.svg
   :align: center 

다음엔 Numpy에서 제공하는 fft 함수를 써봅시다.

.. code-block:: python

 S = np.fft.fft(s)

여기서 :code:`S`를 까보면, 많은 복소수들이 나올꺼에요.

.. code-block:: python

    S =  array([-0.01865008 +0.00000000e+00j, -0.01171553 -2.79073782e-01j,0.02526446 -8.82681208e-01j,  3.50536075 -4.71354150e+01j, -0.15045671 +1.31884375e+00j, -0.10769903 +7.10452463e-01j, -0.09435855 +5.01303240e-01j, -0.08808671 +3.92187956e-01j, -0.08454414 +3.23828386e-01j, -0.08231753 +2.76337148e-01j, -0.08081535 +2.41078885e-01j, -0.07974909 +2.13663710e-01j,...

힌트: 어떤 상황이든 간에 복소수(complex number)를 다루게 된다면, 크기(magnitude)와 위상(phase)을 계산해보면 훨씬 이해하기 쉽습니다. 우리도 바로 그렇게 해보고, 크기와 위상을 그래프로 그려보겠습니다.
대부분의 프로그래밍 언어에서 `abs()` 함수는 복소수의 크기를 구하는 함수입니다. 위상을 구하는 함수는 언어마다 다르지만, 파이썬에서는 NumPy의 :code:`np.angle()` 을 사용할 수 있습니다. 이 함수는 위상을 라디안(radian) 단위로 반환합니다.


.. code-block:: python

 import matplotlib.pyplot as plt
 S_mag = np.abs(S)
 S_phase = np.angle(S)
 plt.plot(t,S_mag,'.-')
 plt.plot(t,S_phase,'.-')

.. image:: ../_images/fft-python2.svg
   :target: ../_images/fft-python2.svg
   :align: center

현재 우리는 그래프에 x축을 따로 지정하지 않고 있으며, 단순히 배열의 인덱스(0부터 증가)를 x축으로 사용하고 있습니다.이때 수학적인 이유로 인해, FFT의 출력은 다음과 같은 형식을 가지게 됩니다:


.. image:: ../_images/fft-python3.svg
   :align: center
   :target: ../_images/fft-python3.svg
   :alt: FFT 시프트(중심을 0에 맞추는 것)을 수행하기 전, FFT 출력의 배열(배치) 형태
   
하지만 우리가 원하는 것은 0 Hz(DC)가 가운데에 오고, 음의 주파수들이 왼쪽에 배치되는 형태입니다. (이는 단순히 우리가 신호를 그렇게 시각화하는 데 익숙하기 때문입니다.) 따라서 FFT를 수행할 때마다 반드시 "FFT 시프트(FFT shift)"를 해주어야 합니다. 이는 단순한 배열 재배치 작업으로, 원형 시프트(circular shift)와 비슷하지만, 사실상 “이걸 저쪽으로 옮기고, 저걸 이쪽으로 옮기는” 정도의 정렬 작업입니다.

아래 다이어그램은 FFT 시프트 연산이 정확히 무엇을 하는지 보여줍니다:


.. image:: ../_images/fft-python4.svg
   :align: center
   :target: ../_images/fft-python4.svg
   :alt: FFT 시프트(FFT shift) 함수의 참조 다이어그램. 양의 주파수, 음의 주파수, 그리고 DC(0 Hz)가 어떻게 배치되는지를 보여줍니다.


다행히도 NumPy에는 FFT 시프트를 위한 함수 :code:`np.fft.fftshift()` 가 준비되어 있습니다.
따라서 :code:`np.fft.fft()` 줄을 다음과 같이 바꿔주면 됩니다:


.. code-block:: python

 S = np.fft.fftshift(np.fft.fft(s))

이제 x축 값과 라벨을 정해줄 필요가 있습니다. 단순화를 위해 샘플레이트를 1 Hz로 사용했다고 기억해 둡시다. 이 경우 주파수 영역 플롯의 왼쪽 끝은 -0.5 Hz, 오른쪽 끝은 0.5 Hz가 됩니다. 만약 지금 당장은 잘 이해가 안 되더라도, :ref:`sampling-chapter` 를 공부하면 분명히 이해될 것입니다.

따라서 샘플레이트가 1 Hz라는 가정을 그대로 두고, FFT 출력의 크기(magnitude)와 위상(phase)을 올바른 x축 라벨과 함께 플로팅해 보겠습니다. 아래는 이 파이썬 예제의 최종 버전과 그 출력 결과입니다:


.. code-block:: python

 import numpy as np
 import matplotlib.pyplot as plt
 
 Fs = 1 # Hz
 N = 100 # number of points to simulate, and our FFT size
 
 t = np.arange(N) # because our sample rate is 1 Hz
 s = np.sin(0.15*2*np.pi*t)
 S = np.fft.fftshift(np.fft.fft(s))
 S_mag = np.abs(S)
 S_phase = np.angle(S)
 f = np.arange(Fs/-2, Fs/2, Fs/N)
 plt.figure(0)
 plt.plot(f, S_mag,'.-')
 plt.figure(1)
 plt.plot(f, S_phase,'.-')
 plt.show()

.. image:: ../_images/fft-python5.svg
   :target: ../_images/fft-python5.svg
   :align: center


우리가 만든 사인파의 주파수인 0.15 Hz에서 스파이크(spike)가 나타난 것을 확인할 수 있습니다. 이는 FFT가 올바르게 동작했다는 의미입니다! 만약 사인파를 생성한 코드 내용을 모르고, 단순히 샘플 값의 리스트만 받았다고 하더라도, FFT를 이용해 주파수를 알아낼 수 있습니다. 또한 -0.15 Hz에서도 스파이크가 보이는 이유는 입력 신호가 복소 신호(complex)가 아니라 실수 신호(real)이었기 때문입니다. 이 부분은 뒤에서 더 자세히 다루게 될 것입니다.


******************************
Windowing
******************************

우리가 FFT(Fast Fourier Transform)를 사용하여 신호의 주파수 성분을 측정할 때,FFT는 입력으로 주어진 신호 조각이 *주기적(periodic)* 신호라고 가정합니다. 즉, 우리가 제공한 신호 조각이 무한히 반복된다고 생각하는 것이죠. 마치 잘라낸 신호의 마지막 샘플이 첫 번째 샘플과 연결되는 것처럼 동작합니다.

이러한 가정은 푸리에 변환(Fourier Transform)의 이론적 기반에서 비롯된 것입니다. 즉 첫 번째 샘플과 마지막 샘플 사이에 갑작스러운 전환이 생기지 않도록 해야 한다는 의미입니다. 왜냐하면 시간 영역에서의 갑작스러운 전환은 여러 주파수 성분으로 나타나기 때문입니다. 실제로는 마지막 샘플이 첫 번째 샘플과 연결되어 있지 않으므로 문제가 발생할 수 있습니다.

간단히 말해, 100개의 샘플에 대해 FFT를 수행한다고 할 때
:code:`np.fft.fft(x)` 를 사용한다면, :code:`x[0]` 과 :code:`x[99]` 의 값은 동일하거나 적어도 비슷해야 합니다.

이러한 주기적 성질을 보정하는 방법은 "윈도잉(windowing)"입니다. FFT를 수행하기 직전에 신호 조각(slice)에 윈도 함수(window function)를 곱해줍니다. 이때 윈도 함수란 양쪽 끝에서 0으로 점점 줄어드는 임의의 함수입니다. 결론적으로 이를 적용하면 신호 조각이 처음과 끝에서 0으로 시작하고 끝나면서 자연스럽게 이어지게 됩니다.

대표적인 윈도 함수로는 Hamming, Hanning, Blackman, Kaiser 등이 있습니다. 아무런 윈도잉을 적용하지 않는 경우에는 "직사각형(rectangular)" 윈도를 사용한다고 부릅니다. 이는 단순히 모든 값이 1인 배열을 곱하는 것과 동일하기 때문입니다.

아래 그림은 여러 윈도 함수의 모양을 보여줍니다:

.. image:: ../_images/windows.svg
   :align: center
   :target: ../_images/windows.svg
   :alt: 직사각형, 해밍, 해닝, 바틀렛, 블랙맨, 카이저 윈도의 시간 영역과 주파수 영역에서의 윈도 함수

초보자에게 가장 간단한 방법은 Hamming 윈도를 사용하는 것입니다. Python에서는 :code:`np.hamming(N)` 으로 생성할 수 있으며, 여기서 N은 배열의 원소 개수이자 FFT 크기입니다 위의 예제에서라면, FFT를 수행하기 직전에 윈도를 적용하면 됩니다. 즉, 두 번째 코드 줄 바로 뒤에 다음을 삽입합니다:

.. code-block:: python

   s = s * np.hamming(100)

혹시 잘못된 윈도를 고를까 걱정할 필요는 없습니다. Hamming, Hanning, Blackman, Kaiser 윈도 사이의 차이는, 윈도를 전혀 사용하지 않는 경우와 비교하면 아주 미미합니다. 이들 모두 양쪽 끝에서 0으로 수렴하며, 근본적인 문제를 해결해주기 때문입니다.



*******************
FFT Sizing
*******************
마지막으로 알아둘 것은 FFT 크기(FFT sizing)입니다. 가장 좋은 FFT 크기는 항상 2의 거듭제곱입니다. 이는 FFT가 구현되는 방식 때문입니다. 물론 2의 거듭제곱이 아닌 크기를 사용할 수도 있지만 속도가 느려집니다. 일반적으로는 128에서 4,096 사이의 크기를 자주 사용하지만, 더 큰 크기를 쓰는 것도 가능하긴 합니다.

현실적으로는 수백만, 수십억 개의 샘플로 이루어진 신호를 처리해야 하는 경우가 많습니다. 이럴 때는 신호를 여러 조각으로 나누어 많은 FFT를 수행해야 하며, 그 결과 출력도 여러 개가 나옵니다. 이 출력들을 평균 내거나, 혹은 시간에 따라 변화하는 모습을 그래프로 그릴 수 있습니다 (특히 신호가 시간에 따라 변할 때 유용합니다).

신호의 *모든* 샘플을 FFT에 넣지 않아도, 해당 신호의 주파수 영역 표현은 충분히 얻을 수 있습니다. 예를 들어, 어떤 신호에서 10만 개 샘플마다 1,024개 샘플만 FFT를 수행해도, 신호가 항상 켜져 있다면 주파수 영역 결과는 충분히 잘 보일 것입니다.


.. _spectrogram-section:

*********************
Spectrogram/Waterfall
*********************

스펙트로그램(spectrogram)은 시간에 따른 주파수 변화를 보여주는 플롯입니다. 즉, 여러 개의 FFT를 차곡차곡 쌓아놓은 것이라 볼 수 있습니다. (주파수를 가로축에 두고 싶다면 FFT들을 세로로 쌓는 셈입니다). 실시간으로 표시할 수도 있는데, 이를 흔히 워터폴(waterfall)이라고 부릅니다. 스펙트럼 분석기(spectrum analyzer)라는 장비는 바로 이러한 스펙트로그램/워터폴을 보여주는 장비입니다.

아래 그림은 IQ 샘플 배열을 어떻게 잘라내어 스펙트로그램을 구성하는지 보여줍니다:

.. image:: ../_images/spectrogram_diagram.svg
   :align: center
   :target: ../_images/spectrogram_diagram.svg
   :alt: 스펙트로그램(워터폴) 다이어그램 – FFT 조각들을 쌓아 올려 시간-주파수 플롯을 만드는 방식

스펙트로그램은 2차원 데이터를 그리는 것이므로, 사실상 3차원 플롯입니다. 따라서 FFT의 크기(즉, 우리가 표현하고자 하는 값)를 색상으로 나타내기 위해 컬러맵(colormap)을 사용합니다.

아래는 예시 스펙트로그램으로, 가로축(x축)은 주파수, 세로축(y축)은 시간입니다. 파란색은 에너지가 가장 낮은 부분을, 빨간색은 가장 높은 부분을 의미합니다. 중앙의 DC(0 Hz)에서 강한 스파이크가 나타나 있으며, 그 주위로 변동하는 신호가 보입니다. 파란색 부분은 잡음 바닥(noise floor)을 나타냅니다.

.. image:: ../_images/waterfall.png
   :scale: 120 %
   :align: center

기억하세요, 스펙트로그램은 단순히 FFT 결과들을 행(row)으로 차곡차곡 쌓아 놓은 것입니다. 각 행은 하나의 FFT 결과(정확히는 그 크기 값)를 의미합니다. 따라서 입력 신호를 FFT 크기 단위로 잘라내야 합니다 (예: 1024 샘플 단위).

이제 스펙트로그램을 생성하는 코드를 살펴보기 전에, 사용할 예시 신호를 먼저 만들어 보겠습니다.이는 단순히 백색 잡음 속에 있는 하나의 톤(tone)입니다:

.. code-block:: python


 import numpy as np
 import matplotlib.pyplot as plt
 
 sample_rate = 1e6
 
 # Generate tone plus noise
 t = np.arange(1024*1000)/sample_rate # time vector
 f = 50e3 # freq of tone
 x = np.sin(2*np.pi*f*t) + 0.2*np.random.randn(len(t))

이건 실제로 시간 도메인에서 어떻게 보이는지 입니다 (처음 200개 샘플):

.. image:: ../_images/spectrogram_time.svg
   :align: center
   :target: ../_images/spectrogram_time.svg

파이썬에서는, 아래와 같이 스펙토그램을 생성할 수 있습니다:

.. code-block:: python

 # simulate the signal above, or use your own signal
  
 fft_size = 1024
 num_rows = len(x) // fft_size # // is an integer division which rounds down
 spectrogram = np.zeros((num_rows, fft_size))
 for i in range(num_rows):
     spectrogram[i,:] = 10*np.log10(np.abs(np.fft.fftshift(np.fft.fft(x[i*fft_size:(i+1)*fft_size])))**2)
 
 plt.imshow(spectrogram, aspect='auto', extent = [sample_rate/-2/1e6, sample_rate/2/1e6, len(x)/sample_rate, 0])
 plt.xlabel("Frequency [MHz]")
 plt.ylabel("Time [s]")
 plt.show()

위 코드는 다음과 같은 결과를 만듭니다. 시간에 따른 변화가 없기 때문에 아주 흥미로운 스펙트로그램은 아니죠. 실수(real) 신호를 시뮬레이션했기 때문에 톤이 두 개 보이는데, 실수 신호의 전력 스펙트럼 밀도(PSD)는 양의 주파수 측과 동일한 음의 주파수 측이 항상 존재(복소수 공액 대칭)하기 때문입니다. 더 흥미로운 스펙트로그램 예시는 https://www.IQEngine.org 에서 확인해 보세요!


.. image:: ../_images/spectrogram.svg
   :align: center
   :target: ../_images/spectrogram.svg

*********************
FFT Implementation
*********************

NumPy가 이미 FFT를 구현해주고 있지만, 내부적으로 어떻게 동작하는지 기본 원리를 알아두는 것도 좋습니다. 가장 널리 사용되는 FFT 알고리즘은 Cooley-Tukey FFT 알고리즘으로, 최초에는 1805년경 카를 프리드리히 가우스(Carl Friedrich Gauss)에 의해 발명되었고, 이후 1965년 제임스 쿠울리(James Cooley)와 존 터키(John Tukey)가 재발견하여 널리 알려지게 되었습니다.

이 알고리즘의 기본 버전은 FFT 크기가 2의 거듭제곱일 때 작동하며, 복소수 입력을 대상으로 하지만 실수 입력에도 적용할 수 있습니다. 이 알고리즘의 기본 구성 요소는 버터플라이(butterfly)라고 불리며, 이는 사실상 크기 2의 FFT로 두 번의 곱셈과 두 번의 덧셈으로 이루어집니다:

.. image:: ../_images/butterfly.svg
   :align: center
   :target: ../_images/butterfly.svg
   :alt: Cooley-Tukey FFT 알고리즘 버터플라이(radix-2)

또는 다음과 같이 표현할 수 있습니다:

.. math::
   y_0 = x_0 + x_1 w^k_N

   y_1 = x_0 - x_1 w^k_N

여기서 :math:`w^k_N = e^{j2\pi k/N}` 는 트위들 팩터(twiddle factor)라고 불리며, :math:`N`은 서브 FFT의 크기이고 :math:`k`는 인덱스입니다. 입력과 출력은 복소수임을 주의하세요. 예를 들어 :math:`x_0` 은 0.6123 - 0.5213j 같은 복소수일 수 있으며, 덧셈과 곱셈도 모두 복소수 연산입니다.

이 알고리즘은 재귀적으로 작동하며, 절반씩 나누어 결국 일련의 버터플라이 연산만 남게 됩니다. 아래 그림은 크기 8 FFT의 과정을 나타낸 것입니다:

.. image:: ../_images/butterfly2.svg
   :align: center
   :target: ../_images/butterfly2.svg
   :alt: Cooley-Tukey FFT 알고리즘 크기 8 예시

이 패턴에서 각 열은 병렬로 수행할 수 있는 연산 집합을 의미하며, :math:`log_2(N)` 단계가 수행됩니다. 그렇기 때문에 FFT의 계산 복잡도는 O(:math:`N \log N`)이지만, DFT는 O(:math:`N^2`)입니다.

수식보다 코드를 선호하는 사람들을 위해, 아래에는 FFT의 간단한 Python 구현과 함께, 톤과 잡음으로 구성된 예제 신호를 사용하여 FFT를 시험해볼 수 있는 코드가 나와 있습니다.


.. code-block:: python

 import numpy as np
 import matplotlib.pyplot as plt
 
 def fft(x):
     N = len(x)
     if N == 1:
         return x
     twiddle_factors = np.exp(-2j * np.pi * np.arange(N//2) / N)
     x_even = fft(x[::2]) # yay recursion!
     x_odd = fft(x[1::2])
     return np.concatenate([x_even + twiddle_factors * x_odd,
                            x_even - twiddle_factors * x_odd])
 
 # Simulate a tone + noise
 sample_rate = 1e6
 f_offset = 0.2e6 # 200 kHz offset from carrier
 N = 1024
 t = np.arange(N)/sample_rate
 s = np.exp(2j*np.pi*f_offset*t)
 n = (np.random.randn(N) + 1j*np.random.randn(N))/np.sqrt(2) # unity complex noise
 r = s + n # 0 dB SNR
 
 # Perform fft, fftshift, convert to dB
 X = fft(r)
 X_shifted = np.roll(X, N//2) # equivalent to np.fft.fftshift
 X_mag = 10*np.log10(np.abs(X_shifted)**2)
 
 # Plot results
 f = np.linspace(sample_rate/-2, sample_rate/2, N)/1e6 # plt in MHz
 plt.plot(f, X_mag)
 plt.plot(f[np.argmax(X_mag)], np.max(X_mag), 'rx') # show max
 plt.grid()
 plt.xlabel('Frequency [MHz]')
 plt.ylabel('Magnitude [dB]')
 plt.show()


.. image:: ../_images/fft_in_python.svg
   :align: center
   :target: ../_images/fft_in_python.svg
   :alt: 파이썬으로 구현한 fft의 예시

JavaScript 및/또는 WebAssembly 기반 구현에 관심이 있다면, 웹이나 NodeJS 애플리케이션에서 FFT를 수행할 수 있는 `WebFFT <https://github.com/IQEngine/WebFFT>`_ 라이브러리를 확인해 보세요. 내부적으로 여러 가지 구현이 포함되어 있으며, 각 구현의 성능을 비교할 수 있는 `벤치마킹 도구 <https://webfft.com>`_ 도 제공됩니다.

