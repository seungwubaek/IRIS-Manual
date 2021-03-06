anomalies
====================================================================================================

개요
----------------------------------------------------------------------------------------------------

주어진 input 데이터에서 일반적인 범위를 벗어난 비정상적인 값을 찾아내는 기능입니다.

설명
----------------------------------------------------------------------------------------------------

| Input 으로 받은 Data 를 대상으로 anomaly 한 값을 찾습니다.
| 옵션으로 입력한 이상탐지 알고리즘(basic / robust) 을 적용하여 anomal 여부를 알 수 있는 컬럼을 추가하여 Dataframe 형식의 data를 결과로 반환합니다.
| 옵션으로 명령어 실행 결과에서 anomalies 로  판명된 값만 따로 뽑아내는 것도 가능합니다.

Parameters
----------------------------------------------------------------------------------------------------

.. code-block:: bash

   * | anomalies index target
                 (by=<column>)?
                 (alg=(basic|robust))?
                 (bound=<0 초과 실수>)?
                 (direct=(below|above|both))?
                 (alert_window=last_<숫자><suffix>)?
                 (seasonality=(additive|multiplicative))?
                 (period=<정수>)?
                 (index_type=(timestamp|date))?

.. list-table::
   :header-rows: 1
   :widths: 15 55 20 10 10

   * - 이름
     - 설명
     - 적용알고리즘
     - 기본값
     - 필수/옵션
   * - index
     - 시계열 데이터에서 시간 필드명 입니다.
     - basic, robust 공통
     - 
     - 필수
   * - target
     - anomaly 탐지할 대상 데이터 필드명 입니다.
     - basic, robust 공통
     - 
     - 필수
   * - by
     - 그룹으로 각각의 이상탐지를 시행 |br| 예 : by=fieldA
     - basic, robust 공통
     - 
     - 옵션
   * - alg
     - basic/robust 알고리즘 선택 가능 |br| basic: 기본통계 |br| robust: STL decomposition |br| 예 : alg=robust
     - 
     - basic
     - 옵션
   * - bound
     - 임계값 범위의 scale을 지정 |br| 위의 수식에 z값의 배수값으로 bound가 커지면 upper/lower limit 의 범위가 늘어납니다.
     - basic, robust 공통
     - 2
     - 옵션
   * - direct
     - anomaly 한 값을 판정할 때 이상치의 방향을 below/above/both 로 선택 |br| below: lower 보다 낮은 수치 |br| above: upper 보다 높은 수치 |br| both: below and above
     - basic, robust 공통
     - both
     - 옵션
   * - alert_window
     - 최근 시간 범위 내 이상치 값을 탐지 |br| 예 : last_60s 이면 최근 60초 이내 데이터 중 이상치를 탐지 |br| 예 : last_1m 이면 최근 1분 이내 데이터 중 이상치를 탐지 br| 예 : last_1h 이면 최근 1시간 이내 데이터 중 이상치를 탐지
     - basic, robust 공통
     -
     - 옵션
   * - seasonality
     - additive/multiplicative 모델 선택 가능, 약자 사용 가능 |br| additive(a, add): ``trend + seasonality + erro``, 데이터의 진폭이 일정할 경우 사용(실수) |br| multiplicative(m, multi): ``trend * seasonality * erro``, 데이터의 폭이 점점 증가하거나 감소할 때 사용(**0제외**)
     - alg=robust 일 때 사용
     - additive
     - 옵션
   * - period
     - robust 알고리즘을 사용 할 때 사용되는 옵션으로, timeseries 의 기간 옵션
     - alg=robust 일 때 사용
     - 6
     - 옵션
   * - index_type
     - index 로 지정되는 시계열 데이터가 시간데이터가 포함된 것인지(timestamp), day 단위 데이터인지 (date) 구분을 하여 입력
     - basic, robust 공통
     - timestamp
     - 옵션


- basic : 단순 통계적 방법을 사용하였습니다.

    - 1.959964는 신뢰구간 95% z상수 값입니다. z상수 값으로 upper limit와 lower limit 를 구하여 이상치를 판단합니다.

- robust : Seasonal_Decomposition을 사용한 알고리즘입니다. 계절성, 추세, 잔차 값을 구별하여 잔차 값으로 임계값을 구하여 이상치를 판단합니다.


**검색어 사용예시**

