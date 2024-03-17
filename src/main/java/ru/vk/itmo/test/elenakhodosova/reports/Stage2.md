# Этап 2. Асинхронный сервер

### PUT

Сравним производительность с реализацией 1 этапа, где точка разладки была примерно на 38000 RPS

Параметры:
 - Размер очереди 256
 - Используемая очередь: ArrayBlockingQueue
 - Максимальное количество потоков 128
```
wrk2 -d 30 -t 1 -c 1 -R 31000 -L -s put.lua http://127.0.0.1:8080
Running 30s test @ http://127.0.0.1:8080
  1 threads and 1 connections
  Thread calibration: mean lat.: 97.036ms, rate sampling interval: 296ms
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   284.43ms   84.82ms 451.07ms   53.55%
    Req/Sec    30.62k   545.86    33.04k    77.61%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%  273.41ms
 75.000%  359.17ms
 90.000%  408.06ms
 99.000%  438.02ms
 99.900%  450.05ms
 99.990%  451.07ms
 99.999%  451.33ms
100.000%  451.33ms

#[Mean    =      284.431, StdDeviation   =       84.818]
#[Max     =      451.072, Total count    =       610764]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  916052 requests in 30.00s, 58.53MB read
Requests/sec:  30535.33
Transfer/sec:      1.95MB
```
Теперь сервер не выдерживает даже 31000 RPS

Посмотрим, что происходит, нагрузив сервис на 60 секунд с 25000 RPS
```
 wrk2 -d 60 -t 1 -c 1 -R 25000 -L -s put.lua http://127.0.0.1:8080
Running 1m test @ http://127.0.0.1:8080
  1 threads and 1 connections
  Thread calibration: mean lat.: 0.855ms, rate sampling interval: 10ms
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   675.78us  628.39us   9.18ms   96.60%
    Req/Sec    26.42k     2.32k   37.89k    72.18%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%  621.00us
 75.000%    0.91ms
 90.000%    1.08ms
 99.000%    4.04ms
 99.900%    6.81ms
 99.990%    8.57ms
 99.999%    9.10ms
100.000%    9.18ms

  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

       0.021     0.000000            1         1.00
       0.150     0.100000       125020         1.11
       0.270     0.200000       250607         1.25
       0.388     0.300000       375704         1.43
       0.505     0.400000       500514         1.67
       0.621     0.500000       625822         2.00
       0.679     0.550000       688410         2.22
       0.737     0.600000       751007         2.50
       0.794     0.650000       813013         2.86
       0.851     0.700000       875373         3.33
       0.909     0.750000       938271         4.00
       0.937     0.775000       968849         4.44
       0.966     0.800000      1000442         5.00
       0.995     0.825000      1031976         5.71
       1.023     0.850000      1062491         6.67
       1.052     0.875000      1093946         8.00
       1.067     0.887500      1110030         8.89
       1.081     0.900000      1125080        10.00
       1.096     0.912500      1141486        11.43
       1.110     0.925000      1156410        13.33
       1.125     0.937500      1172623        16.00
       1.132     0.943750      1180199        17.78
       1.139     0.950000      1187613        20.00
       1.146     0.956250      1195261        22.86
       1.153     0.962500      1203220        26.67
       1.161     0.968750      1211835        32.00
       1.165     0.971875      1215341        35.56
       1.172     0.975000      1218887        40.00
       1.205     0.978125      1222531        45.71
       1.635     0.981250      1226431        53.33
       2.419     0.984375      1230336        64.00
       2.875     0.985938      1232294        71.11
       3.327     0.987500      1234245        80.00
       3.781     0.989062      1236195        91.43
       4.215     0.990625      1238152       106.67
       4.607     0.992188      1240103       128.00
       4.795     0.992969      1241093       142.22
       4.987     0.993750      1242081       160.00
       5.183     0.994531      1243034       182.86
       5.391     0.995313      1244026       213.33
       5.615     0.996094      1244990       256.00
       5.711     0.996484      1245488       284.44
       5.815     0.996875      1245967       320.00
       5.931     0.997266      1246458       365.71
       6.079     0.997656      1246950       426.67
       6.219     0.998047      1247430       512.00
       6.307     0.998242      1247670       568.89
       6.415     0.998437      1247917       640.00
       6.523     0.998633      1248160       731.43
       6.675     0.998828      1248409       853.33
       6.831     0.999023      1248648      1024.00
       6.939     0.999121      1248769      1137.78
       7.027     0.999219      1248890      1280.00
       7.143     0.999316      1249014      1462.86
       7.267     0.999414      1249134      1706.67
       7.455     0.999512      1249256      2048.00
       7.551     0.999561      1249317      2275.56
       7.627     0.999609      1249380      2560.00
       7.731     0.999658      1249438      2925.71
       7.831     0.999707      1249502      3413.33
       7.927     0.999756      1249560      4096.00
       8.035     0.999780      1249592      4551.11
       8.139     0.999805      1249621      5120.00
       8.287     0.999829      1249653      5851.43
       8.431     0.999854      1249682      6826.67
       8.503     0.999878      1249716      8192.00
       8.535     0.999890      1249730      9102.22
       8.575     0.999902      1249743     10240.00
       8.615     0.999915      1249759     11702.86
       8.647     0.999927      1249776     13653.33
       8.711     0.999939      1249790     16384.00
       8.735     0.999945      1249799     18204.44
       8.751     0.999951      1249804     20480.00
       8.775     0.999957      1249813     23405.71
       8.839     0.999963      1249820     27306.67
       8.911     0.999969      1249827     32768.00
       8.935     0.999973      1249831     36408.89
       8.975     0.999976      1249835     40960.00
       9.007     0.999979      1249840     46811.43
       9.023     0.999982      1249844     54613.33
       9.031     0.999985      1249846     65536.00
       9.047     0.999986      1249848     72817.78
       9.063     0.999988      1249850     81920.00
       9.079     0.999989      1249852     93622.86
       9.103     0.999991      1249854    109226.67
       9.119     0.999992      1249856    131072.00
       9.127     0.999993      1249857    145635.56
       9.143     0.999994      1249858    163840.00
       9.151     0.999995      1249859    187245.71
       9.159     0.999995      1249860    218453.33
       9.167     0.999996      1249861    262144.00
       9.167     0.999997      1249861    291271.11
       9.175     0.999997      1249863    327680.00
       9.175     0.999997      1249863    374491.43
       9.175     0.999998      1249863    436906.67
       9.175     0.999998      1249863    524288.00
       9.175     0.999998      1249863    582542.22
       9.183     0.999998      1249865    655360.00
       9.183     1.000000      1249865          inf
#[Mean    =        0.676, StdDeviation   =        0.628]
#[Max     =        9.176, Total count    =      1249865]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  1499965 requests in 1.00m, 95.84MB read
Requests/sec:  24999.34
Transfer/sec:      1.60MB
```
[CPU flame graph](data/stage2/profile-put-25000.html)

