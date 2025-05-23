# Анализ сообществ в VK с помощью Python

Данное исследование направлено на анализ пользователей и комментариев в сообществах, связанных с ЗОЖ (здоровым образом жизни), веганством и смежными темами, с использованием языка программирования Python с целью понять и проанализировать взаимодействия пользователей в таких сообществах и выявить паттерны, тренды и важные аспекты, связанные с этими темами.

Для достижения этой цели мы соберем данные о пользователях и комментариях из выбранных сообществ. Используя язык программирования Python и соответствующие инструменты, мы разработаем скрипты для извлечения, обработки и анализа данных.

Анализировать данные пользователей и комментариев позволит нам получить информацию о влиянии сообществ на предпочтения и поведение пользователей, а также выявить наиболее обсуждаемые темы и настроения вокруг ЗОЖ и веганства.

В данной работе мы намерены представить подробный анализ, основанный на данных, собранных и обработанных с использованием Python. Результаты этого исследования могут быть полезными для различных заинтересованных сторон, стремящихся понять и взаимодействовать с сообществами, связанными с ЗОЖ и веганством.

## 1. Выбор сообществ

Выберем 3 сообщества в социальной сети ВК, на основе которых будем проводить исследование.
Интересуемая область: ЗОЖ, ПП, веганство
- Здоровье ВКонтакте:	https://vk.com/vkhealth
- Секреты здоровья:	https://vk.com/sekreti_zdorovya
- Вегетарианцы и веганы:	https://vk.com/vegetarians

## 2. Анализ контента группы по направлениям

В данном разделе проанализируем “Мешок слов” – таблицу, где самые распространенные слова, встречающиеся в постах, сопоставлены с частотой их появления.
Было создано по одному “Мешку слов” на каждое сообщество.

```python
import requests
from tqdm.notebook import tqdm # библиотека для того, чтобы показывало прогрузку (зелененькая полоска)
import time # библиотека для того, чтобы сделать задержку при парсинге
from collections import Counter # библиотека для того, чтобы подсчитать что-то
import statistics
import pandas as pd
import nltk # библиотека для разбивки по слову
!pip install pymorphy2
import pymorphy2 # библиотека для замены падежей

token = 'x'
url = 'https://api.vk.com/method/wall.get'
elements = {'access_token':token, 'v':'5.131', 'domain':'viktortorino', 'count':100, 'offset':0}
inf = requests.get(url,params = elements).json()

post_text = []
for x in tqdm(range(0, 401, 100)):
    elements['offset'] = x
    inf = requests.get(url,params = elements).json()
    for post in inf['response']['items']:
        post_text.append(post['text'])

post_text[0]

text = post_text[0] # ввели переменную и ниже разделили предложение по частям, чтобы было удобно работать
text.split(' ')

post_list = ['NOUN', 'ADJF', 'VERB']

text_tokens_1 = nltk.word_tokenize(text) #!!!ОШИБКА!!!

text_tokens_1

morph = pymorphy2.MorphAnalyzer()
morph.parse('ваши')

p = morph.parse('поездку')[0]
p.tag

post_text = []
for x in tqdm(range(0, 401, 100)):
    elements['offset'] = x
    inf = requests.get(url,params = elements).json()
    for post in inf['response']['items']:
        post_text.append(post['text'])
text = post_text[0]
text.split(' ')
morph = pymorphy2.MorphAnalyzer()
morph.parse() # !!!ОШИБКА!!!

text_tokens_1 = nltk.word_tokenize(text)
morph = pymorphy2.MorphAnalyzer()
token_morph = morph.parse('поиграл')[0] #если указываем 0 - 1 форма, кароче ставишь цифорки и смотришь что будет
token_morph

tokens = [] #для текста собранного в постах мы его токинизируем
for text in tqdm(post_text):
    text_tokens_1 = nltk.word_tokenize(text) #для каждого токена в переменной мы берем морф форму этого токена
    for token in text_tokens_1:
        token_morph = morph.parse(token)[0] #после этого мы проверяем правильность подобранной части речи, добавляем еще ступень, в которой отбираются только токены для одной из 3х частей речи, которые мы ввели до этого
        if token_morph.tag.POS in post_list:
            tokens.append(token_morph.normal_form)
print (tokens)

df = pd.DataFrame(dict(Counter(tokens)).items(), columns = ['Токен', 'Частота']) # дикт - словарь, каунтер - будет считать кол-во упоминаний каждого токена - слова

df.to_excel('Mешок.xlsx')
```

### Сообщество “Здоровье ВКонтакте”

