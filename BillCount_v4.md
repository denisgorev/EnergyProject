

```python
import pandas as pd
import os
import re
import numpy as np
```


```python
total_sum = 0 #переменная для вычисления общей суммы
value_massive = [] #массив, в котором будут храниться сумма для каждого пользователя по месяцам (фрейм месяц == элементу массива)
i = 0 #счетчик файлов
value = pd.DataFrame()

def bill_count(df): #функция подготовки данных
    df.drop(df.columns[df.columns.str.contains('Unnamed')], axis = 1, inplace = True) #удаляем лишний столбец
    df['KWT'] = df['KWT'].str.replace('_', '') #убираем лишние артефакты из KWT
    df['KWT']=df['KWT'].astype(float)  #преобразовываем строчный формат KWT в число с плавающей точкой
    pattern = r'\w+ (\d+):\d+' #паттерн, чтобы найти время
    df['HOUR'] = df['TIME'].str.findall(pattern).str[0].astype(int) #добавление новой колонки HOUR, полученной из TIME
    df_price = pd.read_excel('Prices.xlsx', header = 1) #загружаем данные по ценам
    df_price['HOUR'] = df_price['HOUR'] - 1 
    df = pd.merge(df, df_price, on = 'HOUR') #мерджим основную таблицу и таблицу с ценами по столбцу HOUR (ставим в соответствие)
    df_priv = pd.read_csv('HAS_PRIVELEGE.csv', index_col=0) # считываем инфо по привилегиям
    df = pd.merge(df, df_priv, on = 'USER') #мерджим основную таблицу и привилегии
    df['PRIVELEGE'] = df['PRIVELEGE'].replace(1, 0.85) #для удобства расчета меняем значения, где есть привилегия на 85%
    df['PRIVELEGE'] = df['PRIVELEGE'].replace(0, 1) #для удобства расчета меняем значения, где нет привилегии на 100%
    df['SUBTOTAL_BILL'] = df['KWT'] * df['PRICE'] * df['PRIVELEGE'] #считаем значения по каждой записи в месяце
    return df

pattern = r'\w+\d+\.csv' #regex, выбираем только файлы, где инфа по месяцам
for line in os.listdir(): #проходим по всем элементам в папке
    if re.findall(pattern, line.strip()): #отбираем только с нужным названием
        df = pd.read_csv(line) #загрузка файла
        df = bill_count(df) #вызываем функцию расчета и передаем туда полученный из файла фрейм
        df = df.groupby('USER')['SUBTOTAL_BILL'].sum() #группируем по айдишнику пользователя и считаем сумму
        df = pd.DataFrame(df) #массив переводим в фрейм
        #value = pd.concat(value, df)
        value_massive.append(df) #добавляем в массив расчет по месяцу
        i+=1

        
k=0 #вспомогательная переменная для цикла
result = pd.concat(value_massive) #склеиваем
print(result['SUBTOTAL_BILL'].sum())
```

    4368351.0478
    


```python
value_massive2 = []
for line in os.listdir(): #проходим по всем элементам в папке
    if re.findall(pattern, line.strip()): #отбираем только с нужным названием
        df = pd.read_csv(line) #загрузка файла
        df = bill_count(df) #вызываем функцию расчета и передаем туда полученный из файла фрейм
        df['TIME'] = pd.to_datetime(df['TIME'], format ='%d/%m/%y  %H:%M') #переводит в datetime
        value_massive2.append(df) #каждый месяц в массив
        
result2 =  pd.concat(value_massive2) #склеиваем 

max_bill = result2.groupby(['USER', pd.Grouper(key = 'TIME', freq = '1D')])['SUBTOTAL_BILL'].sum().rolling(15).sum() #группируем по 15 дней, считаем сумму чеков
max_bill_value = max_bill.max() #находим самое большое значение
max_bill_id = max_bill.idxmax()[0] #id
max_bill_date = max_bill.idxmax()[1] #дата

max_kwt = result2.groupby(['USER', pd.Grouper(key = 'TIME', freq = '1D')])['KWT'].sum().rolling(15).sum()
max_kwt_value = max_kwt.max()
max_kwt_id = max_kwt.idxmax()[0]
max_kwt_date = max_kwt.idxmax()[1]

result3 = pd.DataFrame({'KWT':[max_kwt_value, max_kwt_id, max_kwt_date], 'BILL':[max_bill_value, max_bill_id, max_bill_date]}, index = ['value','id','time' ])
print(result3)
```

                           KWT                 BILL
    value               138.87              2751.04
    id         49a573d9-f36f-1      49a573ee-f36f-1
    time   2013-01-19 00:00:00  2013-01-19 00:00:00
    
