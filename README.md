# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #3 выполнил(а):
- Кулева Дарья Андреевна
- РИ210936
Отметка о выполнении заданий (заполняется студентом):

| Задание | Выполнение | Баллы |
| ------ | ------ | ------ |
| Задание 1 | * | 60 |
| Задание 2 | * | 20 |
| Задание 3 | # | 20 |

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
Познакомиться с программными средствами для создания системы машинного обучения и ее интеграции в Unity.

## Задание 1
### Реализовать систему машинного обучения в связке Python - Google-Sheets – Unity.
Ход работы:

- В созданный проект добавьте ML Agent

![image](https://user-images.githubusercontent.com/113285427/203013836-99d90d51-9605-4449-b981-8613ec145975.png)

- Далее пишем серию команд для создания и активации нового ML-агента, а также для скачивания необходимых библиотек:
-устанавливаем MLAgent, необходим для связки Unity-Python для просчетов
![image](https://user-images.githubusercontent.com/113285427/201112278-075eb9c6-18a8-4dd0-af0a-71532c294b3d.png)

-torch использует ПК для создании потоков, распределений
![image](https://user-images.githubusercontent.com/113285427/201112373-dbfbc0b0-76fb-4aa5-b93c-dfdc61e2db1b.png)

-Добавляем RollerAgent скрипт RollerAgent.cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Unity.MLAgents;
using Unity.MLAgents.Sensors;
using Unity.MLAgents.Actuators;

public class RollerAgent : Agent
{
    Rigidbody rBody;
    // Start is called before the first frame update
    void Start()
    {
        rBody = GetComponent<Rigidbody>();
    }

    public Transform Target;
    public override void OnEpisodeBegin()
    {
        if (this.transform.localPosition.y < 0)
        {
            this.rBody.angularVelocity = Vector3.zero;
            this.rBody.velocity = Vector3.zero;
            this.transform.localPosition = new Vector3(0, 0.5f, 0);
        }

        Target.localPosition = new Vector3(Random.value * 8-4, 0.5f, Random.value * 8-4);
    }
    public override void CollectObservations(VectorSensor sensor)
    {
        sensor.AddObservation(Target.localPosition);
        sensor.AddObservation(this.transform.localPosition);
        sensor.AddObservation(rBody.velocity.x);
        sensor.AddObservation(rBody.velocity.z);
    }
    public float forceMultiplier = 10;
    public override void OnActionReceived(ActionBuffers actionBuffers)
    {
        Vector3 controlSignal = Vector3.zero;
        controlSignal.x = actionBuffers.ContinuousActions[0];
        controlSignal.z = actionBuffers.ContinuousActions[1];
        rBody.AddForce(controlSignal * forceMultiplier);

        float distanceToTarget = Vector3.Distance(this.transform.localPosition, Target.localPosition);

        if(distanceToTarget < 1.42f)
        {
            SetReward(1.0f);
            EndEpisode();
        }
        else if (this.transform.localPosition.y < 0)
        {
            EndEpisode();
        }
    }
}

- Далее, пропуская некоторые пункты, описанные в Методичке, переходим к пункту "Запустите работу ml-агента"
![image](https://user-images.githubusercontent.com/113285427/203015127-1f5ef44f-fc9d-4b3b-8624-68aac355a35b.png)

- После завершения обучения проверяем работу модели.
![image](https://user-images.githubusercontent.com/113285427/203015370-2fe70b01-0c91-4e66-8ac8-1924cc24ed3c.png)

- Вывод: В ходе обучения, RollerAgent научился катиться в сторону куба и не падать с платформы, а точнее частота и колличество падений заметно сократились. За время последнего наблюдения я не заметил ни одного случая падения с поатформы, шар всегда следует а кубом. А значит обучение прошло успешно. 

## Задание 2
### Подробно опишите каждую строку файла конфигурации нейронной сети. Самостоятельно найдите информацию о компонентах Decision Requester, Behavior Parameters, добавленных на сфере.

- Decision Requester - цикл наблюдение-решение-действие-вознаграждение повторяется каждый раз, когда Agent запрашивает решение. Агенты запросят решение, когда вызовется Agent.Request Decision(). Если нужно, чтобы агент запрашивал решения самостоятельно через регулярные промежутки времени, то добавляетя компонент Decision Requester в GameObject агента. Принятие решений с регулярными интервалами шагов, как правило, наиболее подходит для моделирования на основе физики. Например, агент в роботизированном симуляторе, который должен обеспечивать точный контроль крутящих моментов в суставах, должен принимать свои решения на каждом этапе моделирования. 

- Behavior Parameters - Класс Policy абстрагирует логику принятия решений от самого агента, так что можно использовать одну и ту же политику в нескольких агентах. То, как Policy принимает свои решения, зависит от Behavior Parameters, связанных с агентом. Если установить для типа поведения значение Behavior Type to Heuristic Only, агент будет использовать свой метод Heuristic()  для принятия решений, которые позволяют управлять агентом вручную или писать свою собственную Policy. Если у агента есть файл модели, его политика будет использовать модель нейронной сети для принятия решений, как в нашей лабораторной. 


## Задание 3
### Доработайте сцену и обучите ML-Agent таким образом, чтобы шар перемещался между двумя кубами разного цвета. Кубы должны, как и в первом задании, случайно изменять координаты на плоскости.



## Выводы

В ходе выполнения лабораторной работы:
- Я установила такие ПО как: Unity, Visual Studio Code, Anacondа;
- Используя элементарные функции языка Python, вывела на экран надпись "Hello World" в Google Colab, ознакомилась с функциями облачной среды;
- Используя элементарные функции языка C#, вывела на консоль надпись "Hello World" в Unity, ознакомилась с функциями платформы;
- А также в ходе выполнения второго задания познакомилась с основными операторами языка Python на примере реализации линейной регрессии. 
## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**