Группа "Здоровье ВКонтакте" в большей степени нацелена на проведение розыгрышей и марафонов, посвященных теме здоровья, а также продвижение сервисов экосистемы ВК, а не на распространение информации по теме.
Это видно по частоте слов, используемых в постах: самыми употребляемыми были слова, связанные с темой проведения розыгрышей (марафон, комментарий, розыгрыш, победитель, участник, приз, вызов, конкурс, активность, награда, трансляция, эфир, получить) и продвижения собственных сервисов (сервис, ВКонтакте, шаги).

Распределим топ 100 самых распространенных слов по категориям и найдем суммарную частоту использования слов в каждой категории (таблица 1).

Таблица 1 – Распределение слов по категориям в сообществе “Здоровье ВКонтакте”

| Категория        | Частота       | Удельный вес  |
| :-------------: |:-------------:| :-----:|
| Розыгрыши и марафоны      | 1116 | 58% |
| Продвижение сервисов   | 631     |   33% |
| Информация о здоровье | 176   |    9% |

### Сообщество “Секреты здоровья”
Группа "Секреты здоровья" в большей степени нацелена на распространение информации по теме здоровья.
Это видно по частоте слов, используемых в постах: самыми употребляемыми были слова, связанные с темой здоровья (заболевание, иммунитет, активизировать, организм, человек, полезный, здоровье), правильного питания (калий, мед, напиток, печень, масло), спорта (фитнес, энергия, растяжка) и ментального здоровья (сон, медитация, психолог, энергия)

Распределим топ 100 самых распространенных слов по категориям и найдем суммарную частоту использования слов в каждой категории (таблица 2).

Таблица 2 – Распределение слов по категориям в сообществе “Секреты здоровья”

| Категория        | Частота       | Удельный вес  |
| :-------------: |:-------------:| :-----:|
| Здоровье      | 286 | 31% |
| Правильное питание   | 398     |   43% |
| Спорт | 158   |    17% |
| Ментальное здоровье | 89   |    10% |

### Сообщество “Вегетарианцы и веганы”
Группа "Вегетарианцы и веганы" в большей степени нацелена на распространение информации по теме растительного питания, этичного отношения к животным и заботы о природе.
Это видно по частоте слов, используемых в постах: самыми употребляемыми были слова, связанные с темой растительного питания и исключения животных продуктов (растительный, мясо, питание, веганский, веган, диета, веганство, блюдо) и этичного отношения к природе (животное, исследование, жизнь, вид, выброс, климат, среда, окружающий, животноводство, отношение, глобальный экологический).

Распределим топ 100 самых распространенных слов по категориям и найдем суммарную частоту использования слов в каждой категории (таблица 3).

Таблица 3 – Распределение слов по категориям в сообществе “Секреты здоровья”

| Категория        | Частота       | Удельный вес  |
| :-------------: |:-------------:| :-----:|
| Растительное питание      | 1084 | 35% |
| Этичное отношение к природе и проблемы экологии   | 1981     |   65% |

## 3. Анализ целевой аудитории по полу и возрасту

В данном разделе будет проведен анализ активных комментаторов каждого из сообществ по критерию пол и возраст.
Будут использоваться данные из таблицы “Данные пользователей”, где указаны ФИ, дата рождения и пол комментаторов сообществ.

```python
import requests
from tqdm.notebook import tqdm
import time
from collections import Counter
import statistics
import pandas as pd

token = "X"
url = "https://api.vk.com/method/wall.get"
owner_id = "-124985100"
elements = {'access_token': token, 'v': '5.131','owner_id': owner_id, 'count': 100, 'offset': 0}
inf = requests.get(url, params = elements).json()

id_list = []
for x in inf['response']['items']:
  id_list.append(x['id'])

url = "https://api.vk.com/method/wall.getComments/"
id_commentators = []
for i in tqdm(id_list):
  elements['post_id'] = i
  inf = requests.get(url, params = elements).json()
  time.sleep(0.2)
  if inf['response']['count'] != 0:
    for y in inf['response']['items']:
      id_commentators.append(y['from_id'])

commentators_dict = dict(Counter(id_commentators))

statistics.mean(commentators_dict.values())

crazy_commentators = []
for k,v in commentators_dict.items():
  if v > statistics.mean(commentators_dict.values()):
    crazy_commentators.append(k)

crazy_commentators

elements = {'access_token':'vk1.a.x',
  'v':'5.131',
  'user_ids':crazy_commentators, 'fields': 'name, fname, bdate, city, education, sex, relation'}

url = 'https://api.vk.com/method/users.get/'
inf = requests.get(url, params = elements).json()
name = []
fname = []
bdates = []
cities = []
education = []
genders = []
relation = []
for person in tqdm(crazy_commentators):
  elements['user_ids'] = person
  inf = requests.get(url,params = elements).json()
  time.sleep(0.2)
  try:
    name.append(inf['response'][0]['first_name'])
  except:
    name.append('-')
  try:
    fname.append(inf['response'][0]['last_name'])
  except:
    fname.append('-')
  try:
    bdates.append(inf['response'][0]['bdate'])
  except:
    fname.append('-')
  try:
    cities.append(inf['response'][0]['city']['title'])
  except:
    cities.append('-')
  try:
    education.append(inf['response'][0]['university_name']['title'])
  except:
    genders.append('-')
  try:
    genders.append(inf['response'][0]['sex'])
  except:
    genders.append('-')
  try:
    relation.append(inf['response'][0]['relation'])
  except:
    relation.append('-')

full = list(zip(crazy_commentators,name,fname,bdates,cities,education,genders,relation))
df = pd.DataFrame(full, columns = ['id', 'Имя', 'Фамилия', 'Дата рождения', 'Город', 'Университет', 'Пол', 'Семейное положение'])

df.to_excel('111.xlsx')
```


