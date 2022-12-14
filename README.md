# Прогнозирование объемов продаж
> **Note:**  [Ноутбук данного задания в Google Colab](https://colab.research.google.com/drive/1So2pWLAzHiwZz5T09Qp31p7_UQBEF0kK?usp=sharing)

В данном задании мы создадим модель прогнозирования объемов продаж сети быстрого питания, обучив ее на объемах ежедневных продаж полученных в течении  424 дней. Мы создадим прогноз реализации каждого блюда по каждому типу реализации для каждой точки на  следущие 31 день. 

Мы будем использовать библиотеку darts разработанную для работы с временными рядами. Библиотека предоставляет API, позволяющее настроить и натренировать алгоритм машинного обучения для прогнозирования временного ряда.
```python
!pip install darts
```
Начинаем с загрузки имеющегося датасета объемов Объемы.csv и файла Точки.xlsx с дополнительной информацией о точках продаж.

Загрузим датасет объемов продаж в датафрейм: 
```python
import pandas as pd
df = pd.read_csv('Объемы.csv', encoding='cp1251', sep = ';', engine= 'python')
df.columns
```
>Index(['s. Days fact', 'Точки', 'Тип реализации', 'Блюда', 'Объёмы'], dtype='object')

Взглянем на фрагмент данных: 
```python
df.iloc[0:46,0:5]
```
<sup>
	 <p><b> s. Days fact	Точки	Тип реализации	Блюда	Объёмы</b></p> 
<p> 0	1 Jan 21	#1	Где взял - там съел	Картошка без крахмала	317</p>  
<p> 1	1 Jan 21	#2	Где взял - там съел	Картошка без крахмала	291</p>  
<p> 2	1 Jan 21	#3	Где взял - там съел	Картошка без крахмала	326</p>
<p> 3	1 Jan 21	#4	Где взял - там съел	Картошка без крахмала	328</p>
<p> 4	1 Jan 21	#5	Где взял - там съел	Картошка без крахмала	273</p>
<p> 5	1 Jan 21	#6	Где взял - там съел	Картошка без крахмала	394</p>
<p> 6	1 Jan 21	#7	Где взял - там съел	Картошка без крахмала	0</p>
<p> 7	1 Jan 21	#8	Где взял - там съел	Картошка без крахмала	315</p>
<p> 8	1 Jan 21	#9	Где взял - там съел	Картошка без крахмала	469</p>
<p> 9	1 Jan 21	#10	Где взял - там съел	Картошка без крахмала	357</p>
<p> 10	1 Jan 21	#11	Где взял - там съел	Картошка без крахмала	427</p>
<p> 11	1 Jan 21	#12	Где взял - там съел	Картошка без крахмала	381</p>
<p> 12	1 Jan 21	#13	Где взял - там съел	Картошка без крахмала	303</p>
<p> 13	1 Jan 21	#14	Где взял - там съел	Картошка без крахмала	268</p>
<p> 14	1 Jan 21	#15	Где взял - там съел	Картошка без крахмала	248</p>
<p> 15	1 Jan 21	#16	Где взял - там съел	Картошка без крахмала	0</p>
<p> 16	1 Jan 21	#17	Где взял - там съел	Картошка без крахмала	270</p>
<p> 17	1 Jan 21	#18	Где взял - там съел	Картошка без крахмала	240</p>
<p> 18	1 Jan 21	#19	Где взял - там съел	Картошка без крахмала	217</p>
<p> 19	1 Jan 21	#20	Где взял - там съел	Картошка без крахмала	237</p>
<p> 20	1 Jan 21	#21	Где взял - там съел	Картошка без крахмала	354</p>
<p> 21	1 Jan 21	#1	Бандеролью	Картошка без крахмала	254</p>
<p> 22	1 Jan 21	#2	Бандеролью	Картошка без крахмала	177</p>
<p> 23	1 Jan 21	#3	Бандеролью	Картошка без крахмала	333</p>
<p> 24	1 Jan 21	#4	Бандеролью	Картошка без крахмала	325</p>
<p> 25	1 Jan 21	#5	Бандеролью	Картошка без крахмала	346</p>
<p> 26	1 Jan 21	#6	Бандеролью	Картошка без крахмала	454</p>
<p> 27	1 Jan 21	#7	Бандеролью	Картошка без крахмала	0</p>
<p> 28	1 Jan 21	#8	Бандеролью	Картошка без крахмала	390</p>
<p> 29	1 Jan 21	#9	Бандеролью	Картошка без крахмала	355</p>
<p> 30	1 Jan 21	#10	Бандеролью	Картошка без крахмала	199</p>
<p> 31	1 Jan 21	#11	Бандеролью	Картошка без крахмала	472</p>
<p> 32	1 Jan 21	#12	Бандеролью	Картошка без крахмала	216</p>
<p> 33	1 Jan 21	#13	Бандеролью	Картошка без крахмала	354</p>
<p> 34	1 Jan 21	#14	Бандеролью	Картошка без крахмала	356</p>
<p> 35	1 Jan 21	#15	Бандеролью	Картошка без крахмала	283</p>
<p> 36	1 Jan 21	#16	Бандеролью	Картошка без крахмала	0</p>
<p> 37	1 Jan 21	#17	Бандеролью	Картошка без крахмала	415</p>
<p> 38	1 Jan 21	#18	Бандеролью	Картошка без крахмала	457</p>
<p> 39	1 Jan 21	#19	Бандеролью	Картошка без крахмала	213</p>
<p> 40	1 Jan 21	#20	Бандеролью	Картошка без крахмала	204</p>
<p> 41	1 Jan 21	#21	Бандеролью	Картошка без крахмала	302</p>
<p> 42	1 Jan 21	#1	На коне	Картошка без крахмала	0</p>
<p> 43	1 Jan 21	#2	На коне	Картошка без крахмала	0</p>
<p> 44	1 Jan 21	#3	На коне	Картошка без крахмала	0</p>
<p> 45	1 Jan 21	#4	На коне	Картошка без крахмала	238</p>
</sup>
Как видно из вышеприведенного фрагмента, данные отсортированы по дням/точкам/типам реализации/блюдам. То есть, соседние строки в большинстве случаев отличаются друг от друга на один из вышеупомянутых признаков. Так же можно заметить, что некоторые значения колонки Объёмы равны нулю. Мы используем оба эти наблюдения позже в ходе подготовки данных для использования в модели прогнозирования.

Теперь просмотрим дополнительный файл с информацией о точках: 
```python
df_places = pd.read_excel('Точки.xlsx', index_col=0, engine='openpyxl')
df_places.reset_index(drop=True)
```
<sup>
	<p> <b>Item Name	Display Name	Круглосуточно?	Авто?	Дата открытия	Дата закрытия</b></p>
<p> 0	Все точки	NaN	NaN	NaN	NaN	NaN
<p> 1	#1	Нижне-Волжская набережная, 19	True	False	06.08.2015	NaN </p> 
<p> 2	#2	площадь Максима Горького, 1	False	False	12.09.2019	NaN </p>
<p> 3	#3	площадь Революции, 5	True	False	16.04.2004	10.03.2022 </p>
<p> 4	#4	Веденяпина, 2Б	True	True	25.07.2016	NaN </p>
<p> 5	#5	Керченская, 13	True	True	30.09.2006	NaN </p>
<p> 6	#6	Космонавта Комарова, 36	True	True	23.11.2018	NaN </p>
<p> 7	#7	Маршала Рокоссовского К.К., 19	False	False	17.09.2021	NaN </p>
<p> 8	#8	проспект Ленина, 10	True	True	04.07.2018	NaN </p>
<p> 9	#9	Советская площадь, 5	True	True	28.11.1998	NaN </p>
<p> 10	#10	Коминтерна, 162	False	False	09.10.2020	NaN </p>
<p> 11	#11	Московское шоссе, 126	True	True	26.04.2013	NaN </p>
<p> 12	#12	Родионова, 163	False	True	02.11.2010	NaN
<p> 13	#13	Большая Покровская, 76	False	False	10.01.2008	13.05.2021 </p>
<p> 14	#14	Московское шоссе, 77	False	False	14.03.2012	NaN </p>
<p> 15	#15	Казанское шоссе, 6в	True	False	16.05.2007	NaN </p>
<p> 16	#16	Казанское шоссе, 1	False	False	24.06.2021	NaN </p>
<p> 17	#17	Родионова, 189	True	True	13.03.2011	NaN </p>
<p> 18	#18	проспект Гагарина, 208	True	True	05.09.1996	NaN </p>
<p> 19	#19	Волжская набережная, 26	False	False	24.02.2006	NaN </p>
<p> 20	#20	ул Плотникова, дом 3	False	True	07.04.2012	NaN </p>
<p> 21	#21	ш Южное, дом 12А	True	True	26.09.2009	NaN </p>
</sup>

Как видно, файл содержит много полезной для анализа информации, например: обслуживает ли точка автомобилистов. Но в целях подготовки данных для предсказательной модели нас будет интересовать только информация о точках закрытых до марта 22-го года, чтобы исключить их из последующих расчетов. Такой точкой будет только точка №13 закрытая 13.05.2021. Возвращаясь к Объемам, удалим все строки связанные с этой точкой:
```python
df = df[(df['Точки']!='#13')]
```
Теперь взглянем сколько уникальных элементов содержит каждый признак:
```python
print("Кол-во строк во временном ряду (кол-во дней): ", len(df['s. Days fact'].unique()))
print("Кол-во точек:", len(df['Точки'].unique())) 
print("Кол-во типов реализации: ",len(df['Тип реализации'].unique()))  
print("Кол-во блюд: ", len(df['Блюда'].unique()))
```
> Кол-во строк во временном ряду (кол-во дней):  424  
> Кол-во точек: 20  
> Кол-во типов реализации:  3  
> Кол-во блюд:  6  

Перемножив вышеприведенные числа, ожидаемо получаем кол-во строк в датасете (после удаления точки #13).

>152640

Если исключить кол-во строк (кол-во дней) из умножения, получим 360 - кол-во комбинаций (точка/тип реализации/блюдо). И для каждой комбинации мы имеем значение целевой переменной (объем) на каждый день. Наша задача создать предсказательную модель и обучить её на имеющемся временном ряде из 424 дней, чтобы спрогнозировать объемы на следующие 31 день для каждой из 360 комбинаций (точка/тип реализации/блюдо).

Для этого нам нужно преобразовать имеющийся датасет, подготовив его для обработки моделью. В частности, в рамках такой подготовки, необходимо сделать решейпинг. 

Тут есть несколько вариантов. Один из них - это вытянуть данные относящиеся к одному конкретному дню в одну строку, то есть преобразовать весь датасет в 424 строки (по количеству дней). Этот вариант хорош тем, что позволяет наглядно увидеть присутствие во временном ряду, так называемых статических со-переменных (static covariates), то есть тех переменных значения которых в данной позиции не будут меняться от строки к строке временного ряда. В данном случае это будут идентификаторы комбинации: точка/тип реализации/блюдо. Такие данные по конкретной  комбинации будут расположены в одних и тех же позициях в любой строке и не меняться при переходе к следующей. 

Давайте проведем это преобразование.

Начнем с того, что переведем строковые значения дат в s. Days fact колонке в даты. Это понадобится для корректной сортировки, которая будет осуществлена после группировки по этой колонке.
```python
import datetime as dt
df['s. Days fact'] = df.apply(lambda row: dt.datetime.strptime(row['s. Days fact'], '%d %b %y'), axis=1)
```
Проведем группировку по колонке s. Days fact и затем "флатенизацию", т.е. преобразуем все строки в группе в одну строку, чтобы получить единственную строку для каждой даты:
```python
cc = df.groupby(['s. Days fact']).cumcount() + 1
df_ts = df.set_index(['s. Days fact', cc]).unstack().sort_index(1, level=1)
df_ts.columns = ['_'.join(map(str,i)) for i in df_ts.columns]
df_ts = df_ts.reset_index() 
df_ts
```
<sup>
<p><b>	s. Days fact	Блюда_1	Объёмы_1	Тип реа..._1	Точки_1	...	Блюда_360	Объёмы_360	Тип ре..._360	Точки_360</b></p>
<p> 0	2021-01-01	Картошка без крахмала	317	Где взял - там съел	#1 ...	Холодец с кровью	233	На коне	#21 </p>
<p> 1	2021-01-02	Картошка без крахмала	397	Где взял - там съел	#1 ...	Холодец с кровью	291	На коне	#21 </p>
<p> 	2021-01-03	Картошка без крахмала	424	Где взял - там съел	#1 ...	Холодец с кровью	309	На коне	#21 </p>
<p> 3	2021-01-04	Картошка без крахмала	266	Где взял - там съел	#1 ...	Холодец с кровью	193	На коне	#21 </p>
<p> 4	2021-01-05	Картошка без крахмала	240	Где взял - там съел	#1 ...	Холодец с кровью	173	На коне	#21 </p>
<p> ...	...	...	...	...	...	...	... </p>
<p> 419	2022-02-24	Картошка без крахмала	458	Где взял - там съел	#1 ...	 Холодец с кровью	150	На коне	#21 </p>
<p> 420	2022-02-25	Картошка без крахмала	658	Где взял - там съел	#1 ...		Холодец с кровью	231	На коне	#21 </p>
<p> 421	2022-02-26	Картошка без крахмала	826	Где взял - там съел	#1 ...	Холодец с кровью	281	На коне	#21 </p>
<p> 422	2022-02-27	Картошка без крахмала	886	Где взял - там съел	#1 ...	Холодец с кровью	301	На коне	#21 </p>
<p> 423	2022-02-28	Картошка без крахмала	554	Где взял - там съел	#1 ...	Холодец с кровью	181	На коне	#21 </p>
<p> 424 rows × 1441 columns </p>
</sup>
Присмотревшись к колонкам Блюда_x, Тип реализации_x и Точки_x,  можно заметить что значения в этих колонках не меняются от первой строки до последней, то есть эти колонки содержат статические переменные, как упоминалось ранее. Вспомните также, что мы говорили о том, что соседние группы (точка/тип реализации/блюдо) будут отличаться только одним элементом друг от друга в большинстве строк, и поскольку соотвествующие им значения объемов  будут соседствовать в такой же последовательности в каждой строке, то использование колонок со статическими переменными (точка/тип реализации/блюдо) становится излишним, с точки зрения модели машинного обучения. Другими словами мы можем их просто проигнорировать.

Мы отделим статические данные от нашего временного ряда, оставив только объемы (целевая переменная, которая меняется от строки к строке): 
```python
p_c = 'Точки'
t_c = 'Тип реализации'
d_c = 'Блюда'
df_target = df_ts.loc[:,[(d_c not in i) and (t_c not in i) and (p_c not in i) for i in df_ts.columns]]
df_target 
```
 <sup>
 <p>	s. Days fact	Объёмы_1	Объёмы_2 ...	Объёмы_359	Объёмы_360 </p>
<p> 0	2021-01-01	317	291	...	197	233 </p>
<p> 1	2021-01-02	397	365	...	246	291 </p>
<p> 2	2021-01-03	424	390	...	262	309 </p>
<p> 3	2021-01-04	266	245	...	164	193 </p>
<p> 4	2021-01-05	240	221	...	147	173 </p>
<p> ...	...	...	...	...	... </p>
<p> 419	2022-02-24	458	441	...	144	150 </p>
<p> 420	2022-02-25	658	617	...	210	231 </p>
<p> 421	2022-02-26	826	735	...	269	281 </p>
<p> 422	2022-02-27	886	795	...	281	301 </p>
<p> 423	2022-02-28	554	515	...	175	181 </p>
<p> 424 rows × 361 columns </p>
</sup>
И создадим временной ряд со-переменных (пока пустой):  

```python
df_covar = df_ts.loc[:,['s. Days fact' in i for i in df_ts.columns]]
df_covar  
```  

>s. Days fact  
>0	2021-01-01  
>1	2021-01-02  
>2	2021-01-03  
>3	2021-01-04  
>4	2021-01-05  
> ...	...  
>419	2022-02-24  
>420	2022-02-25  
>421	2022-02-26  
>422	2022-02-27  
>423	2022-02-28  
>424 rows × 1 columns

Как упоминалось, ряд с со-переменными содержащими статические данные, которые не меняются от строки к строке, не очень полезен для модели прогнозирования. А вот что будет меняться и может улучшить предиктивные способности создаваемой модели - это день недели. Создаем колонку с днями недели:  
```python
df_covar['dw'] = df_covar.apply(lambda row: row['s. Days fact'].weekday(), axis=1) 
dw_column = df_covar.pop('dw')
df_covar.insert(1, 'dw', dw_column)
df_covar  
```  
>	s. Days fact	dw  
>0	2021-01-01	4  
>1	2021-01-02	5  
>2	2021-01-03	6  
>3	2021-01-04	0  
>4	2021-01-05	1  
>...	...	...  
>419	2022-02-24	3  
>420	2022-02-25	4  
>421	2022-02-26	5  
>422	2022-02-27	6  
>423	2022-02-28	0  
>424 rows × 2 columns

Подготавливая данные для дальнейшей обработки, преобразуем s. Days fact колонку в индекс в обоих рядах.
```python
df_target.set_index('s. Days fact', inplace=True)
df_target  
```
<sup>
<p>	Объёмы_1	Объёмы_2	...	Объёмы_359	Объёмы_360 </p>
<p>s. Days fact </p>																					
<p>2021-01-01	317	291	...	197	233 </p>
<p>2021-01-02	397	365	...	246	291 </p>
<p>2021-01-03	424	390	...	262	309 </p>
<p>2021-01-04	266	245	...	164	193 </p>
<p>2021-01-05	240	221	...	147	173 </p>
<p>...	...	...		...	...	... </p>
<p>2022-02-24	458	441	...	144	150 </p>
<p>2022-02-25	658	617	...	210	231 </p>
<p>2022-02-26	826	735	...	269	281 </p>
<p>2022-02-27	886	795	...	281	301 </p>
<p>2022-02-28	554	515	...	175	181 </p>
<p>424 rows × 360 columns </p>
</sup>

Проверим есть ли такие комбинации (точка\тип\блюдо) объемы которых равны нулю на всем промежутке наблюдения.
```python
cols = df_target.columns.values.tolist() 
empty_sales = [col for col in cols if df_target[col].mean() == 0]
non_empty_sales = [col for col in cols if df_target[col].mean() != 0]
```
Проверим сумму всех ненулевых и нулевых колонок (должно быть 360)
```python
print(len(non_empty_sales))
print(len(empty_sales))
```
>306
>54

Удалим все нулевые колонки из набора для обучения.
```python
df_target = df_target[non_empty_sales]
```
Преобразуем все встречающиеся нули в общем ненулевых колонках на среднее значение по колонке.
```python
for col in non_empty_sales:
  df_target[col].replace(to_replace = 0, value = int(df_target[col].mean()), inplace=True)
```
Теперь преобразуем s. Days fact колонку в индекс в ряде сопеременных:
```python
df_covar.set_index('s. Days fact', inplace=True)
df_covar
```  
>	dw
>s. Days fact    
>2021-01-01	4  
>2021-01-02	5  
>2021-01-03	6  
>2021-01-04	0  
>2021-01-05	1  
>...	...  
>2022-02-24	3  
>2022-02-25	4  
>2022-02-26	5  
>2022-02-27	6  
>2022-02-28	0  
>424 rows × 1 columns

Удлиним ряд со-переменных до 31 марта 2022. Мы легко можем это сделать, потому что дни недели известны наперед (это пример использования так называемых будущих со-переменных (future covariates)):
```python
df_covar_march = pd.DataFrame()
df_covar_march['s. Days fact'] = pd.date_range(start="2022-03-01",end="2022-03-31")
df_covar_march['dw'] = df_covar_march.apply(lambda row: row['s. Days fact'].weekday(), axis=1) 
df_covar_march.set_index('s. Days fact', inplace=True)
df_covar_full = pd.concat([df_covar, df_covar_march])
```
К данному моменту мы в целом готовы перейти к созданию модели. Для этого будем использовать библиотеку darts специально разработанную для работы с временными рядами. 

Сперва, преобразуем наши временные ряды в формат darts библиотеки:
```python
from darts import TimeSeries
df_ts_target = TimeSeries.from_dataframe(df_target)
df_ts_covar = TimeSeries.from_series(df_covar)
df_ts_covar_full = TimeSeries.from_series(df_covar_full)
```
Добавим еще несколько со-переменных к ряду, а именно год и месяц:
```python
from darts.utils.timeseries_generation import datetime_attribute_timeseries
covar_year = datetime_attribute_timeseries(df_ts_covar, attribute="year")
covar_month = datetime_attribute_timeseries(df_ts_covar, attribute="month")
df_ts_covar = covar_year.stack(covar_month).stack(df_ts_covar)
df_ts_covar
```
>array([[[2.021e+03],
>        [1.000e+00],
>        [4.000e+00]],
>
>       [[2.021e+03],
>        [1.000e+00],
>        [5.000e+00]],

>       [[2.021e+03],
>        [1.000e+00],
>        [6.000e+00]],

>       ...,

>       [[2.022e+03],
>        [2.000e+00],
>        [5.000e+00]],

>       [[2.022e+03],
>        [2.000e+00],
>        [6.000e+00]],

>       [[2.022e+03],
>        [2.000e+00],
>        [0.000e+00]]])
> Coordinates:  
>  s. Days fact (s. Days fact) datetime64[ns] 2021-01-01 ... 2022-02-28  
>  component    (component)    object         'year' 'month' 'dw'  
>Attributes:  
>  static_covariates :None  
>  hierarchy :None

Проведем масштабирование данных, чтобы сделать их более "удобоваримыми" для модели.
```python
from darts.dataprocessing.transformers import Scaler
scaler_df_ts_target = Scaler()
df_ts_target = scaler_df_ts_target.fit_transform(df_ts_target)
scaler_df_ts_covar = Scaler()
df_ts_covar = scaler_df_ts_covar.fit_transform(df_ts_covar)
scaler_df_ts_covar_full = Scaler()
df_ts_covar_full = scaler_df_ts_covar_full.fit_transform(df_ts_covar_full)
```
Разделим наши ряды на train/validation сеты (в качестве валидационного сета будем использовать данные за февраль, 2022):
```python
df_ts_target_train, df_ts_target_val = df_ts_target[:-28], df_ts_target[-28:]  
df_ts_covar_train, df_ts_covar_val = df_ts_covar[:-28], df_ts_covar[-28:]
```
Создадим модель:
```python
#from darts.models import TransformerModel
from darts.models import BlockRNNModel
model_cov = BlockRNNModel(
    model="LSTM",
    input_chunk_length=60,
    output_chunk_length=30,
    n_epochs=300,
    random_state=0,
)
```
И натренируем ее на наших данных:
```python
model_cov.fit(
    series=df_ts_target_train,
    past_covariates=df_ts_covar_train,
    verbose=True,
)
```
Используя модель, сделаем предсказание на февраль, 2022:
```python
pred_cov = model_cov.predict(n=28, series = df_ts_target_train, past_covariates=df_ts_covar)
```
Размаштабируем данные:
```python
df_ts_target_val = scaler_df_ts_target.inverse_transform(df_ts_target_val)
pred_cov = scaler_df_ts_target.inverse_transform(pred_cov)
```
Оценим как отработала модель, используя метрику MAPE
```python
from darts.metrics import mape
print("MAPE = {:.2f}%".format(mape(df_ts_target_val, pred_cov, n_jobs=-1)))
```
>MAPE = 16.99%

Интересно, что предсказание на первые 10 дней получается значительно точнее, чем на месяц.
```python
print("MAPE = {:.2f}%".format(mape(df_ts_target_val[0:10], pred_cov[0:10], n_jobs=-1)))
```
>MAPE = 9.73%

Сделаем график для первой комбинации (#1/Где взял - там съел/Картошка без крахмала) на февраль.
```python
import matplotlib.pyplot as plt
actual = df_ts_target_val.all_values()
actual = actual.reshape(28,306)
actual = TimeSeries.from_values(actual[0:28,0:1])
pred = pred_cov.all_values()
pred = pred.reshape(28,306)
pred= TimeSeries.from_values(pred[0:28,0:1])
actual.plot(label="actual")
pred.plot(label="forecast")
plt.legend()
```
<img src="https://github.com/nlptechbook/MultivariateTimeseriesForecast/blob/main/forecast.png" width="500px"/>

Теперь сделаем предсказание на март. Начнем с создания модели:
```python
model_cov_march = BlockRNNModel(
    model="LSTM",
    input_chunk_length=60,
    output_chunk_length=31,
    n_epochs=300,
    random_state=0,
)
model_cov_march.fit(
    series=df_ts_target,
    past_covariates=df_ts_covar_full,
    verbose=True,
)
```
Делаем предсказание:
```python
pred_cov_march = model_cov_march.predict(n=31, series = df_ts_target, past_covariates=df_ts_covar_full)
```
Размасштабируем данные:
```python
pred_cov_march = scaler_df_ts_target.inverse_transform(pred_cov_march)
pred_cov_march
```
>array([[[711.61058127],
>        [653.68262344],
>        [384.90356626],
>        ...,
>        [410.56567283],
>        [213.02886271],
>        [222.78182118]],

>       [[683.17531034],
>        [643.56500992],
>        [447.96427608],
>        ...,
>        [444.43744744],
>        [222.4060234 ],
>        [220.91086781]],

>       [[624.21628358],
>        [666.82765411],
>        [369.64701814],
>        ...,
>...
>        ...,
>        [360.45995857],
>        [179.94649059],
>        [200.05689598]],

>       [[624.29059966],
>        [681.59253186],
>        [373.26502243],
>        ...,
>        [333.79427529],
>        [206.03282291],
>        [222.8414366 ]],

>       [[462.38989884],
>        [553.86677562],
>        [245.18340393],
>        ...,
>        [288.03913161],
>        [151.29370079],
>        [154.69513403]]])

Преобразуем данные в массив и удалим ненужное измерение:
```python
pred_cov_march.all_values().shape
```
>(31, 306, 1)
```python
pred_march = pred_cov_march.all_values()
pred_march = pred_march.reshape(31,306)
```
Приступим к созданию датафрейма из нащих данных. Для этого создадим и затем соединим два датафрейма: первый с предсказанными данными, второй с нулевыми колонками :
```python
df_pred = pd. DataFrame(pred_march, columns=non_empty_sales, index = pred_cov_march.time_index ) 
len(df_pred.columns)
```
>306
```python
import numpy as np
arr = np.zeros([31,54])
df_empty = pd.DataFrame(arr, columns=empty_sales, index = pred_cov_march.time_index)
len(df_empty.columns)
```
>54
```python
df_march = pd.concat([df_pred, df_empty], axis = 1)
len(df_march.columns)
```
>360

Перестроим столбцы в первоначальном порядке:
```python
df_march = df_march[cols]
df_march = df_march.reset_index(level=0)
```
Теперь приведем датафрейм в формат исходного датасета. Начнем с того, что переформатируем датафрейм, чтобы каждый объем был в отдельной строке. Делаем флатенизацию и сортировку в нужном порядке:
```python
df_flat = df_march.reset_index()
df_flat = pd.lreshape(df_flat, {'Объёмы' : cols})
df_flat = df_flat.sort_values(['s. Days fact', 'index'])
df_flat = df_flat.drop(['index'], axis=1).reset_index(drop=True)
```
Восстанавливаем колонки как в исходном датасете:
```python
df_2 = df.reset_index(drop=True)
df_flat = df_flat.assign(Точки = lambda x: (df_2.iloc[x.index]['Точки']))
df_flat = df_flat.assign(Типреализации = lambda x: (df_2.iloc[x.index]['Тип реализации']))
df_flat = df_flat.assign(Блюда = lambda x: (df_2.iloc[x.index]['Блюда']))
df_flat = df_flat.rename(columns = {'Типреализации':'Тип реализации'})
df_flat.columns.tolist()
s_column = df_flat.pop('Объёмы')
df_flat.insert(4, 'Объёмы', s_column)
df_flat = df_flat.rename(columns = {'Объёмы': 'Прогнозные объёмы', 's. Days fact': 's. Days forecast'})
df_flat = df_flat.astype({'Прогнозные объёмы':'int'})
df_flat['s. Days forecast'] = df_flat.apply(lambda row: dt.datetime.strftime(row['s. Days forecast'], '%d %b %y'), axis=1)
df_flat
```
<sup>
<p>	s. Days forecast	Точки	Тип реализации	Блюда		Прогнозные объёмы </p>
<p>0	01 Mar 22	#1	Где взял - там съел	Картошка без крахмала		711</p>
<p>1	01 Mar 22	#2	Где взял - там съел	Картошка без крахмала		653</p>
<p>2	01 Mar 22	#3	Где взял - там съел	Картошка без крахмала		384</p>
<p>3	01 Mar 22	#4	Где взял - там съел	Картошка без крахмала		681</p>
<p>4	01 Mar 22	#5	Где взял - там съел	Картошка без крахмала		602</p>
<p>...	...	...	...	...	...	...</p>
<p>11155	31 Mar 22	#17	На коне	Холодец с кровью		183</p>
<p>11156	31 Mar 22	#18	На коне	Холодец с кровью		288</p>
<p>11157	31 Mar 22	#19	На коне	Холодец с кровью		0</p>
<p>11158	31 Mar 22	#20	На коне	Холодец с кровью		151</p>
<p>11159	31 Mar 22	#21	На коне	Холодец с кровью		154</p>
<p>11160 rows × 6 columns</p>
</sup>

```python
df_flat.to_csv('March.csv', encoding='cp1251', sep = ';')
```
