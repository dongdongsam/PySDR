.. _intro-chapter:

#############
머릿말
#############

***************************
강의 목적과 주요 대상
***************************

일단 첫번째로 가장 중요한건, 아래의 두 단어입니다.

**Software-Defined Radio (SDR):**
    *개념* 적으로는 라디오나 RF 기기에 의해서 처리되었던 RF 신호를 소프트웨어를 이용해 처리하는 것을 말합니다. 이때 범용 컴퓨터(CPU) 뿐만 아니라 FPGA(커스텀 칩), 심지어는 GPU를 이용할 수 도 있습니다. 더불어 실시간 처리 뿐 만 아니라, 녹화된 신호에 대한 RF 처리를 할 수 있습니다. 유사한 단어로는 "소프트웨어 라디오", 그리고 RF 시그널 디지털 처리(DSP)가 있습니다.

    *실체* 적으로는 (예 : "SDR 기기") 대게 안테나를 꽂아서 RF 신호를 받아 올 수 있는 장치를 말합 니다. 그리고 이 장치가 받은 RF 신호들은 디지털화 되어서 컴퓨터에 (USB, PCI express 등) 저장됩니다. 또한 많은 SDR 기기들은 컴퓨터를 통해서 RF 시그널을 보낼수 있는 기능(TX)을 지원하고, 임베디드 성격의 SDR 기기들은 온보드 컴퓨터를 탑재하기도 합니다.

**Digital Signal Processing (DSP):**
    신호의 디지털 처리를 의미하고, 여기서는 RF 신호가 대상입니다.

이 교과서는 DSP, SDR 그리고 RF 신호 처리에 있어서 직접 해보는 지침적 활동을 제공합니다. 따라서 아래와 같은 분들에게 권장 드립니다 :

#. 뭔가 쩌는걸 하기 위해서 *SDR* 을 이용하는데 관심 있으신 분들
#. 파이썬 잘 치시는 분들
#. 상대적으로 DSP, SDR, 무선통신에 입문하신지 얼마 안되신 분들
#. 방정식에 대해서 시각적 애니메이션으로 이해하는 걸 좋아하시는 분들
#. 개념을 이해한 후에 방정식을 잘 이해하시는 분들 *(선개념 후학습)*
#. 1000페이지 짜리 수업이 아니라 간결한 설명을 원하시는 분들.

그러니까 예를 들어서, 한 컴공 학생이 졸업 후 통신업계에 관심이 있을 수 있습니다 이때 이 자료는 프로그래밍 경험이 있고 SDR에 대해서 배우고 싶을 때 유용할 수 있습니다.  고로, 대게의 DSP 수업이 복잡한 수식을 바탕으로 하는 반면에 이 수업은 그렇지 않습니다.  방정식에 묻혀 사는거 보다는, 간결한 이미지와 애니메이션이 이해에는 더 나을 수 있고, 이는 아래와 같은 FFT 애니메이션이 증명합니다. 본 저자는 실체적인 연습과 개념을 *이해한 다음에* 방정식이 잘 이해가 된다고 생각합니다. 그리고 이건 아마존에 카피본이 없는 이유이기도 한데, 엄청난 양의 애니메이션이 들어있기 때문이죠.

.. image:: ../_images/fft_logo_wide.gif
   :scale: 70 %   
   :align: center
   :alt: FFT를 이용해 만든 PySDR의 로고
   
이 교과서는 저자가 현명하게 DSP와 SDR을 이해하기 위해서, 쉽고 간결한 이해를 돕게 하는 것을 목적으로 합니다.  따라서 모든 SDR/DSP에 관한 토픽을 다루는 것은 아니며, 이에 대해서는 - `Analog Device's SDR textbook
<https://www.analog.com/en/education/education-library/software-defined-radio-for-engineers.html>`_ 와 `dspguide.com <http://www.dspguide.com/>`_ 같은 훌륭한 교과서들이 이미 있습니다. 삼각함수 항등식이나 섀넌 한계(노이즈가 있는 채널에서 오류 없이 데이터를 전송할 수 있는 최대의 정도) 같은 건 언제든지 구글을 통해 찾아볼 수 있습니다. 이 교재는 DSP와 SDR의 세계로 들어가는 입문서로 생각하면 됩니다. 전통적인 강의나 교재에 비해 더 가볍고, 시간이나 비용 부담도 적습니다.