### Сообщество “Здоровье ВКонтакте”

Анализ ЦА данного сообщества показал, что количество женщин слегка превышает количество мужчин, а самые многочисленные возрастные группы – молодежь и средний возраст (таблица 4)

Таблица 4 – ЦА сообщества “Здоровье ВКонтакте”

| Категория        | Удельный вес           |
| :-------------: |:-------------:|
| Мужчины      | 46% |
| Женщины      | 54%      |
||
| Молодежь (10 – 30 лет) | 41%      |
| Средний возраст (30 – 50 лет) | 40%      |
| Выше среднего возраста (50-80 лет) | 19%      |

### Сообщество “Секреты здоровья”
Анализ ЦА данного сообщества показал, что количество мужчин слегка превышает количество женщин, а самые многочисленная возрастная группа –средний возраст (таблица 5).

Таблица 5 – ЦА сообщества “Секреты здоровья”

| Категория        | Удельный вес           |
| :-------------: |:-------------:|
| Мужчины      | 55% |
| Женщины      | 45%      |
||
| Молодежь (10 – 30 лет) | 34%      |
| Средний возраст (30 – 50 лет) | 53%      |
| Выше среднего возраста (50-80 лет) | 13%      |

### Сообщество “Вегетарианцы и веганы”
Анализ ЦА данного сообщества показал, что количество женщин слегка превышает количество мужчин, а самые многочисленная возрастная группа – молодежь (таблица 6).

Таблица 6 – ЦА сообщества “Секреты здоровья”

| Категория        | Удельный вес           |
| :-------------: |:-------------:|
| Мужчины      | 47% |
| Женщины      | 53%     |
||
| Молодежь (10 – 30 лет) | 48%      |
| Средний возраст (30 – 50 лет) | 38%    |
| Выше среднего возраста (50-80 лет) | 14%      |

## 4. Анализ популярных постов
В данном разделе будет проведен анализ популярных постов, отобранных по критерию количество лайков и комментариев.

```pyhon

import requests
from tqdm.notebook import tqdm
import pandas as pd

token = "x"
url = "https://api.vk.com/method/wall.get"
elements = {'access_token': token, 'v': '5.131','domain': 'moezdorovieclub'}
inf = requests.get(url, params = elements).json()

inf ['response']['items'][0]['text']

all_text = []
for x in inf:
  all_text.extend(x)

count_text = []
count_like = []
count_comments = []
count_post_view = []
for x in inf['response']['items']:
  count_text.append(x["text"])
  count_like.append(x["likes"]['count'])
  count_comments.append(x["comments"]['count'])
print(str(count_like).strip("[]"))

likes = []
post = []
dates = []
comments = []
for i in tqdm(range(0, 3)):
  wall_get = requests.get(url, params = elements).json()
  for y in wall_get['response']['items']:
    post.append(y['text'])
    likes.append(y['likes']['count'])
    comments.append(y['comments']['count'])
    dates.append(y['date'])
    elements['offset'] = i

full_list = list(zip(likes, post, dates, comments))

df = pd.DataFrame(full_list, columns = ['Число лайков', 'Число постов', 'Дата публикации', 'Число комментариев'])
df.to_excel('Данные сообщества.xlsx')

df
```

### Сообщество “Здоровье ВКонтакте”

Анализ постов показал, что самые популярные (или самые раскрученные) посты - розыгрыши техники (iPhone, фены Dyson и т. д.), связанные с продвижением сервисов ВК.Шаги и др. и распространением данного сообщества.

