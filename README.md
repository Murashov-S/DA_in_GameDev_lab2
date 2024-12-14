# Анализ данных в разработке игр
Отчет по лабораторной работе #2 выполнил:
- Мурашов Сергей Алексеевич
- НМТ-233004

## Отметка о выполнении заданий
| Задание   | Выполнение | Баллы |
| --------- | ---------- | ----- |
| Задание 1 | *          | 60    |
| Задание 2 | *          | 20    |
| Задание 3 | *          | 20    |

Знак "*" - задание выполнено; знак "#" - задание не выполнено;

## Работу проверили
- к.т.н., доцент Денисов Д.В.
- к.э.н., доцент Панов М.А.
- ст. преп., Фадеев В.О.

## Структура отчета
- Данные о работе: название работы, фио, группа, выполненные задания.
- Цель работы.
- Задание 1.
- Задание 2.
- Задание 3.
- Выводы.

## Цель работы
Научиться передавать в Unity данные из Google Sheets с помощью Python.

### Задание 1
Задание: Выбрать игровую переменную и описать её роль в игре, условия изменения / появления и диапазон допустимых значений. Построить схему экономической модели в игре и указать место выбранного ресурса в ней.  
Ход работы:
- Выбрать игровую переменную.
- Описать роль выбранной игровой переменной в игре.
- Описать условия изменения / появления игровой переменной в игре.
- Описать диапазон допустимых значений игровой переменной в игре.
- Построить схему экономической модели в игре и указать место выбранного ресурса в ней.  

#### Выбранная игровая переменная
Монеты.

#### Роль монет в игре
Игровая валюта необходимая для покупки патронов и оружия.

#### Условия изменения
- +Убийство зомби.
- +Подбор монет из сундуков.
- +Просмотр рекламы.
- +Оценка игры.
- -Покупки в магазине.

#### Диапазон допустимых значений
[0; +inf)

#### Схема экономической модели для монет
![Схема](./1.png)  
Монеты занимают ключевую роль в экономике игры.

### Задание 2
Задание: С помощью python заполните google-таблицу и визуализируйте данные. Опишите характер изменения выбранной игровой переменной.  
Ход работы:
- Написать скрипт на языке Python, который заполнит google-таблицу данными, описывающими выбранную игровую переменную.
- Визуализировать данные в google-таблице.
- Описать характер изменения этой величины и недостатки в реализации этой величины.

#### Скрипт для заполнения google-таблицы
Будем рассматривать получение монет только при убийстве зомби. А трату монет только на покупку патронов к начальному оружию. Покупка будет происходить в конце каждого раунда, покупать патроны будем на все монеты.  
Рассмотрим 3-х игроков:
- Идеальный игрок, который всеми выстрелами попадает в голову.
- Средний игрок, который на одного зомби в среднем тратит по 3 выстрела.
- Плохой игрок, который на одного зомби в среднем тратит по 5 выстрелов.

``` python
import gspread
import numpy as np

# Подключение к Google Sheets
gc = gspread.service_account(filename='da-in-gamedev-lab-2-c9299262864c.json')
sh = gc.open('DA-in-GameDev-lab-2')
worksheet = sh.sheet1

# Очистка таблицы от старых данных
sh.sheet1.clear() 

# Параметры
players_count     = 3   # Количество типов игроков

total_rounds      = 15  # Всего раундов

zombies_started   = 10  # Начальное количество зомби
zombies_per_round = 5   # Увеличение количества зомби каждый раунд
price_zombie      = 1   # Награда за убийство зомби

head_shot         = 1   # Необходимо выстрелов в голову для убийста
body_shot         = 9   # Необходимо выстрелов в тело для убийста

current_coins     = 0   # Текущие количество монет
current_ammo      = 150 # Текущие количество патронов

shop_ammo         = 12  # Количество патронов в одном комплекте
price_shop_ammo   = 20  # Цена одного комлпекта патронов

# Заголовки таблицы
row = 1
sh.sheet1.update_acell('A' + str(row), 'Раунд')
sh.sheet1.update_acell('B' + str(row), 'Монеты')          # Монет на конец раунда
sh.sheet1.update_acell('C' + str(row), 'Игрок 1 патроны') # Патронов на конец раунда игрока 1
sh.sheet1.update_acell('D' + str(row), 'Игрок 2 патроны')
sh.sheet1.update_acell('E' + str(row), 'Игрок 3 патроны')

for player in range(players_count):
    current_coins = 0
    current_ammo = 150
    row = 1

    # shot_count - колличетство необходимых выстрелов игроку для убийства одного зомби
    if(player == 0):
        shot_count = head_shot
        cell_1 = 'C'
    elif(player == 1):
        shot_count = (body_shot + head_shot + head_shot + head_shot) // 4
        cell_1 = 'D'
    else:
        shot_count = (body_shot + head_shot) // 2
        cell_1 = 'E'

    for round_number in range(total_rounds):
        zombies_count = zombies_started + zombies_per_round*round_number # Количество зомби в раунде
        current_coins += zombies_count*price_zombie
        current_ammo -= zombies_count*shot_count

        row += 1
        sh.sheet1.update_acell('A' + str(row), str(round_number+1))

        if(current_ammo < 1):
            sh.sheet1.update_acell(cell_1 + str(row), "0")
            break
        else:
            k = current_coins//price_shop_ammo
            current_ammo += k*shop_ammo
            current_coins -= k*price_shop_ammo
            sh.sheet1.update_acell('B' + str(row), str(current_coins))
            sh.sheet1.update_acell(cell_1 + str(row), str(current_ammo))
```

