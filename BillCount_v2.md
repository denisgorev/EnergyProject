

```python
import pandas as pd
import os
import re
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
kwt_lst = [] # массив для kwt
bill_lst = [] # массив для счета
index = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
cols = ['id', 'KWT', 'Date']
cols_bill = ['id', 'BILL', 'Date']
for line in os.listdir(): #проходим по всем элементам в папке
    if re.findall(pattern, line.strip()): #отбираем только с нужным названием
        df = pd.read_csv(line) #загрузка файла
        df = bill_count(df) #вызываем функцию расчета и передаем туда полученный из файла фрейм
        df['TIME'] = pd.to_datetime(df['TIME'])
        df = df.set_index('TIME') #устанавливаем индекс в TIME, чтобы далее можно было группировать по времени
        df=df.groupby('USER').resample('15D').sum() # группируем по пользователю и по 15-ти дневке
        kwt_value = df['KWT'].max() #находим самое большое значение для KWT
        kwt_index = df['KWT'].idxmax() #находим id и время
        kwt_lst.append([kwt_index[0], kwt_value, kwt_index[1]]) #сохраняем значения в лист
        
        bill_value = df['SUBTOTAL_BILL'].max() #находим самое большое значение для bill
        bill_index = df['SUBTOTAL_BILL'].idxmax() #находим id и время
        bill_lst.append([bill_index[0], bill_value, bill_index[1]]) #сохраняем значения в лист
              
df1 = pd.DataFrame(kwt_lst, columns=cols, index = index) #фрейм для kwt
df2 = pd.DataFrame(bill_lst, columns=cols_bill, index = index) #фрейм для счета
print(df1)
print()
print(df2)
```

                      id      KWT       Date
    Jan  49a57420-f36f-1  126.976 2013-01-16
    Feb  49a573f5-f36f-1  113.060 2013-02-01
    Mar  49a573d4-f36f-1  118.084 2013-03-16
    Apr  49a573d1-f36f-1  111.943 2013-04-01
    May  49a57406-f36f-1  110.008 2013-05-01
    Jun  49a573d8-f36f-1  106.412 2013-06-16
    Jul  49a5740f-f36f-1  114.602 2013-07-16
    Aug  49a5740c-f36f-1  115.751 2013-08-01
    Sep  49a57407-f36f-1  111.913 2013-09-01
    Oct  49a5740c-f36f-1  116.548 2013-10-01
    Nov  49a573d3-f36f-1  107.843 2013-11-16
    Dec  49a573f8-f36f-1  120.175 2013-12-16
    
                      id      BILL       Date
    Jan  49a573ee-f36f-1  2624.284 2013-01-16
    Feb  49a573f5-f36f-1  2297.992 2013-02-01
    Mar  49a5741a-f36f-1  2354.292 2013-03-01
    Apr  49a573da-f36f-1  2260.236 2013-04-01
    May  49a573ef-f36f-1  2249.512 2013-05-01
    Jun  49a573f8-f36f-1  2129.416 2013-06-16
    Jul  49a57402-f36f-1  2315.660 2013-07-01
    Aug  49a57411-f36f-1  2308.800 2013-08-01
    Sep  49a57407-f36f-1  2342.708 2013-09-01
    Oct  49a5742a-f36f-1  2330.456 2013-10-01
    Nov  49a573d3-f36f-1  2178.996 2013-11-16
    Dec  49a573f8-f36f-1  2460.524 2013-12-16
    