.. code-block:: bash

  #### alg=basic ####

  # anomalies INDEX TARGET 
   ... | anomalies CTIME PM2_5 
   ... | anomalies CTIME PM2_5 alg=basic

  # anomalies INDEX TARGET by=그룹컬럼 bound=숫자 direct=[below|above|both]
   ... | anomalies CTIME PM2_5 by=station bound=3 direct=below
   ... | anomalies CTIME PM2_5 by=station bound=3 direct=above

  # anomalies INDEX TARGET by=그룹컬럼 bound=숫자 direct=[below|above|both] alert_window=last_숫자[s|m|h]
   ... | anomalies CTIME PM2_5 by=station bound=3 direct=below alert_window=last_1h
   ... | anomalies CTIME PM2_5 by=station bound=3 direct=above alert_window=last_1m
   ... | anomalies CTIME PM2_5 by=station bound=3 direct=both alert_window=last_10m

  # anomalies INDEX TARGET by=그룹컬럼 bound=숫자 direct=[below|above|both] index_type=[timestamp|date]
   ... | anomalies CTIME PM2_5 by=station bound=3 direct=both index_type=timestamp

   ... | anomalies YYYYMMDD PM2_5 by=station bound=3 direct=both index_type=date

  
  
  #### alg=robust ####

  # anomalies INDEX TARGET alg=robust
   ... | anomalies CTIME PM2_5 alg=robust

  # anomalies INDEX TARGET by=그룹컬럼 alg=robust period=정수 bound=정수 direct=[below|above|both]
   ... | anomalies CTIME PM2_5 by=station alg=robust period=12 bound=3 direct=below
   ... | anomalies CTIME PM2_5 by=station alg=robust period=6 bound=3 direct=above

  # anomalies INDEX TARGET by=그룹컬럼 alg=robust bound=숫자 direct=[below|above|both] alert_window=last_숫자[s|m|h]
   ... | anomalies CTIME PM2_5 by=station alg=robust bound=3 direct=below alert_window=last_1h
   ... | anomalies CTIME PM2_5 by=station alg=robust bound=3 direct=above alert_window=last_1m
   ... | anomalies CTIME PM2_5 by=station alg=robust bound=3 direct=both alert_window=last_10m

  # anomalies INDEX TARGET by=그룹컬럼 alg=robust bound=숫자 direct=[below|above|both] index_type=[timestamp|date]
   ... | anomalies CTIME PM2_5 by=station alg=robust bound=3 direct=both index_type=timestamp

   ... | anomalies YYYYMMDD PM2_5 by=station alg=robust bound=3 direct=both index_type=date





Examples
----------------------------------------------------------------------------------------------------

- 예제 데이터

.. list-table::
   :header-rows: 1

   * - CTIME
     - station
     - PM2_5
   * - 20170101180000
     - Changping
     - 443.0
   * - 20170102180000
     - Changping
     - 199.0
   * - 20170103180000
     - Changping
     - 199.0
   * - 20170104180000
     - Changping
     - 262.0
   * - 20170105180000
     - Changping
     - 258.0
   * - 20170106180000
     - Changping
     - 181.0
   * - 20170107180000
     - Changping
     - 104.0
   * - 20170108180000
     - Changping
     - 24.0
   * - 20170109180000
     - Changping
     - 44.0
   * - 20170110180000
     - Changping
     - 14.0
   * - 20170111180000
     - Changping
     - 67.0
   * - 20170112180000
     - Changping
     - 55.0
   * - 20170113180000
     - Changping
     - 12.0
   * - 20170114180000
     - Changping
     - 18.0
   * - 20170115180000
     - Changping
     - 94.0
   * - 20170116180000
     - Changping
     - 118.0
   * - 20170117180000
     - Changping
     - 146.0
   * - 20170118180000
     - Changping
     - 31.0
   * - 20170119180000
     - Changping
     - 12.0
   * - 20170120180000
     - Changping
     - 32.0
   * - 20170121180000
     - Changping
     - 14.0
   * - 20170122180000
     - Changping
     - 26.0
   * - 20170123180000
     - Changping
     - 3.0
   * - 20170124180000
     - Changping
     - 76.0
   * - 20170125180000
     - Changping
     - 160.0
   * - 20170126180000
     - Changping
     - 9.0
   * - 20170127180000
     - Changping
     - 70.0
   * - 20170128180000
     - Changping
     - 218.0
   * - 20170129180000
     - Changping
     - 8.0
   * - 20170130180000
     - Changping
     - 52.0
   * - 20170131180000
     - Changping
     - 23.0

- 예제1) basic 알고리즘을 사용 하는 예

    - basic 알고리즘은 디폴트 알고리즘으로 alg=basic 을 생략할 수 있습니다.

.. code-block:: bash

   ... | anomalies CTIME PM2_5
   ... | anomalies CTIME PM2_5 alg=basic