Можно сделать вывод, что данная группа - не что иное, как коммерческий продукт, созданный в целях продвижения сервисов ВК.

В таблице 7 приведена информация о 3 самых популярных постах сообщества.

Таблица 7 – 3 самых популярных постов сообщества “Здоровье ВКонтакте”
| Тема поста        | Количество лайков           |  Количество комментариев  |
| :-------------: |:-------------:|:-------------:|
| Марафон (Шаги ВКонтакте) |1728|2537|
| Розыгрыш (Шаги ВКонтакте) |1343|2132|
| Марафон (Шаги ВКонтакте |1187|1582|

### Сообщество “Секреты здоровья”

Самый популярный контент группы - рецепты, информация о пользе различных продуктов, нутрициологические сводки.

Лайки распределяются равномерно по постам группы (чем старше пост, тем больше лайков). Не наблюдается скачков числа лайков и комментариев под постами с определённым содержанием.

В таблице 8 приведена информация о 3 самых популярных постах сообщества.

Таблица 8 – 3 самых популярных поста сообщества “Секреты здоровья”

| Тема поста        | Количество лайков           |  Количество комментариев  |
| :-------------: |:-------------:|:-------------:|
| Психосоматика |646|34|
| Налаживание режима сна |239|2|
| Полезные добавки в чай |203|3|

### Сообщество “Вегетарианцы и веганы”

Самый популярный контент группы - посты про этическое отношение к животным и экологические проблемы. Самые популярные посты - с информацией про новости муниципалитетов (Урал, Рязань и т.д.) и компаний (Burger King, Крошка Картошка и т.д.) по вопросу экологичности и этичности их деятельности.

В таблице 9 приведена информация о 3 самых популярных постах сообщества.

Таблица 9 – 3 самых популярных поста сообщества “Секреты здоровья”

| Тема поста        | Количество лайков           |  Количество комментариев  |
| :-------------: |:-------------:|:-------------:|
| Спасение козочки на Урале |574|40|
| Дружба поросенка и собаки |314|27|
| Протесты против легализации убийства уличных животных |282|112|

## Системные выводы

**1. Контент групп.** 3 сообщества, хоть и имеют схожую тематику названий, но на деле представляют из себя кардинально различающиеся группы:
- “Здоровье ВКонтакте” ориентировано исключительно на продвижение услуг и сервисов ВК посредством конкурсов, марафонов
- “Секреты здоровья” продвигают научно-популярную информацию из сферы нутрициологии и ЗОЖ с инфографикой, рецептами и советами
- “Вегетарианцы и веганы” направлены на пропаганду уважения к природе и ответственному отношению к жизням животных

**2. ЦА групп**. ЦА 3 сообществ различается незначительно:
- Количество мужчин и женщин примерно равно
- Наименьшая возрастная группа – возраст выше среднего

**3. Самые популярные посты**. Самые популярные посты групп соответствуют их основному направлению:
- “Здоровье ВКонтакте” успешно продвигает свои конкурсы и марафоны, оставляя в тени посты с действительно полезной информацией
- В “Секретах здоровья” популярны посты с рецептами и интересными фактами о ЗОЖ
- Подписчикам “Вегетарианцев и веганов” крайне интересны конфликты с государством и компании по вопросу этичности и экологичности из законодательства, стандартов и деятельности

## ЗАКЛЮЧЕНИЕ

В заключение, данное исследование было направлено на анализ пользователей и комментариев в сообществах, связанных с ЗОЖ и веганством, с использованием языка программирования Python. Целью было понять и проанализировать взаимодействия пользователей в таких сообществах, выявить паттерны, тренды и важные аспекты, связанные с этими темами.

С помощью скриптов, разработанных на Python, были собраны и обработаны данные о пользователях и комментариях из выбранных сообществ. Анализ этих данных позволил получить информацию о влиянии сообществ на предпочтения и поведение пользователей, а также выявить наиболее обсуждаемые темы и настроения вокруг ЗОЖ и веганства.

Результаты этого исследования представляют собой подробный анализ, основанный на данных, собранных и обработанных с использованием Python. Эти результаты могут быть полезными для различных заинтересованных сторон, которые стремятся понять и взаимодействовать с сообществами, связанными с ЗОЖ и веганством.

В дальнейшем, возможно, можно провести более глубокий анализ данных, включающий машинное обучение и анализ сентимента, чтобы получить более детальное представление о реакциях пользователей и динамике обсуждений. Это поможет дополнительно расширить наши познания о ЗОЖ и веганстве, а также предоставит полезную информацию для разработки соответствующих стратегий и подходов в этой области.
