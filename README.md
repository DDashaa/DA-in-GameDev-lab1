# АНАЛИЗ ДАННЫХ И ИСКУССТВЕННЫЙ ИНТЕЛЛЕКТ [in GameDev]
Отчет по лабораторной работе #4 выполнил(а):
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
Ознакомиться с основными операторами зыка Python на примере реализации линейной регрессии.

## Задание 1
### В проекте Unity реализовать перцептрон, который умеет производить вычисления:
- OR |дать комментарии о корректности работы
- AND |дать комментарии о корректности работы
- NAND |дать комментарии о корректности работы
- XOR |дать комментарии о корректности работы

Ход работы:
- Создаю пустой проект на Unity. Добавляю пустой объект Empty Game Object с именем Perceptron. 
![image](https://user-images.githubusercontent.com/113285427/204619293-2c5ef24b-ed82-426c-9073-549326192254.png)

- Подключаю Perceptron.cs к пустому Game Object с именем Perceptron.

```py

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[System.Serializable]
public class TrainingSet
{
	public double[] input;
	public double output;
}

public class Perceptron : MonoBehaviour {

	public TrainingSet[] ts;
	double[] weights = {0,0};
	double bias = 0;
	double totalError = 0;

	double DotProductBias(double[] v1, double[] v2) 
	{
		if (v1 == null || v2 == null)
			return -1;
	 
		if (v1.Length != v2.Length)
			return -1;
	 
		double d = 0;
		for (int x = 0; x < v1.Length; x++)
		{
			d += v1[x] * v2[x];
		}

		d += bias;
	 
		return d;
	}

	double CalcOutput(int i)
	{
		double dp = DotProductBias(weights,ts[i].input);
		if(dp > 0) return(1);
		return (0);
	}

	void InitialiseWeights()
	{
		for(int i = 0; i < weights.Length; i++)
		{
			weights[i] = Random.Range(-1.0f,1.0f);
		}
		bias = Random.Range(-1.0f,1.0f);
	}

	void UpdateWeights(int j)
	{
		double error = ts[j].output - CalcOutput(j);
		totalError += Mathf.Abs((float)error);
		for(int i = 0; i < weights.Length; i++)
		{			
			weights[i] = weights[i] + error*ts[j].input[i]; 
		}
		bias += error;
	}

	double CalcOutput(double i1, double i2)
	{
		double[] inp = new double[] {i1, i2};
		double dp = DotProductBias(weights,inp);
		if(dp > 0) return(1);
		return (0);
	}

	void Train(int epochs)
	{
		InitialiseWeights();
		
		for(int e = 0; e < epochs; e++)
		{
			totalError = 0;
			for(int t = 0; t < ts.Length; t++)
			{
				UpdateWeights(t);
				Debug.Log("W1: " + (weights[0]) + " W2: " + (weights[1]) + " B: " + bias);
			}
			Debug.Log("TOTAL ERROR: " + totalError);
		}
	}

	void Start () {
		Train(8);
		Debug.Log("Test 0 0: " + CalcOutput(0,0));
		Debug.Log("Test 0 1: " + CalcOutput(0,1));
		Debug.Log("Test 1 0: " + CalcOutput(1,0));
		Debug.Log("Test 1 1: " + CalcOutput(1,1));		
	}
	
	void Update () {
		
	}
}

```

- OR |Для корректной работы понадобилось 4 эпохи

![image](https://user-images.githubusercontent.com/113285427/204625866-0110e61c-1040-4512-aa12-9951a7bdd8d4.png)

- AND |Для корректной работы оптимальным значением количества эпох является 7

![image](https://user-images.githubusercontent.com/113285427/204628027-27fe1561-e9fb-480f-a78f-8004b97d296d.png)

- NAND |Для корректной работы понадобилось 6 эпох

![image](https://user-images.githubusercontent.com/113285427/204629221-c89ef1c6-aee6-4dfb-9004-ea02b263077f.png)

- XOR |Корректной работы не получилось добиться, колличество ошибок стандартно 4, 
максимальное значение Train использовала 1000

![image](https://user-images.githubusercontent.com/113285427/204630687-093d1bba-5703-4716-bbc8-b2eaebe584f4.png)


## Задание 2
### Построить график зависимости количества эпох от ошибки обучения. Указать от чего зависит необходимое количество эпох обучения.

Проведя эксперемент несколько раз, на основе полученных данных, я составила диаграммы изменения колличества ошибок от значения эпохи. 

![image](https://user-images.githubusercontent.com/113285427/204638941-c27bdbfa-e042-4f64-a1e6-95eca1c70f8c.png)

![image](https://user-images.githubusercontent.com/113285427/204638986-f5c05dd6-72b0-495b-9ed3-61ba8d53d2e7.png)

![image](https://user-images.githubusercontent.com/113285427/204639054-2f3b2642-89ce-401e-8853-ed140850101d.png)

![image](https://user-images.githubusercontent.com/113285427/204639093-96e50cad-9257-4d34-befc-ffa34a63aaf0.png)



## Задание 3
### Построить визуальную модель работы перептрона на сцене Unity.


## Выводы

В ходе выполнения лабораторной работы:
- Я установила такие ПО как: Unity, Visual Studio Code, Anacondа;
- Используя элементарные функции языка Python, вывела на экран надпись "Hello World" в Google Colab, ознакомилась с функциями облачной среды;
- Используя элементарные функции языка C#, вывела на консоль надпись "Hello World" в Unity, ознакомилась с функциями платформы;
- А также в ходе выполнения второго задания познакомилась с основными операторами языка Python на примере реализации линейной регрессии. 
## Powered by

**BigDigital Team: Denisov | Fadeev | Panov**