.. list-table::
   :header-rows: 1

   * - CTIME
     - station
     - PM2_5
     - upper
     - lower
     - anomaly
   * - 2017-01-01 18:00:00
     - Changping
     - 443.0
     - 569.87
     - 316.13
     - False
   * - 2017-01-02 18:00:00
     - Changping
     - 199.0
     - 447.87
     - 194.13
     - False
   * - 2017-01-03 18:00:00
     - Changping
     - 199.0
     - 407.2
     - 153.46
     - False
   * - 2017-01-04 18:00:00
     - Changping
     - 262.0
     - 402.62
     - 148.88
     - False
   * - 2017-01-05 18:00:00
     - Changping
     - 258.0
     - 399.07
     - 145.33
     - False
   * - 2017-01-06 18:00:00
     - Changping
     - 181.0
     - 383.87
     - 130.13
     - False
   * - 2017-01-07 18:00:00
     - Changping
     - 104.0
     - 362.01
     - 108.27
     - True
   * - 2017-01-08 18:00:00
     - Changping
     - 24.0
     - 335.62
     - 81.88
     - True
   * - 2017-01-09 18:00:00
     - Changping
     - 44.0
     - 317.32
     - 63.57
     - True
   * - 2017-01-10 18:00:00
     - Changping
     - 14.0
     - 299.67
     - 45.93
     - True
   * - 2017-01-11 18:00:00
     - Changping
     - 67.0
     - 262.07
     - 8.33
     - False
   * - 2017-01-12 18:00:00
     - Changping
     - 55.0
     - 247.67
     - -6.07
     - False
   * - 2017-01-13 18:00:00
     - Changping
     - 12.0
     - 228.97
     - -24.77
     - False
   * - 2017-01-14 18:00:00
     - Changping
     - 18.0
     - 204.57
     - -49.17
     - False
   * - 2017-01-15 18:00:00
     - Changping
     - 94.0
     - 188.17
     - -65.57
     - False
   * - 2017-01-16 18:00:00
     - Changping
     - 118.0
     - 181.87
     - -71.87
     - False
   * - 2017-01-17 18:00:00
     - Changping
     - 146.0
     - 186.07
     - -67.67
     - False
   * - 2017-01-18 18:00:00
     - Changping
     - 31.0
     - 186.77
     - -66.97
     - False
   * - 2017-01-19 18:00:00
     - Changping
     - 12.0
     - 183.57
     - -70.17
     - False
   * - 2017-01-20 18:00:00
     - Changping
     - 32.0
     - 185.37
     - -68.37
     - False
   * - 2017-01-21 18:00:00
     - Changping
     - 14.0
     - 180.07
     - -73.67
     - False
   * - 2017-01-22 18:00:00
     - Changping
     - 26.0
     - 177.17
     - -76.57
     - False
   * - 2017-01-23 18:00:00
     - Changping
     - 3.0
     - 176.27
     - -77.47
     - False
   * - 2017-01-24 18:00:00
     - Changping
     - 76.0
     - 182.07
     - -71.67
     - False
   * - 2017-01-25 18:00:00
     - Changping
     - 160.0
     - 188.67
     - -65.07
     - False
   * - 2017-01-26 18:00:00
     - Changping
     - 9.0
     - 177.77
     - -75.97
     - False
   * - 2017-01-27 18:00:00
     - Changping
     - 70.0
     - 170.17
     - -83.57
     - False
   * - 2017-01-28 18:00:00
     - Changping
     - 218.0
     - 188.87
     - -64.87
     - True
   * - 2017-01-29 18:00:00
     - Changping
     - 8.0
     - 188.47
     - -65.27
     - False
   * - 2017-01-30 18:00:00
     - Changping
     - 52.0
     - 190.47
     - -63.27
     - False
   * - 2017-01-31 18:00:00
     - Changping
     - 23.0
     - 191.37
     - -62.37
     - False

- 예제2) robust 알고리즘을 사용 하는 예 : 
    - seasonality = additive 는 target 필드의 값에 `` null``  이 있으면 안됩니다.
    - seasonality = multiplicative  는 target 필드의 값에 `` 0 `` 이 있으면 안됩니다.
    - seasonality = additive 를 빼고 실행할 수 있습니다.( default 로 seasonality = additive )
    

.. code-block:: bash

   ... | where PM2_5 is not null | anomalies CTIME PM2_5 alg=robust 