기초적인 DSP 이론을 다루기 위해, 전기공학에서 일반적으로 한 학기 동안 배우는 "신호 및 시스템(Signals and Systems)" 과정을 몇 개의 챕터로 압축해 담았습니다. DSP의 기본 개념을 익힌 후에는 본격적으로 SDR에 관한 주제로 들어가며, 이후에도 DSP와 무선 통신 관련 개념은 교재 전반에 걸쳐 계속 등장합니다.

코드 예제는 **파이썬(Python)** 으로 제공됩니다. 이 예제들은 NumPy를 활용하는데, 이는 배열과 고수준 수학 연산을 위한 파이썬의 표준 라이브러리입니다. 또한, Matplotlib라는 시각화 도구를 사용하여 신호, 배열, 복소수 등을 쉽게 그래프로 표현할 수 있습니다.

파이썬은 일반적으로 C++보다 느리다고 여겨지지만(Cpython의 경우 특히 그렇습니다.), 실제로 대부분의 수학 함수는 C/C++로 구현되어 있으며 고도로 최적화되어 있습니다. 마찬가지로, 우리가 사용할 SDR API도 C/C++로 작성된 함수와 클래스들을 파이썬에서 쓸 수 있도록 바인딩한 것입니다.

파이썬 경험이 많지 않더라도, MATLAB, Ruby, Perl 등에 익숙한 사람이라면 파이썬 문법만 익히면 무리 없이 따라올 수 있습니다.


***************
기여하기
***************

만약 여러분이 PySDR에서 느낀 점이 있다면, 이 주제에 관심이 있을 동료/학생/학습자들을 위해서 공유해 주세요. 또한 기부도 가능하며, 매 페이지 왼쪽에 기부자 명단으로 수록됩니다. `PySDR Patreon <https://www.patreon.com/PySDR>`_ 을 참조해 주세요.

만약 생각이 있어서 marc@pysdr.ogr로 질문/제안/코멘트를 남겨 주신다면, 축하합니다! 이 텍스트북에 기여하게 되는 셈입니다. 또한 텍스트북의 내용 자체를 `textbook's GitHub page <https://github.com/777arc/PySDR/tree/master/content>`_ 를 통해서 직접 수정 가능합니다. (새 Pull Request를 열어서 해주세요.)  개선이나 향상을 위해서는 얼마든지 PR을 주셔도 됩니다.  피드백을 주신 분들은 아래에 이름이 기록됩니다.  만약 Git에 기록하긴 그런데 뭔가 수정해야 할 것 같으면, marc@pysdr.org로 언제든지 메일 주세요.

*****************
감사의 말
*****************

피드백을 주시고 읽어주신 분들에게 감사드립니다. 특히 아래 분들이요 :

- `Barry Duggan <http://github.com/duggabe>`_
- Matthew Hannon
- James Hayek
- Deidre Stuffer
- Tarik Benaddi for `PySDR to French <https://pysdr.org/fr/index-fr.html>`_
- `Daniel Versluis <https://versd.bitbucket.io/content/about.html>`_ for `translating PySDR to Dutch <https://pysdr.org/nl/index-nl.html>`_
- `mrbloom <https://github.com/mrbloom>`_ for `translating PySDR to Ukrainian <https://pysdr.org/ukraine/index-ukraine.html>`_
- `Yimin Zhao <https://github.com/doctormin>`_ for `translating PySDR to Simplified Chinese <https://pysdr.org/zh/index-zh.html>`_
- `Eduardo Chancay <https://github.com/edulchan>`_ for `translating PySDR to Spanish <https://pysdr.org/es/index-es.html>`_

`PySDR Patreon <https://www.patreon.com/PySDR>`_ 후원자 분들도요!