Как видно из графа, суммарно около 30% времени уходит на работу с потоками


Сравним работу с разным количеством потоков при 64 connections и найдем точки разладки, начнем с 1, точка разладки примерно 110000 RPS:
```
 wrk2 -d 60 -t 1 -c 64 -R 110000 -L -s put.lua http://127.0.0.1:8080
Running 1m test @ http://127.0.0.1:8080
  1 threads and 64 connections
  Thread calibration: mean lat.: 30.585ms, rate sampling interval: 134ms
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   386.68ms  214.56ms   1.05s    62.94%
    Req/Sec   109.18k     2.37k  113.67k    65.14%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%  359.17ms
 75.000%  536.06ms
 90.000%  694.27ms
 99.000%  902.66ms
 99.900%  995.33ms
 99.990%    1.05s 
 99.999%    1.05s 
100.000%    1.05s 
----------------------------------------------------------
  6511402 requests in 1.00m, 416.05MB read
Requests/sec: 108523.60
Transfer/sec:      6.93MB

```
Для 2 потоков - примерно 135000 RPS
```
wrk2 -d 60 -t 2 -c 64 -R 135000 -L -s put.lua http://127.0.0.1:8080
Running 1m test @ http://127.0.0.1:8080
  2 threads and 64 connections
  Thread calibration: mean lat.: 1.666ms, rate sampling interval: 10ms
  Thread calibration: mean lat.: 1.782ms, rate sampling interval: 10ms
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     3.39ms    4.89ms  66.94ms   89.39%
    Req/Sec    71.22k    10.97k  103.67k    73.89%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%    1.54ms
 75.000%    4.14ms
 90.000%    8.57ms
 99.000%   22.51ms
 99.900%   49.63ms
 99.990%   60.29ms
 99.999%   65.15ms
100.000%   67.01ms
----------------------------------------------------------
  8026860 requests in 1.00m, 512.89MB read
Requests/sec: 133781.83
Transfer/sec:      8.55MB

```
Для 16 потоков - примерно 143000 RPS
```
wrk2 -d 30 -t 16 -c 64 -R 143000 -L -s put.lua http://127.0.0.1:8080

  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     2.69ms    5.56ms 117.25ms   94.04%
    Req/Sec     9.22k     1.13k   15.69k    76.63%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%    1.36ms
 75.000%    2.40ms
 90.000%    5.77ms
 99.000%   18.69ms
 99.900%   80.32ms
 99.990%  107.78ms
 99.999%  113.98ms
100.000%  117.31ms
----------------------------------------------------------
  4288567 requests in 30.00s, 274.02MB read
Requests/sec: 142967.28
Transfer/sec:      9.14MB

```