.. list-table::
   :header-rows: 1

   * - CTIME
     - station
     - PM2_5
     - residuals
     - upper
     - lower
     - anomaly
   * - 2017-01-01 18:00:00
     - Changping
     - 443.0
     - None
     - None
     - None
     - False
   * - 2017-01-02 18:00:00
     - Changping
     - 199.0
     - -62.14444444444443
     - -14.203839764553216
     - -110.08504912433565
     - False
   * - 2017-01-03 18:00:00
     - Changping
     - 199.0
     - -21.01111111111111
     - 6.362826902113447
     - -89.51838245766899
     - False
   * - 2017-01-04 18:00:00
     - Changping
     - 262.0
     - 3.1555555555555657
     - 21.27393801322456
     - -74.60727134655787
     - False
   * - 2017-01-05 18:00:00
     - Changping
     - 258.0
     - 43.522222222222254
     - 38.82116023544678
     - -57.06004912433565
     - True
   * - 2017-01-06 18:00:00
     - Changping
     - 181.0
     - -0.011111111111082206
     - 40.642826902113455
     - -55.238382457668976
     - False
   * - 2017-01-07 18:00:00
     - Changping
     - 104.0
     - -18.177777777777777
     - 38.82949356878012
     - -57.05171579100231
     - False
   * - 2017-01-08 18:00:00
     - Changping
     - 24.0
     - -14.144444444444442
     - 38.1104459497325
     - -57.77076341004993
     - False
   * - 2017-01-09 18:00:00
     - Changping
     - 44.0
     - 16.655555555555562
     - 41.421160235446784
     - -54.46004912433565
     - False
   * - 2017-01-10 18:00:00
     - Changping
     - 14.0
     - -46.84444444444444
     - 36.94060467989123
     - -58.9406046798912
     - False
   * - 2017-01-11 18:00:00
     - Changping
     - 67.0
     - 40.855555555555554
     - 42.12616023544678
     - -53.75504912433565
     - False
   * - 2017-01-12 18:00:00
     - Changping
     - 55.0
     - 10.322222222222225
     - 49.372826902113445
     - -46.508382457668986
     - False
   * - 2017-01-13 18:00:00
     - Changping
     - 12.0
     - -35.511111111111106
     - 47.92282690211345
     - -47.95838245766898
     - False
   * - 2017-01-14 18:00:00
     - Changping
     - 18.0
     - -4.144444444444442
     - 47.192826902113445
     - -48.688382457668986
     - False
   * - 2017-01-15 18:00:00
     - Changping
     - 94.0
     - 17.322222222222234
     - 44.57282690211345
     - -51.30838245766898
     - False
   * - 2017-01-16 18:00:00
     - Changping
     - 118.0
     - -20.51111111111109
     - 42.522826902113444
     - -53.35838245766899
     - False
   * - 2017-01-17 18:00:00
     - Changping
     - 146.0
     - 66.85555555555555
     - 51.02616023544678
     - -44.85504912433565
     - True
   * - 2017-01-18 18:00:00
     - Changping
     - 31.0
     - -32.01111111111111
     - 49.23949356878011
     - -46.64171579100232
     - False
   * - 2017-01-19 18:00:00
     - Changping
     - 12.0
     - -32.17777777777778
     - 44.35616023544678
     - -51.52504912433565
     - False
   * - 2017-01-20 18:00:00
     - Changping
     - 32.0
     - 31.855555555555554
     - 52.22616023544678
     - -43.655049124335655
     - False
   * - 2017-01-21 18:00:00
     - Changping
     - 14.0
     - -10.011111111111111
     - 47.13949356878011
     - -48.74171579100232
     - False
   * - 2017-01-22 18:00:00
     - Changping
     - 26.0
     - -7.511111111111109
     - 45.35616023544678
     - -50.52504912433565
     - False
   * - 2017-01-23 18:00:00
     - Changping
     - 3.0
     - -12.811111111111114
     - 47.626160235446775
     - -48.255049124335656
     - False
   * - 2017-01-24 18:00:00
     - Changping
     - 76.0
     - -3.677777777777768
     - 47.67282690211344
     - -48.20838245766899
     - False
   * - 2017-01-25 18:00:00
     - Changping
     - 160.0
     - 59.155555555555566
     - 51.85616023544678
     - -44.02504912433565
     - True
   * - 2017-01-26 18:00:00
     - Changping
     - 9.0
     - -51.477777777777774
     - 48.75949356878011
     - -47.12171579100232
     - True
   * - 2017-01-27 18:00:00
     - Changping
     - 70.0
     - -29.011111111111095
     - 39.17282690211344
     - -56.70838245766899
     - False
   * - 2017-01-28 18:00:00
     - Changping
     - 218.0
     - 100.15555555555557
     - 52.38949356878011
     - -43.49171579100232
     - True
   * - 2017-01-29 18:00:00
     - Changping
     - 8.0
     - -65.47777777777777
     - 49.05949356878011
     - -46.82171579100232
     - True
   * - 2017-01-30 18:00:00
     - Changping
     - 52.0
     - 24.322222222222226
     - 48.30616023544678
     - -47.57504912433565
     - False
   * - 2017-01-31 18:00:00
     - Changping
     - 23.0
     - -28.84444444444444
     - 46.42282690211345
     - -49.45838245766898
     - False

