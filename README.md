# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #5 выполнил(а):
- Кулева Дарья Андреевна
- РИ210936
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | * | 20 |

знак "*" - задание выполнено; знак "#" - задание не выполнено;

Работу проверили:
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

[![N|Solid](https://cldup.com/dTxpPi9lDf.thumb.png)](https://nodesource.com/products/nsolid)

[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

Структура отчета

- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 2.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Задание 3.
- Код реализации выполнения задания. Визуализация результатов выполнения (если применимо).
- Выводы.
- ✨Magic ✨

## Цель работы
Научиться интегрировать экономическую систему в Unity и обучить ей пользоваться Ml Agent.

## Задание 1
### Измените параметры файла. yaml-агента и определить какие параметры и как влияют на обучение модели.

Ход работы:

- Открыла проект Unity, доступный для скачивания в методичке.

![image](https://user-images.githubusercontent.com/113285427/205120957-6a9cac62-9a1e-451e-9517-cf104ec430db.png)

- С помощью Anaconda Prompt активировала новый ML-агент и установила новые библиотеки: mlagents 0.28.0, torch 1.7.1. 

```py
conda create -n MLAgents python=3.6
conda activate MLAgents
```

![image](https://user-images.githubusercontent.com/113285427/205110491-10a8f33e-ac9b-4ee6-af84-4c500dab1e65.png)

```py
pip install mlagents==0.28.0
```

![image](https://user-images.githubusercontent.com/113285427/205110682-e9610bd3-d37d-48ba-970a-4760f43c0069.png)

```py
pip install torch~=1.7.1 -f https://download.pytorch.org/whl/torch_stable.html
```

- Затем необходимо перейти в папку с проектом и начать обучение модели.

Файл Move.cs выглядит вот так:

```py

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unity.MLAgents;
using Unity.MLAgents.Sensors;
using Unity.MLAgents.Actuators;

public class Move : Agent
{
    [SerializeField] private GameObject goldMine;
    [SerializeField] private GameObject village;
    private float speedMove;
    private float timeMining;
    private float month;
    private bool checkMiningStart = false;
    private bool checkMiningFinish = false;
    private bool checkStartMonth = false;
    private bool setSensor = true;
    private float amountGold;
    private float pickaxeСost;
    private float profitPercentage;
    private float[] pricesMonth = new float[2];
    private float priceMonth;
    private float tempInf;

    // Start is called before the first frame update
    public override void OnEpisodeBegin()
    {
        // If the Agent fell, zero its momentum
        if (this.transform.localPosition != village.transform.localPosition)
        {
            this.transform.localPosition = village.transform.localPosition;
        }
        checkMiningStart = false;
        checkMiningFinish = false;
        checkStartMonth = false;
        setSensor = true;
        priceMonth = 0.0f;
        pricesMonth[0] = 0.0f;
        pricesMonth[1] = 0.0f;
        tempInf = 0.0f;
        month = 1;
    }
    public override void CollectObservations(VectorSensor sensor)
    {
        sensor.AddObservation(speedMove);
        sensor.AddObservation(timeMining);
        sensor.AddObservation(amountGold);
        sensor.AddObservation(pickaxeСost);
        sensor.AddObservation(profitPercentage);
    }

    public override void OnActionReceived(ActionBuffers actionBuffers)
    {
        if (month < 3 || setSensor == true)
        {
            speedMove = Mathf.Clamp(actionBuffers.ContinuousActions[0], 1f, 10f);
            Debug.Log("SpeedMove: " + speedMove);
            timeMining = Mathf.Clamp(actionBuffers.ContinuousActions[1], 1f, 10f);
            Debug.Log("timeMining: " + timeMining);
            setSensor = false;
            if (checkStartMonth == false)
            {
                Debug.Log("Start Coroutine StartMonth");
                StartCoroutine(StartMonth());
            }

            if (transform.position != goldMine.transform.position & checkMiningFinish == false)
            {
                transform.position = Vector3.MoveTowards(transform.position, goldMine.transform.position, Time.deltaTime * speedMove);
            }

            if (transform.position == goldMine.transform.position & checkMiningStart == false)
            {
                Debug.Log("Start Coroutine StartGoldMine");
                StartCoroutine(StartGoldMine());
            }

            if (transform.position != village.transform.position & checkMiningFinish == true)
            {
                transform.position = Vector3.MoveTowards(transform.position, village.transform.position, Time.deltaTime * speedMove);
            }

            if (transform.position == village.transform.position & checkMiningStart == true)
            {
                checkMiningFinish = false;
                checkMiningStart = false;
                setSensor = true;
                amountGold = Mathf.Clamp(actionBuffers.ContinuousActions[2], 1f, 10f);
                Debug.Log("amountGold: " + amountGold);
                pickaxeСost = Mathf.Clamp(actionBuffers.ContinuousActions[3], 100f, 1000f);
                Debug.Log("pickaxeСost: " + pickaxeСost);
                profitPercentage = Mathf.Clamp(actionBuffers.ContinuousActions[4], 0.1f, 0.5f);
                Debug.Log("profitPercentage: " + profitPercentage);

                if (month != 2)
                {
                    priceMonth = pricesMonth[0] + ((pickaxeСost + pickaxeСost * profitPercentage) / amountGold);
                    pricesMonth[0] = priceMonth;
                    Debug.Log("priceMonth: " + priceMonth);
                }
                if (month == 2)
                {
                    priceMonth = pricesMonth[1] + ((pickaxeСost + pickaxeСost * profitPercentage) / amountGold);
                    pricesMonth[1] = priceMonth;
                    Debug.Log("priceMonth: " + priceMonth);
                }

            }
        }
        else
        {
            tempInf = ((pricesMonth[1] - pricesMonth[0]) / pricesMonth[0]) * 100;
            if (tempInf <= 6f)
            {
                SetReward(1.0f);
                Debug.Log("True");
                Debug.Log("tempInf: " + tempInf);
                EndEpisode();
            }
            else
            {
                SetReward(-1.0f);
                Debug.Log("False");
                Debug.Log("tempInf: " + tempInf);
                EndEpisode();
            }
        }
    }

    IEnumerator StartGoldMine()
    {
        checkMiningStart = true;
        yield return new WaitForSeconds(timeMining);
        Debug.Log("Mining Finish");
        checkMiningFinish = true;
    }

    IEnumerator StartMonth()
    {
        checkStartMonth = true;
        yield return new WaitForSeconds(60);
        checkStartMonth = false;
        month++;

    }
}

```

- Обучим модель и с помощью графиков и посмотрим на результат обучения:

```py
mlagents-learn Economic.yaml --run-id=Economic –-force
```

![image](https://user-images.githubusercontent.com/113285427/205111552-4ab55dc6-0d31-49cc-a788-2c25c2c1cc35.png)

![image](https://user-images.githubusercontent.com/113285427/205111583-25d88c41-9513-474f-9662-86bc636e4f44.png)

Установим TensorBoard для оценки результатов обучения:

```py
pip install tensorflow
```

![image](https://user-images.githubusercontent.com/113285427/205111705-56a87e94-4805-4245-b255-e57f31839b95.png)

Вот такие результаты можно будет увидеть после выполнения команды, перейдя на локальный хост - http://localhost:6006/:

```py
tensorboard --logdir=results\Economic
```

![image](https://user-images.githubusercontent.com/113285427/205112088-595efe4b-ad44-4183-950a-ad59f74b2f4a.png)

В первом запуске использовались следующие параметры в .yaml файле:

```py
behaviors:
  Economic:
    trainer_type: ppo
    hyperparameters:
      batch_size: 1024
      buffer_size: 10240
      learning_rate: 1.0e-4
      learning_rate_schedule: linear
      beta: 1.0e-2
      epsilon: 0.2
      lambd: 0.95
      num_epoch: 3      
    network_settings:
      normalize: false
      hidden_units: 128
      num_layers: 2
    reward_signals:
      extrinsic:
        gamma: 0.99
        strength: 1.0
    checkpoint_interval: 500000
    max_steps: 750000
    time_horizon: 64
    summary_freq: 5000
    self_play:
      save_steps: 20000
      team_change: 100000
      swap_steps: 10000
      play_against_latest_model_ratio: 0.5
      window: 10
```

Cumulative Reward должно увеличиваться во время успешной тренировки (наш график немонотонный), а Policy Loss - уменьшаться (наш график этому соответствует).

В следующих запусках мы будем менять значение одного из параметров и смотреть, как он будет влиять на обучение модели.

- При втором запуске будем менять параметр strength: увеличим его в двое. Это фактор, на который можно умножить вознаграждение, получаемое от окружающей среды.

```py
strength: 2.0
```

![image](https://user-images.githubusercontent.com/113285427/205112503-7e99bdc0-d76d-47db-a797-e84e977e0bb8.png)

![image](https://user-images.githubusercontent.com/113285427/205112530-888a5949-9388-4aea-a94c-72a8cc646c3a.png)

Cumulative Reward всегда равно единице, а Policy Loss - растет вверх. Мы не можем назвать это успешной тренировкой.

- При третьем запуске мы увеличим параметр epsilon на 0.1. Этот параметр влияет на то, насколько быстро политика может развиваться во время обучения. Соответствует допустимому порогу расхождения между старой и новой политиками при обновлении с градиентным спуском. Установка этого малого значения приведет к более стабильным обновлениям, но также замедлит процесс обучения. В нашем случае, он приведет к менее стабильным обновлениям

```py
epsilon: 0.3
```

![image](https://user-images.githubusercontent.com/113285427/205112802-707e71b5-8503-46af-8d7b-c038fe652388.png)

![image](https://user-images.githubusercontent.com/113285427/205112824-45d10c94-f94f-4c4b-a54e-1dd5693d6d5b.png)

Cumulative Reward идет вниз, а Policy Loss - также идет вниз. Мы не можем назвать это успешной тренировкой.

- При четвертом запуске поработаем с параметром learning_rate: уменьшим его в три раза. Этот параметр - это начальная скорость обучения для градиентного спуска. Соответствует силе каждого шага обновления градиентного спуска. Обычно это значение следует уменьшить, если тренировка нестабильна, а вознаграждение постоянно не увеличивается.

```py
learning_rate: 1.0e-4
```

![image](https://user-images.githubusercontent.com/113285427/205113078-788804f9-867b-458a-8d3b-166cf8623fa7.png)

![image](https://user-images.githubusercontent.com/113285427/205113094-8e81fa91-8c33-4c6c-881a-d6106e96d35a.png)

Cumulative Reward всегда равно единице, а Policy Loss - уменьшается. То есть, среднее нашего вознаграждения вседа равно единице, то есть оно однообразно, что не есть интересно.

- И при пятом запуске мы снова поработаем с epsilon: уменьшим его до 0.1 для стабилизации обновлений.

```py
epsilon: 0.1
```

![image](https://user-images.githubusercontent.com/113285427/205113389-d08a028c-c0bd-4aa6-b87a-33822cad63ed.png)

![image](https://user-images.githubusercontent.com/113285427/205113422-1ffbfbcf-fdfb-4952-b937-1b2dc5620382.png)

Cumulative Reward всегда увеличивается, а Policy Loss - уменьшается. Это самая успешная тренировка из пяти.

- Посмторим на графики всех пяти тренировок вместе:

![image](https://user-images.githubusercontent.com/113285427/205113531-c4972879-68bb-482f-aa81-d35216543a3e.png)

![image](https://user-images.githubusercontent.com/113285427/205113569-431aa576-5458-4dd1-b3b9-ec1fcf3957e3.png)

Можно сделать небольшой вывод: параметры epsilon, learning_rate и strength оказывают влияние на обучение модели. Лучший результат показало уменьшение параметра epsilon.

## Задание 2
### Опишите результаты, выведенные в TensorBoard.

- График Cumulative Reward: среднее совокупное вознаграждение за эпизод по всем агентам. Должно увеличиваться во время успешной тренировки. О значении на каждой тренировке было сказано в задании 1.

- График Episode Length: Средняя продолжительность каждого эпизода в окружающей среде для всех агентов. Во всех пяти тренировках был одинаковый.

- График Policy Loss: Средняя величина функции потерь. Соответствует тому, насколько сильно меняется политика (процесс принятия решений о действиях). Этот график должен уменьшаться во время успешной тренировки.

- График Value Loss: Средняя потеря функции обновления значения. Соответствует тому, насколько хорошо модель способна предсказать значение каждого состояния. Этот график должен увеличиваться, пока агент учится, а затем уменьшаться, как только вознаграждение стабилизируется. Во всех пяти тренировках график идет монотонно вниз.

- Графики в разделе Policy показывают изменение некоторых параметров, которые указаны в .yaml файле

- График Beta: У всех пяти тренировок одинаковый: идет монотонно вниз.

- График Entropy: Показывает насколько случайны решения модели. Должен медленно уменьшаться во время успешного тренировочного процесса. Уменьшается при первой и второй попытках, в остальных - увеличивается. (значит, удачная пятая попытка не такая уж и успешная).

- График Epsilon: во всех пяти случаях уменьшается (графики имеют разные начальные значения, так как мы их меняли сами, например при 3 и 5 тренировках).

- График Extrinsic Reward: Этот график соответствует среднему совокупному вознаграждению, полученному от окружающей среды за эпизод. Во втором графике значение 2, так как мы увеличивали параметр strength в два раза.

- График Extrinsic Value Estimate: Оценка среднего значения для всех положений, посещенных агентом. Должно увеличиваться во время успешной тренировки. При 3 и 4 попытках графики сначал растут, а потом уменьшаются, в остальных случаях - наоборот.

- График Learning Rate: все пять графиков идут вниз (4 график имеет другое начальное значение, так как мы сами его таким задали).

С помощью графиков можно понять, какие параметры стоит изменить или оставить для лучшей тренировки модели.


## Выводы
В ходе лабораторной работы интегрировала экономическую систему в проект Unity и обучила ML-Agent. Поработала с параметрами в .yaml файле: меняла значения для изменения обучения модели. В ходе этого выяснила, как некоторые параметры могут влиять на тренировку модели: strength - коэффициент, на который умножается вознаграждение, epsilon - влияет на стабильность обновлений, learning_rate - начальная скорость обучения. С помощью графиков, выведенных в TensorBoard анализировала попытки обучения модели, какие-то были более успешными, а другие - менее. Для поиска наилучшего обучения можно эксперементировать с параметрами в .yaml файле и при этом анализировать графики.

## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**