#### Визуализация данных в google-таблице
![Google-таблица](./2.png)  

#### Описание
Как мы можем увидеть на визуализации, даже игроку с идеальной стрельбой, при заданных условиях, не хватит монет для прохождения 10 волны. Притом что, идеальная стрельба в рамках игры с автоматической стрельбой почти невозможна. В данной реализации необходимо увеличить награду за зомби, либо добавить альтернативный источник монет. Но если давать слишком большую награду, то на монеты будет нечего покупать.

### Задание 3
Задание: Настройте на сцене Unity воспроизведение звуковых файлов, описывающих динамику изменения выбранной переменной
Ход работы:
- Подготовить google-таблицу к редактированию.
- В Unity создать пустой GameObject и подключить к нему скрипт, в котором описывается подключение к google-таблице по API.
- Подключить аудиофайлы для воспроизведения к GameObject.
- Запустить сцену и проверить, что происходит воспроизведение звуковых файлов.

В нашем алгоритме мы тратим монеты на покупку патронов после каждого раунда, поэтому и будем воспроизводить звук для определенного количество патронов:
- Хорошо - количество патронов после раунда больше 100.
- Нормально - количество патронов после раунда больше 50, но меньше 100.
- Плохо - количество патронов после раунда меньше 50.

#### Скрипт
``` C#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Networking;
using SimpleJSON;

public class NewBehaviourScript : MonoBehaviour
{
    public AudioClip goodSpeak;
    public AudioClip normalSpeak;
    public AudioClip badSpeak;
    private AudioSource selectAudio;
    private Dictionary<string, float> dataSet = new Dictionary<string, float>();
    private bool statusStart = false;
    private int i = 1;

    // Start is called before the first frame update
    void Start()
    {
        StartCoroutine(GoogleSheets());
    }

    // Update is called once per frame
    void Update()
    {
        if (dataSet["Mon_" + i.ToString()] <= 50 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioBad());
            StartCoroutine(PlaySelectAudioGood());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] > 50 & dataSet["Mon_" + i.ToString()] < 100 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioNormal());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }

        if (dataSet["Mon_" + i.ToString()] >= 100 & statusStart == false & i != dataSet.Count)
        {
            StartCoroutine(PlaySelectAudioGood());
            Debug.Log(dataSet["Mon_" + i.ToString()]);
        }
    }

    IEnumerator GoogleSheets()
    {
        UnityWebRequest curentResp = UnityWebRequest.Get("https://sheets.googleapis.com/v4/spreadsheets/1O386gPuwBCap3aiPN1dM4uxYrGbxThjYO_7dTw5BUWw/values/Лист1?key=AIzaSyDJYHFa5HwWAo9PF6pSMwaie9CwnRkNDdU");
        yield return curentResp.SendWebRequest();
        string rawResp = curentResp.downloadHandler.text;
        var rawJson = JSON.Parse(rawResp);
        foreach (var itemRawJson in rawJson["values"])
        {
            
            var parseJson = JSON.Parse(itemRawJson.ToString());
            var selectRow = parseJson[0].AsStringList;

            dataSet.Add(("Mon_" + selectRow[0]), float.Parse(selectRow[2]));
        }
    }

    IEnumerator PlaySelectAudioGood()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = goodSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(3);
        statusStart = false;
        i++;
    }
    IEnumerator PlaySelectAudioNormal()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = normalSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(3);
        statusStart = false;
        i++;
    }
    IEnumerator PlaySelectAudioBad()
    {
        statusStart = true;
        selectAudio = GetComponent<AudioSource>();
        selectAudio.clip = badSpeak;
        selectAudio.Play();
        yield return new WaitForSeconds(4);
        statusStart = false;
        i++;
    }
}
```

#### Проверка на Unity
![Вывод](./3.png)  
## Выводы
- Узнал как воспроизводить звуковые файлы на Unity, которые описывают изменение переменных.
- Узнал как генерировать данные с помощью Python.
- Узнал как передавать данные из python в google-таблицу.