- 예제3) alert_window 옵션으로 설정 기간 에만 이상치 탐지

.. code-block:: bash

   ... | where PM2_5 is not null | anomalies CTIME PM2_5 alg=robust alert_window=last_72h

.. list-table::
   :header-rows: 1

   * - CTIME
     - station
     - PM2_5
     - residuals
     - upper
     - lower
     - anomaly
   * - 2017-01-01 18:00:00
     - Changping
     - 443.0
     - None
     - None
     - None
     - False
   * - 2017-01-02 18:00:00
     - Changping
     - 199.0
     - None
     - None
     - None
     - False
   * - 2017-01-03 18:00:00
     - Changping
     - 199.0
     - None
     - None
     - None
     - False
   * - 2017-01-04 18:00:00
     - Changping
     - 262.0
     - None
     - None
     - None
     - False
   * - 2017-01-05 18:00:00
     - Changping
     - 258.0
     - None
     - None
     - None
     - False
   * - 2017-01-06 18:00:00
     - Changping
     - 181.0
     - None
     - None
     - None
     - False
   * - 2017-01-07 18:00:00
     - Changping
     - 104.0
     - None
     - None
     - None
     - False
   * - 2017-01-08 18:00:00
     - Changping
     - 24.0
     - None
     - None
     - None
     - False
   * - 2017-01-09 18:00:00
     - Changping
     - 44.0
     - None
     - None
     - None
     - False
   * - 2017-01-10 18:00:00
     - Changping
     - 14.0
     - None
     - None
     - None
     - False
   * - 2017-01-11 18:00:00
     - Changping
     - 67.0
     - None
     - None
     - None
     - False
   * - 2017-01-12 18:00:00
     - Changping
     - 55.0
     - None
     - None
     - None
     - False
   * - 2017-01-13 18:00:00
     - Changping
     - 12.0
     - None
     - None
     - None
     - False
   * - 2017-01-14 18:00:00
     - Changping
     - 18.0
     - None
     - None
     - None
     - False
   * - 2017-01-15 18:00:00
     - Changping
     - 94.0
     - None
     - None
     - None
     - False
   * - 2017-01-16 18:00:00
     - Changping
     - 118.0
     - None
     - None
     - None
     - False
   * - 2017-01-17 18:00:00
     - Changping
     - 146.0
     - None
     - None
     - None
     - False
   * - 2017-01-18 18:00:00
     - Changping
     - 31.0
     - None
     - None
     - None
     - False
   * - 2017-01-19 18:00:00
     - Changping
     - 12.0
     - None
     - None
     - None
     - False
   * - 2017-01-20 18:00:00
     - Changping
     - 32.0
     - None
     - None
     - None
     - False
   * - 2017-01-21 18:00:00
     - Changping
     - 14.0
     - None
     - None
     - None
     - False
   * - 2017-01-22 18:00:00
     - Changping
     - 26.0
     - None
     - None
     - None
     - False
   * - 2017-01-23 18:00:00
     - Changping
     - 3.0
     - None
     - None
     - None
     - False
   * - 2017-01-24 18:00:00
     - Changping
     - 76.0
     - None
     - None
     - None
     - False
   * - 2017-01-25 18:00:00
     - Changping
     - 160.0
     - None
     - None
     - None
     - False
   * - 2017-01-26 18:00:00
     - Changping
     - 9.0
     - None
     - None
     - None
     - False
   * - 2017-01-27 18:00:00
     - Changping
     - 70.0
     - None
     - None
     - None
     - False
   * - 2017-01-28 18:00:00
     - Changping
     - 218.0
     - 100.15555555555557
     - 52.38949356878011
     - -43.49171579100232
     - True
   * - 2017-01-29 18:00:00
     - Changping
     - 8.0
     - -65.47777777777777
     - 49.05949356878011
     - -46.82171579100232
     - True
   * - 2017-01-30 18:00:00
     - Changping
     - 52.0
     - 24.322222222222226
     - 48.30616023544678
     - -47.57504912433565
     - False
   * - 2017-01-31 18:00:00
     - Changping
     - 23.0
     - -28.84444444444444
     - 46.42282690211345
     - -49.45838245766898
     - False


.. |br| raw:: html

  <br/>