```
wrk2 -d 30 -t 64 -c 64 -R 153000 -L -s put.lua http://127.0.0.1:8080
Running 30s test @ http://127.0.0.1:8080
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   218.44ms  336.55ms   2.20s    86.27%
    Req/Sec     2.35k   358.79     8.33k    76.22%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%   51.55ms
 75.000%  304.38ms
 90.000%  690.69ms
 99.000%    1.50s 
 99.900%    2.03s 
 99.990%    2.18s 
 99.999%    2.20s 
100.000%    2.20s 
----------------------------------------------------------
  4472039 requests in 29.99s, 285.75MB read
Requests/sec: 149132.74
Transfer/sec:      9.53MB

```
Получаем, что при 64 потоках сервер справляется лучше всего и точка разладки примерно на 153000 RPS.

Проверим зависимость пропускной способности от размера очереди:
- Для размера очереди 128 максимальная нагрузка 140000 RPS
- Для размеров 256 - 1024 точка разладки не менялась, поэтому будем считать текущий размер 256 вполне оптимальным.

Нагрузим сервер с такими параметрами на 130000 RPS
```
wrk2 -d 60 -t 64 -c 64 -R 130000 -L -s put.lua http://127.0.0.1:8080   
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.80ms    4.27ms  95.68ms   96.92%
    Req/Sec     2.04k    64.25     3.05k    91.09%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%    1.13ms
 75.000%    1.62ms
 90.000%    2.07ms
 99.000%   22.94ms
 99.900%   56.38ms
 99.990%   72.77ms
 99.999%   91.46ms
100.000%   95.74ms

  Detailed Percentile spectrum:
       Value   Percentile   TotalCount 1/(1-Percentile)

       0.021     0.000000            1         1.00
       0.414     0.100000       651860         1.11
       0.650     0.200000      1302508         1.25
       0.857     0.300000      1952001         1.43
       1.004     0.400000      2602553         1.67
       1.134     0.500000      3253347         2.00
       1.203     0.550000      3575768         2.22
       1.287     0.600000      3902730         2.50
       1.385     0.650000      4227444         2.86
       1.496     0.700000      4550772         3.33
       1.620     0.750000      4876371         4.00
       1.682     0.775000      5037833         4.44
       1.743     0.800000      5200738         5.00
       1.804     0.825000      5363587         5.71
       1.868     0.850000      5524503         6.67
       1.944     0.875000      5686525         8.00
       1.994     0.887500      5767889         8.89
       2.065     0.900000      5848988        10.00
       2.193     0.912500      5930274        11.43
       2.459     0.925000      6011659        13.33
       2.891     0.937500      6092914        16.00
       3.191     0.943750      6133429        17.78
       3.593     0.950000      6173985        20.00
       4.147     0.956250      6214639        22.86
       4.911     0.962500      6255282        26.67
       5.979     0.968750      6295774        32.00
       6.703     0.971875      6316122        35.56
       7.611     0.975000      6336411        40.00
       8.831     0.978125      6356717        45.71
      10.591     0.981250      6377024        53.33
      13.423     0.984375      6397343        64.00
      15.583     0.985938      6407484        71.11
      18.015     0.987500      6417654        80.00
      20.927     0.989062      6427810        91.43
      24.271     0.990625      6437927       106.67
      27.871     0.992188      6448108       128.00
      29.679     0.992969      6453165       142.22
      31.711     0.993750      6458248       160.00
      34.079     0.994531      6463376       182.86
      36.831     0.995313      6468384       213.33
      39.871     0.996094      6473484       256.00
      41.791     0.996484      6476011       284.44
      43.647     0.996875      6478548       320.00
      45.471     0.997266      6481079       365.71
      47.391     0.997656      6483621       426.67
      49.663     0.998047      6486170       512.00
      50.879     0.998242      6487431       568.89
      52.063     0.998437      6488711       640.00
      53.343     0.998633      6489972       731.43
      54.943     0.998828      6491243       853.33
      56.607     0.999023      6492510      1024.00
      57.567     0.999121      6493150      1137.78
      58.623     0.999219      6493770      1280.00
      59.775     0.999316      6494412      1462.86
      61.055     0.999414      6495047      1706.67
      62.591     0.999512      6495675      2048.00
      63.295     0.999561      6496009      2275.56
      63.935     0.999609      6496315      2560.00
      64.703     0.999658      6496631      2925.71
      65.727     0.999707      6496947      3413.33
      67.007     0.999756      6497267      4096.00
      67.583     0.999780      6497419      4551.11
      68.287     0.999805      6497577      5120.00
      69.183     0.999829      6497739      5851.43
      70.207     0.999854      6497908      6826.67
      71.359     0.999878      6498053      8192.00
      72.191     0.999890      6498137      9102.22
      72.959     0.999902      6498219     10240.00
      74.111     0.999915      6498294     11702.86
      75.519     0.999927      6498372     13653.33
      76.927     0.999939      6498450     16384.00
      78.463     0.999945      6498492     18204.44
      79.743     0.999951      6498531     20480.00
      81.215     0.999957      6498569     23405.71
      83.583     0.999963      6498610     27306.67
      85.055     0.999969      6498650     32768.00
      85.695     0.999973      6498669     36408.89
      86.783     0.999976      6498688     40960.00
      87.807     0.999979      6498708     46811.43
      88.575     0.999982      6498731     54613.33
      89.407     0.999985      6498748     65536.00
      89.919     0.999986      6498759     72817.78
      90.623     0.999988      6498767     81920.00
      91.391     0.999989      6498779     93622.86
      91.647     0.999991      6498789    109226.67
      92.095     0.999992      6498797    131072.00
      92.287     0.999993      6498802    145635.56
      92.863     0.999994      6498807    163840.00
      93.183     0.999995      6498813    187245.71
      93.631     0.999995      6498817    218453.33
      93.759     0.999996      6498822    262144.00
      93.887     0.999997      6498824    291271.11
      94.015     0.999997      6498828    327680.00
      94.079     0.999997      6498829    374491.43
      94.271     0.999998      6498833    436906.67
      94.335     0.999998      6498834    524288.00
      94.399     0.999998      6498835    582542.22
      94.847     0.999998      6498838    655360.00
      94.847     0.999999      6498838    748982.86
      94.975     0.999999      6498839    873813.33
      95.039     0.999999      6498840   1048576.00
      95.167     0.999999      6498841   1165084.44
      95.295     0.999999      6498843   1310720.00
      95.295     0.999999      6498843   1497965.71
      95.295     0.999999      6498843   1747626.67
      95.295     1.000000      6498843   2097152.00
      95.423     1.000000      6498844   2330168.89
      95.423     1.000000      6498844   2621440.00
      95.423     1.000000      6498844   2995931.43
      95.615     1.000000      6498845   3495253.33
      95.615     1.000000      6498845   4194304.00
      95.615     1.000000      6498845   4660337.78
      95.615     1.000000      6498845   5242880.00
      95.615     1.000000      6498845   5991862.86
      95.743     1.000000      6498846   6990506.67
      95.743     1.000000      6498846          inf
#[Mean    =        1.802, StdDeviation   =        4.274]
#[Max     =       95.680, Total count    =      6498846]
#[Buckets =           27, SubBuckets     =         2048]
----------------------------------------------------------
  7799170 requests in 1.00m, 498.34MB read
Requests/sec: 130008.96
Transfer/sec:      8.31MB
```
[CPU flame graph](data/stage2/profile-put-130000.html)

