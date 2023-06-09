
# Первое соревнование от MTS Digital Big Data

![image](https://user-images.githubusercontent.com/111744291/233374322-52cdf42d-9321-4e9c-84bc-1f80d7c4e85e.png)

<b> В репазитории размещены материалы и решение по заданию с соревнования MTS ML CUP 2023 </b>

## Состав репазитория:
- Finish_prepare_data.ipynb - авторский блокнот с кодом для считывания, агрегации и первичной подготовки сырых данных для дальнейшего анализа и построения моделей
- Finish_calc_features and predict.ipynb - авторский блокнот с кодом для EDA данных, дизайна новых признаков, машинного обучения и предиктивного анализа
- mts27.xls - результаты предсказания для закрытых данных
- подкаталог data с данными от организаторов

## Описание:
Целью соревнования было предсказать для каждого пользователя из закрытой выборки две целевые переменные (пол и возрастную группу). В качестве данных для обучения был предоставлен набор паркетных файлов с данными о запросах, поступающих с сайтов, которые посещали пользователи, содержащих:
- название сайта
- дата посещения
- время посещения (утро, день, вечер, ночь)
- данные устройства, с которого поступила информация (тип, производитель, модель, цена)
- область
- город.

Решение задания разбито на два блокнота:
1. В первом (Finish_prepare_data.ipynb) реализовано начальная обработка данных:
- считывание паркетных файлов
- агрегация записей по идентификаторам пользователей с расчетом агрегированных признаков по каждому пользователю:
- списка всех названий сайтов, размещенных в хронологической последовательности;
- количества посещений по времени суток (утром, днем, вечером, ночью)
- количества посещений по дням недели (понедельник, вторник и т.д.)
- наиболее часто встречающегося города
- наиболее часто встречающегося региона
- количества уникальных городов
- количества уникальных регионов
- наиболее часто встречающегося типа устройства
- наиболее часто встречающейся модели устройства
- средней цены устройства.

2. Во втором (Finish_calc_features and predict.ipynb) реализованы:
- базовая преодобработка (уточнение типов данных, обработка пропусков)
- дизайн новых признаков, связанных с названиями сайтов
- разделение выборки на закрытую и открытую, кторая в свою очередь поделена на тренироввочную и тестовую
- подбор гиперпараметров для двух моделей на базе LGBMClassifier, которые по отдельности должны рассчитывать целевые переменные
- обучение моделей и расчет целевых переменных для закрытых данных
- оформление расчетов и их выгрузка в файл, который можно отправить на площадку для submit-а.

В процессе дизайна реализованы и опробованы разные подходы к повышению информативности данных и снижению их размерности. Основные из них:
1. Удаление из названий сайтов доменных зон, чисел и повторяющихся комбинаций типа (подход показал свою эфективность и оставлен в решении)
2. Замена названий сайтов на их категории (автотовары, спорт, игры и т.д.), рассчитанные на основании отдельного общедоступного датафрейма, в котором храняться записи с названиями сайтов и их категориями с несколькими уровнями конкретизации (не дал заметного эффекта, в конечном варианте решения не применялся)
3. Создание дополнительных столбцов для каждого из популярных релевантных сайтов (*) с последующим расчетом числа посещений этих сайтов для каждого из пользователей

(*) популярные релевантные сайты по полу получаются так:
- рассчитывается список самых популярных сайтов для мужчин
- рассчитывается список самых популярных сайтов для женщин
- из списков выбираются те сайты, которые находять вне пересечения полученных множеств, из них сохраняются 10 самых популярных
- для этих популярных и уникальных сайтов составляются доп. столбцы и расчитывается количество посещений сайта столбца каждым пользователем

(**) популярные релевантные сайты по возрастной группе получаются аналогично, но стратификация проводиться по возрастной группе

4. Создание дополнительных столбцов для каждой из популярных релевантных категорий сайтов (*) с последующим расчетом числа посещений этих категорий для каждого из пользователей (подход не оправдал себя и в конечное решение не попал)

В результате моделирования была достигнута итоговая метрика, равная 1.62, рассчитанная по формуле:
```
score = 2*f1 + 2*roc_auc - 1,  где:
- f1 - метрика для задачи классификации возрастных групп (f1_score(y_true, y_pred, average = 'weighted'))
- roc_auc - метрика для задачи классификации по полу (roc_auc_score(y_true, y_pred)
```

 ## Что можно улучшить:
 1. Добавить новые признаки на этапе агрегации.
 1. Использовать CatBoost вместо LightGBM (к сожалению, скромные вычислительные мощности этому препятствовали)
 2. Не кодировать категориальные переменные, доверить это бустинговой модели (LightGBM и CatBoost умеют рабоать с ними напрямую).
 3. Подготавливать отдельные выборки с отдельными признаками (особенно по релевантным сайтам) для модели классификации по полу и модели классификации по возрастной группе.
 4. Подружить название сайта и время его посещения и создать новый столбец со списками сайтов, состоящими из пар (время, название сайта). 
 5. Использовать нейросети для преобработки тектов и генерирования признаков, на которых затем обучить бустинговую модель.
