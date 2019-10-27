

```python
import pandas as pd
```


```python
total_sum = 0 #переменная для вычисления общей суммы
value_massive = [] #массив, в котором будут храниться сумма для каждого пользователя по месяцам (фрейм месяц == элементу массива)
i = 1 #начинаем с января

def bill_count(df): #функция подготовки данных
    df.drop(df.columns[df.columns.str.contains('Unnamed')], axis = 1, inplace = True) #удаляем лишний столбец
    df['KWT'] = df['KWT'].str.replace('_', '') #убираем лишние артефакты из KWT
    df['KWT']=df['KWT'].map(lambda x: float(x))  #преобразовываем строчный формат KWT в число с плавающей точкой
    df['HOUR'] = df['TIME'].map(lambda x: pd.to_datetime(x).hour) #добавление новой колонки HOUR, полученной из TIME
    df_price = pd.read_excel('Prices.xlsx', header = 1) #загружаем данные по ценам
    df_price['HOUR'] = df_price['HOUR'].replace(24, 0) #заменяем 24 на 0, для соответствия формата в таблице с ценами
    df = pd.merge(df, df_price, on = 'HOUR') #мерджим основную таблицу и таблицу с ценами по столбцу HOUR (ставим в соответствие)
    df_priv = pd.read_csv('HAS_PRIVELEGE.csv', index_col=0) # считываем инфо по привилегиям
    df = pd.merge(df, df_priv, on = 'USER') #мерджим основную таблицу и привилегии
    df['PRIVELEGE'] = df['PRIVELEGE'].replace(1, 0.85) #для удобства расчета меняем значения, где есть привилегия на 85%
    df['PRIVELEGE'] = df['PRIVELEGE'].replace(0, 1) #для удобства расчета меняем значения, где нет привилегии на 100%
    df['SUBTOTAL_BILL'] = df['KWT'] * df['PRICE'] * df['PRIVELEGE'] #считаем значения по каждой записи в месяце
    return df

while i <=12: #всего 12 месяцев
    if i<=9: #так как формат названия месяцев 01..09, 10..12 разбиваем на два условия
        df = pd.read_csv('Month_0{}.csv'.format(i)) #загружаем таблицы от 1 до 9
        #print('идет расчет для 0{} месяца'.format(i))
        df = bill_count(df) #вызываем функцию расчета и передаем туда полученный из файла фрейм
        df = df.groupby('USER')['SUBTOTAL_BILL'].sum() #группируем по айдишнику пользователя и считаем сумму
        df = pd.DataFrame(df) #массив переводим в фрейм
        value_massive.append(df) #добавляем в массив расчет по месяцу
        i+=1 
    else:
        df = pd.read_csv('Month_{}.csv'.format(i)) #загружаем таблицы от 10 до 12
        #print('идет расчет для {} месяца'.format(i))
        df = bill_count(df)
        df = df.groupby('USER')['SUBTOTAL_BILL'].sum() #группируем по айдишнику пользователя и считаем сумму
        df = pd.DataFrame(df) #массив переводим в фрейм
        value_massive.append(df)
        i+=1
        
k=0 #вспомогательная переменная для цикла
for k in range(i-1):
    total_sum+=(value_massive[k]['SUBTOTAL_BILL'].sum()) #прибавляем сумму каждого месяца к общей сумме    
print(total_sum)
```

    4354858.0714
    


```python
df = pd.read_csv('Month_01.csv')
df = bill_count(df) #готовим данные для работы

df['TIME'] = pd.to_datetime(df['TIME'])
df = df.set_index('TIME') #устанавливаем индекс в TIME, чтобы далее можно было группировать по времени
df=df.groupby('USER').resample('15D').sum() # группируем по пользователю и по 15-ти дневке

kwt = df['KWT'].idxmax() #находим самое большое значение для KWT
print(kwt[0] + ' самый большой расход электроэнергии')
bill = df['SUBTOTAL_BILL'].idxmax() #находим самое большое значения для чека
print(bill[0] + ' самый большой чек')
```

    49a57420-f36f-1 самый большой расход электроэнергии
    49a573ee-f36f-1 самый большой чек
    