Около 51% уходит на poll(), около 20% суммарно уходит на работу с потоками (старт,
ожидание блокировок) - из-за чего средняя latency выросла по сравнению с синхронной версией из 1 этапа (с 0,658ms до 1.80ms).
Чтение и запись в сокет занимают примерно по 8% и чуть меньше уходит 3% на саму бизнес-логику.

[Alloc flame graph](data/stage2/profile-put-130000-alloc.html)

Примерно 50% аллокаций занимает handleRequest, из которых около 20% - запись в DAO, остальное - работа one.nio (парсинг request, id, sendResponse).
Вторая половина аллокаций - работа selectorThread из one.nio. Из них примерно 5% сам select и остальное на парсинг запроса.

[Lock flame graph](data/stage2/profile-put-130000-lock.html)

Примерно 60% приходится на ArrayBlockingQueue.take, примерно 5% на sendResponse из one.nio, и почти 35% на selectorThread,
где основная часть локов приходится на ThreadPoolExecutor, метод offer в ArrayBlockingQueue.


Для сравнения посмотрим, как будет вести себя синхронная версия с 64 потоками и соединениями:
```
wrk2 -d 30 -t 64 -c 64 -R 147000 -L -s put.lua http://127.0.0.1:8080  
Running 30s test @ http://127.0.0.1:8080
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   865.01ms  798.58ms   3.75s    63.06%
    Req/Sec     2.33k   463.52     5.98k    78.71%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%  784.90ms
 75.000%    1.35s 
 90.000%    1.90s 
 99.000%    3.30s 
 99.900%    3.64s 
 99.990%    3.74s 
 99.999%    3.75s 
100.000%    3.75s 
```
Cервис перестает выдерживать нагрузку примерно на 147000 RPS, теперь проверим его на стабильной нагрузке в 130000 RPS аналогично с асинхронной версией
```
wrk2 -d 60 -t 64 -c 64 -R 130000 -L -s put.lua http://127.0.0.1:8080
R Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.77ms    6.52ms 198.40ms   98.35%
    Req/Sec     2.04k    74.05     4.12k    96.63%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%    1.08ms
 75.000%    1.50ms
 90.000%    1.87ms
 99.000%   20.58ms
 99.900%  101.38ms
 99.990%  185.60ms
 99.999%  195.33ms
100.000%  198.53ms
```
Получается, что в синхронной версии ниже средняя latency, но максимальное время обработки растет почти до 200ms.


