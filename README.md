# Классификация видеороликов на YouTube

## **Описание проекта**
**Заказчик**  
Компания "Союзмультфильм". Основной вид деятельности Студии "Союзмультфильм" - производство контента, анимационных фильмов. 

Для оценки популярности проектов студия мониторит рынок и собирает статистику. И очень важно агрегировать статистику именно по проектам, а не отдельным роликам. Поэтому ролики классифицируются по принадлежности к проектам. Необходимо создать автоматизированное решение по определению принадлежности ролика к тому или иному проекту на основе анализа текстового описания видеороликов и другой доступной информации. 

## **Цель**  
Предложить решение для классификации роликов по проектам.
1. Максимизировать метрику f1
2. Создать модуль (класс, функцию) для определения проекта: 
    * на вход подается список уникальных идентификаторов yt_reel_id
    * на выходе словарь {‘yt_reel_id’: идентификатор, ‘cartoon’: название_проекта_или_none}
    * для роликов, которые не входят ни в один проект, в поле cartoon должна возвращаться строка ‘none’
  
## Структура репозитория:

| #    | Наименование файла                | Описание   |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1. | [README.md](https://github.com/IliaShi/souz_mulfilm/blob/main/README.md) | Основная информация по проекту и его результатах   |
| 2. | [reels_classification.ipynb](https://github.com/IliaShi/souz_mulfilm/blob/main/reels_classification.ipynb) | Тетрадка с результатами работы |
| 3. |[own_functions.py](https://github.com/IliaShi/souz_mulfilm/blob/main/own_functions.py) | Модуль с собственными функциями для обзора и подготовки данных |
| 4. |[reels_clf.py](https://github.com/IliaShi/souz_mulfilm/blob/main/reels_clf.py) | Модуль с итоговым решением для классификации роликов |
| 5. | [demonstration.ipynb](https://github.com/IliaShi/souz_mulfilm/blob/main/demonstration.ipynb) | Тетрадка, демонтрирующая работу модуля reels_clf.py |
| 6. |[data_enrichment_dict.json](https://github.com/IliaShi/souz_mulfilm/blob/main/data_enrichment_dict.json) |json-файл с данными для обогащения тренировочного датасета, содержит словарь в формате: ключ - id ролика, значение - метка класса |
| 7. |[target_errors.json](https://github.com/IliaShi/souz_mulfilm/blob/main/target_errors.json) | json-файл с данными для коррекции ошибок разметки в тренировочном датасете, содержит словарь в формате: ключ - id ролика, значение - правильная метка класса |
| 8. | [requirements.txt](https://github.com/IliaShi/souz_mulfilm/blob/main/requirements.txt)) | Список зависимостей |

## **Описание данных**
|Параметр|Описание|
|---|---|
|date|дата, когда ролик появился на ютубе  |
|reel_name|название ролика  |
|yt_reel_id  |уникальный идентификатор ролика на ютубе  |
|cartoon |название проекта, целевая переменная  |
|url |ссылка на ролик (включает идентификатор)  |
|text |текст сниппета, включает название ролика и описание  |
|seconds | длительность|
|is_shorts | вертикальные видеоролики продолжительностью <60сек  |
|broadcast | лайвы, прямые эфиры|
|yt_channel_id |идентификатор ютуб канала  |
|yt_channel_name |название ютуб канала  |
|yt_ch_url |ссылка на ютуб канал  |
|yt_channel_type |тип канала (Мультфильмы, Детские, Блогеры, Shorts…)  |
|flag_closed |ютуб канал закрыт, если 1  |
|international |метка международных каналов, каналов на иностранном языке (переведенный контент)  |
|language| язык|

## Выводы по исследовательскому анализу данных.
1. Имеем обучающий датасет: 83 тыс. наблюдений, 15 параметров, 45 классов.
2. Имеем резкий дисбаланс классов. Количество наблюдений для мажорного класса 'none' и минорных (например 'Вспыш') отличаются в 26 000 раз.
3. Перспективные для обучения модели параметры:
    * reel_name, text - основные параметры, содержащие информацию о видео.
    * seconds, is_shorts, broadcast, yt_channel_id (=yt_channel_name), yt_channel_type, international - дополнительные параметры, которые могут быть ассоциированы с таргетом.
4. Параметры, использование которых для обучения нецелесообразно:
    * Дата и год выпуска - использовать нецелесообразно, т.к. при использовании в последующем все новые ролики будут иметь более позднюю дату выпуска, чем в трейне.
    * language - 99,4% значение не определено.
    * yt_reel_id, url, yt_ch_url, flag_closed - технические данные, не несущие информации о свойствах контента.
5. Выявлены ошибки при разметке обучающей выборки:
    * В классе "ЖилаБыла Царевна" - примесь других мультфильмов из других проектов ("Кротик и панда", "Пип и Альба" - не отслеживаются согласно ТЗ, т.е. 'none'), правильная разметка 73%.  
    * В классе "Царевны" примесь видео "ЖилаБыла Царевна", доля правильной разметки всего лишь 29%.
    * В классе "none" - найдены единичные ролики из других классов, но проверить весь датасет не представляется возможным.
6. Тестовый датасет:
    * 55 тыс.наблюдений.
    * Аналогичен по структуре и свойствам данных обучающему.

## В процессе поиска решения проведены эксперименты для проверки ряда гипотез:
| Эксперимент | Эффект на метрику| Вывод |
|---|---|---|
|Использование параметра reel_name вместо text| <span style='color: green;'>+18%</span> |берем в качестве параметра reel_name вместо text|
|Очистка датасета от знаков препинания и чисел| <span style='color: green;'>+1%</span> | оставляем в алгоритме обработки данных |
|Очистка датасета от эмодзи| 0 | оставляем в алгоритме обработки данных |
|Удаление стоп-слов| <span style='color: green;'>+0,1%</span> | оставляем в алгоритме обработки данных|
|Добавление категориального признака yt_channel_id| <span style='color: green;'>+1%</span>  |оставляем в качестве признака |
|Добавление признака is_shorts| <span style='color: red;'>-1%</span>  | |
|Добавление признака broadcast| <span style='color: red;'>-9%</span>  | |
|Добавление признака yt_channel_type| <span style='color: red;'>-2%</span>  | |
|Добавление признака closed| <span style='color: red;'>-1%</span>  | |
|Добавление признака "длина 'text'"| <span style='color: green;'>+0.4%</span>  |оставляем в качестве признака |
|Использование как моно- так и биграмм TfidfVectorizer(ngram_range=(1, 2) | <span style='color: red;'>-19%</span> | |
|Использование только биграмм TfidfVectorizer(ngram_range=(2, 2) | <span style='color: red;'>-24%</span> | |
|Использование  моно-, би- и триграмм TfidfVectorizer(ngram_range=(1, 3) | <span style='color: red;'>-21%</span> | |
|Обогащение тренировочного датасета наблюдениями по минорным классам +204 наблюдения (Цветняшки, Буба, Трансформеры, Буба, Финник, Акуленок, Кукутики, Cry babies magic tears, Чудинки, Паровозики Чаггингтон)| <span style='color: green;'>+4%</span>  |оставляем обогащенный датасет |
|Дополнительное обогащение тренировочного датасета наблюдениями по минорным классам | <span style='color: green;'>+2%</span>  |оставляем обогащенный датасет |
|Добавление в параметры именованных сущностей (~8000) | <span style='color: red;'>-0.2%</span>  | |
|Добавление в параметры именованных сущностей с фильтром по длине (~7000) | <span style='color: red;'>-1%</span>  | |
|Undersampling класса 'none' | <span style='color: red;'>-4%</span>  | |
|Использование в качестве классификатора CatBoost | <span style='color: red;'>-20%</span>  | |
|Использование в качестве классификатора нейросетевой модели | <span style='color: red;'>-20%</span>  | |
|Исправление ошибок разметки | <span style='color: red;'>-0.5%</span>  | Хотя на тесте Kaggle метрика чуть проседает, этап обработки оставляем, т.к. на реальных данных качество модели будет выше |