[CPU flame graph](data/stage2/profile-put-130000-sync.html)

[Alloc flame graph](data/stage2/profile-put-130000-sync-alloc.html)

[Lock flame graph](data/stage2/profile-put-130000-sync-lock.html)

По результатам профилирования видно, что время, уходящее на poll() cелектора в асинхронной версии сократилось с на 20%
Примерно 15% аллокаций затрачены на бизнес-логику, соответственно в асинхронной версии на 5% больше
 

Вернемся к асинхронной версии и попробуем уменьшить размер пула потоков до 8
```
wrk2 -d 60 -t 64 -c 64 -R 130000 -L -s put.lua http://127.0.0.1:8080
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.74ms    5.62ms 152.45ms   97.97%
    Req/Sec     2.09k   132.15     4.20k    79.27%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%    1.09ms
 75.000%    1.54ms
 90.000%    1.91ms
 99.000%   18.90ms
 99.900%   98.94ms
 99.990%  135.17ms
 99.999%  149.50ms
100.000%  152.57ms
```
Средняя latency немного уменьшилась, что не является особо показательным, так как +- 0.06ms можно считать погреностью.
Но видна деградация времени выполнения, это может свидетельстовать о том, что задачи накапливаются в очереди.

### GET

Аналогично с PUT запросом сравним производительность с реализацией 1 этапа, где точка разладки была примерно на 35000 RPS
```
lena@Khodosovas-MacBook-Air Downloads %  wrk2 -d 30 -t 1 -c 1 -R 27000 -L -s get.lua http://127.0.0.1:8080 
Running 30s test @ http://127.0.0.1:8080
  1 threads and 1 connections
  Thread calibration: mean lat.: 75.520ms, rate sampling interval: 196ms
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     5.17ms   14.34ms  69.25ms   90.84%
    Req/Sec    27.17k   291.39    28.23k    89.22%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%  707.00us
 75.000%    1.03ms
 90.000%   12.84ms
 99.000%   67.26ms
 99.900%   68.99ms
 99.990%   69.31ms
 99.999%   69.31ms
100.000%   69.31ms
```
Теперь уже при 27000 rps средняя latency выросла до 5.17ms. 

Нагрузим его на 60 секунд с 25000 RPS также с 1 потоком и 1 соединением
```
wrk2 -d 60 -t 1 -c 1 -R 25000 -L -s get.lua http://127.0.0.1:8080 
Running 1m test @ http://127.0.0.1:8080
  1 threads and 1 connections
  Thread calibration: mean lat.: 2.593ms, rate sampling interval: 15ms
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   656.25us  704.62us  15.49ms   99.34%
    Req/Sec    25.90k     1.24k   30.36k    64.49%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%  623.00us
 75.000%    0.91ms
 90.000%    1.08ms
 99.000%    1.22ms
 99.900%   12.96ms
 99.990%   15.19ms
 99.999%   15.46ms
100.000%   15.49ms
```
Cредняя latency почти как и у синхронной реализации, с 99.9 перцентиля есть значительный рост времени выполнения.

Это скорее всего связано с тем, что приходилось обращаться к sstable, на get ушло 26% сэмплов + время на ожидание блокировок в take().

[CPU profile](data/stage2/profile-get-25000-1-1.html)

Для сравнения увеличим количество соединений
и потоков до 64, не меняя остальных параметров:
```
lena@Khodosovas-MacBook-Air Downloads %  wrk2 -d 60 -t 64 -c 64 -R 25000 -L -s get.lua http://127.0.0.1:8080
Running 1m test @ http://127.0.0.1:8080
  64 threads and 64 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     0.93ms  401.13us  10.24ms   70.46%
    Req/Sec   412.03     46.55   777.00     81.01%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%    0.92ms
 75.000%    1.19ms
 90.000%    1.39ms
 99.000%    2.10ms
 99.900%    3.67ms
 99.990%    5.69ms
 99.999%    7.37ms
100.000%   10.25ms
----------------------------------------------------------
  1499887 requests in 1.00m, 165.76MB read
Requests/sec:  25002.41
Transfer/sec:      2.76MB
```
Как мы видим, все запросы выполняются успешно, а средняя latency немного выросла. Думаю, что из-за того, что стало больше времени уходить на блокировки,
так как теперь потоков много, 26% сэмплов на take() против 19% c 1 потоком.

[CPU profile](data/stage2/profile-get-25000-64-64.html)

Для таких параметров сервер перестает справляться с нагрузкой примерно после 155000 RPS
```
lena@Khodosovas-MacBook-Air Downloads %  wrk2 -d 30 -t 64 -c 64 -R 155000 -L -s get.lua http://127.0.0.1:8080
Running 30s test @ http://127.0.0.1:8080
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    14.53ms   34.42ms 357.89ms   91.56%
    Req/Sec     2.50k   539.55     6.00k    77.39%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%    2.03ms
 75.000%    9.14ms
 90.000%   39.97ms
 99.000%  178.30ms
 99.900%  316.93ms
 99.990%  349.95ms
----------------------------------------------------------
  4635309 requests in 29.99s, 512.29MB read
Requests/sec: 154573.75
Transfer/sec:     17.08MB
```
Для 64 соединений снова лучший результат при 64 потоках.
Теперь попробуем изменить размер очереди, начнем с 128 - здесь точка разладки составила 145000 RPS
Аналогично с PUT запросом, для размеров 256 - 1024 точка разладки примерно 155000 RPS.

Нагрузим сервер на 100000 RPS
```
wrk2 -d 60 -t 64 -c 64 -R 90000 -L -s get.lua http://127.0.0.1:8080
 Latency     1.52ms    4.31ms 108.86ms   98.71%
    Req/Sec     1.48k   178.99     4.55k    72.39%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%    1.08ms
 75.000%    1.47ms
 90.000%    1.85ms
 99.000%    9.22ms
 99.900%   71.87ms
 99.990%   95.55ms
 99.999%  105.41ms
100.000%  108.93ms
```

[CPU flame graph](data/stage2/profile-get-90000.html)

[Alloc flame graph](data/stage2/profile-get-90000-alloc.html)

[Lock flame graph](data/stage2/profile-get-90000-lock.html)

Аналогично с PUT запрросом средняя latency выросла по сравнению с синхронной версией из 1 этапа (с 0,631ms до 1.52ms).
Из профля локов видно, что примерно 55% сэмплов приходится на ArrayBlockingQueue.take
Мне кажется, что влияют накладные расходы, связанные с работой потоков, в том числе ожидание блокировок - из профля CPU 21% сэмплов на take() +
почти 19% сэмплов на обращения к sstable, это вероятнее всего запросы, из-за которых выросли 99.9+ перцентили


Для сравнения посмотрим, как будет вести себя синхронная версия с 64 потоками и соединениями:
```
wrk2 -d 30 -t 64 -c 64 -R 153000 -L -s get.lua http://127.0.0.1:8080
Running 30s test @ http://127.0.0.1:8080
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    11.70ms   36.13ms 434.43ms   93.23%
    Req/Sec     2.50k   482.72     9.40k    87.46%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%    1.40ms
 75.000%    2.55ms
 90.000%   22.80ms
 99.000%  221.57ms
 99.900%  330.24ms
 99.990%  409.60ms
 99.999%  431.87ms
100.000%  434.69ms
```
Пропускная способность сервера ниже, примерно 153000 RPS, сравним стабильную нагрузку в 130000 RPS
```
Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.01ms  648.41us  30.21ms   77.18%
    Req/Sec     2.04k    24.20     2.79k    87.07%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%    0.99ms
 75.000%    1.34ms
 90.000%    1.67ms
 99.000%    1.98ms
 99.900%    6.44ms
 99.990%   22.85ms
 99.999%   28.58ms
100.000%   30.22ms
```

[CPU flame graph](data/stage2/profile-get-130000-sync.html)

[Alloc flame graph](data/stage2/profile-get-130000-sync-alloc.html)

[Lock flame graph](data/stage2/profile-get-130000-sync-lock.html)

В отличие от PUT запроса, в асинхронной версии для GET выросла не только средняя latency, но и максимальное время обработки запроса. 
Видимо, сыграло роль время, которое тратится на удержание блокировок в очереди, а также сказывается наполненность бд, при тестировании асинхронного
варианта записей было больше и обращения к sstable чаще. 

Оптимизировать асинхронную версию можно используя разные размеры или реализации очередей. Например, при 
использовании LinkedBlockingQueue latency увеличивается еще больше (примеры профилирования: [CPU flame graph](data/stage2/profile-put-130000-LinkedBlockingQueue.html),
[Alloc flame graph](data/stage2/profile-put-130000-LinkedBlockingQueue-alloc.html)). 

Аналогично с PUT посмотрим, что будет, если уменьшить пул потоков до 8
```
lena@Khodosovas-Air Downloads % wrk2 -d 60 -t 64 -c 64 -R 130000 -L -s get.lua http://127.0.0.1:8080
Running 1m test @ http://127.0.0.1:8080
  64 threads and 64 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.42ms    4.25ms 130.94ms   99.15%
    Req/Sec     2.07k    83.25     3.06k    80.94%
  Latency Distribution (HdrHistogram - Recorded Latency)
 50.000%    1.11ms
 75.000%    1.50ms
 90.000%    1.86ms
 99.000%    4.11ms
 99.900%   78.91ms
 99.990%  112.13ms
 99.999%  129.73ms
100.000%  131.01ms
```
Средняя latency немного выросла, как и максимальное время обработки запроса.

### Выводы
Работа с потоками суммарно занимает даже больше времени, чем сама бизнес-логика, выполняемая приложением, поэтому при 1
соединении и 1 потоке асинхронная версия немного проигрывает синхронной. При увеличении количества потоков и соединений 
средняя latency в асинхронной реализации значительно увеличивается, так как больше времени тратится на ожидание блокировок.
Однако, максимальная пропускная способность сервиса все-таки увеличивается, что уже хорошо.

Из всех параметров ThreadPoolExecutor, на пропускную способность сервиса влияют сильнее всего резмер очереди и количество 
используемых потоков. Опытным путем было получено, что для данной реализации оптимальный размер очереди - 256, а для пула потоков - 64