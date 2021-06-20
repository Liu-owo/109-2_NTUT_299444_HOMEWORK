# 使用pandas分析網頁日誌

### 參考資料來源與內容
主要來源：[Using Python Pandas for Log Analysis](https://akshayranganath.github.io/Using-Python-Pandas-for-Log-Analysis/)。
- 初始化資料庫 (import pandas)
- 讀取csv檔
- 計算、篩選、排序資料
- 印出分析資料

網頁日誌來源：[Online Judge ( RUET OJ) Server Log Dataset](https://www.kaggle.com/shawon10/web-log-dataset)。


### 實作結果
##### 一、啟動Jupyter Lab(win10)
1.下載Anacoda 3。(可以事先安裝好Python)
2.啟動Anaconda 3 Prompt。
3.輸入**jupyter lab**並執行，會自動跳出網頁(或複製執行結果的URL也可以)。
```sh
(base) C:\ > jupyter lab
```
4.進入網頁，創建一個NoteBook。

##### 二、使用pandas操作
1.載入pandas套件。
```sh
import pandas as pd  #載入pandas
```
2.匯入csv。
 ```sh
df = pd.read_csv("D:\\Documents\weblog.csv") #匯入csv檔
df.head() #顯示列0~4的資料，()內可輸入數字控制顯示的列數
```
3.剔除不必要的列。
```sh
# 欄'IP'包含的這些關鍵字都存在indexNames
indexNames = df[ (df['IP'] == 'a.out:') | (df['IP'] == 'chmod:') | (df['IP'] == 'rm:') | 
(df['IP'] == 'sh:') | (df['IP'] == 'timeout:') | (df['IP'] == '[Fri') |
(df['IP'] == '[Mon') |(df['IP'] == '[Sat') | (df['IP'] == '[Thu') | (df['IP'] == '[Tue') | (df['IP'] == '[Wed')].index

df.drop(indexNames , inplace=True) #刪除有再indexNames裡的, 列(inplace= True)
df['Staus'] = pd.to_numeric(df['Staus']) #將Staus轉成數值型資料
```
4.把月份擷取出來，並替換成20xx.xx。
```sh
#提取欄'Time'裡[4]~[11]位置的文字，並放在新增的欄'Months'
df['Months'] = df['Time'].str[4:12]
#如果欄Months的資料符合'Nov/2017'，就在新增的欄'Month'顯示2017.11(為了可以順利轉換為數值型資料)
df.loc[df['Months'] == 'Nov/2017', 'Month'] = '2017.11'  
#如果欄Months的資料符合'Dec/2017'，就在新增的欄'Month'顯示2017.12
df.loc[df['Months'] == 'Dec/2017', 'Month'] = '2017.12' 
#如果欄Months的資料符合'Jan/2018'，就在新增的欄'Month'顯示2018.01
df.loc[df['Months'] == 'Jan/2018', 'Month'] = '2018.01'  
#如果欄Months的資料符合'Feb/2018'，就在新增的欄'Month'顯示2018.02
df.loc[df['Months'] == 'Feb/2018', 'Month'] = '2018.02'
#如果欄Months的資料符合'Mar/2018'，就在新增的欄'Month'顯示2018.03
df.loc[df['Months'] == 'Mar/2018', 'Month'] = '2018.03'  
df.head()
```
5.紀錄StatusOK ,Rediraction ,Error的數量。
```sh
#新增欄'Status OK'，如果符合條件，就顯示'1'，否則為'0'
df['Status OK'] = df['Staus'].apply(lambda x: '1' if x <300 else '0')   
#新增欄'Status Rediraction'，如果符合條件，就顯示'1'，否則為'0'
df['Status Rediraction'] = df['Staus'].apply(lambda x: '1' if x >=300 | x<400 else '0') 
#新增欄'Status Client Error'，如果符合條件，就顯示'1'，否則為'0'
df['Status Client Error'] = df['Staus'].apply(lambda x: '1' if x>=400 else '0')   
df['Status OK'] = pd.to_numeric(df['Status OK'])                       #轉成數值型資料
df['Status Rediraction'] = pd.to_numeric(df['Status Rediraction'])     #轉成數值型資料
df['Status Client Error'] = pd.to_numeric(df['Status Client Error'])   #轉成數值型資料
df.head()
```
6.加總目前的數值型資料。
```sh
results = df.groupby('Month').sum() 以Month為主加總資料，並指派給變數results
```
7.列印Status(200、300、400)計數的bar chart，查看Client Error發生的數量。
```sh
import matplotlib.pyplot as plt #載入資料視覺化套件
Months = ['2017.11','2017.12','2018.01','2018.02','2018.03'] #新增一串列Months，當作x軸使用

#新增一bar，x軸為Months，y軸為results裡的Staus OK，alpha調整顏色,圖例顯示名稱='OK'
OK = plt.bar(Months,results['Status OK'], alpha = 0.4, label = 'OK ') 
#新增第二bar，x軸為Months，y軸為results裡的Staus Rediraction，alpha調整顏色,圖例顯示名稱='Rediraction'
Red = plt.bar(Months,results['Status Rediraction'], alpha = 0.4, label = 'Rediraction')
#新增第三bar，x軸為Months，y軸為results裡的Staus Client Error，alpha調整顏色,圖例顯示名稱='Client Error'
CErr = plt.bar(Months,results['Status Client Error'], alpha = 0.4, label = 'Client Error')

def createLabels(data): #每個bar顯示數值
    for item in data:
        height = item.get_height()
        plt.text(
            item.get_x()+item.get_width()/2., 
            height*1.05, 
            '%d' % int(height),
            ha = "center",
            va = "bottom",
        )
createLabels(OK)
createLabels(Red)
createLabels(CErr)

plt.ylabel("Number of Status")#y軸的標籤名字
plt.xlabel("Months")#x軸的標籤名字
plt.title("Number of Status in Every Months ")#title的名字

plt.legend()#顯示圖例
plt.show()
```
![N|Solid](https://raw.githubusercontent.com/Liu-owo/109-2_NTUT_299444_HOMEWORK/main/Bar%20chart_1.PNG)
9.提取HTTP Method有關的資料
```sh
df['HTTP Method'] = df['URL'].str[0:4] #提取URL資料裡0~3的文字，將提取的文字放在HTTP Method
```
10.將提取的資料加入並轉換在新增的欄裡
```sh
df.loc[df['HTTP Method'] == 'GET ', 'GET'] = '1'  #尋找HTTP Method裡為GET 的資料，新增欄'GET',如果有找到，則在新欄顯示'1'
df.loc[df['GET'].isnull(), 'GET'] = 0      #填補空值為0
df.loc[df['HTTP Method'] == 'POST', 'POST'] = '1'  #尋找HTTP Method裡為POST的資料，新增欄'POST',如果有符合，則在新欄顯示'1'
df.loc[df['POST'].isnull(), 'POST'] = 0      #填補空值為0
df['GET'] = pd.to_numeric(df['GET'])      #將欄'GET轉換為數值型資料
df['POST'] = pd.to_numeric(df['POST'])    #將欄'POST'轉換為數值型資料
df.head()
```
11.資料以Month進行加總。
```sh
df.groupby('Month').sum() #加總目前的數值型資料
```
12.之後要新增折線圖，考量資料型態問題，再建立一個表格。
```sh
#自己再建一個表格
dataGP = {'Month':['201711', '201712', '201801', '201802', '201803'],
        'GET':[6772, 1821, 5239, 1102, 164], 'POST':[482, 108, 12, 70, 10]} #資料欄列內容
df_GP = pd.DataFrame(dataGP) #顯示表格
df_GP.head() #顯示表格
```
13.畫GET和POST計數的折線圖
```sh
#畫第一條線，plt.plot(x, y, c)分別為x軸資料、y軸資料、線顏色 = 紅色
plt.plot(df_GP['Month'], df_GP['GET'],c = "r")  
#畫第二條線，plt.plot(x, y, c)分別為x軸資料、y軸資料、線顏色 = 綠色及線條為 -.
plt.plot(df_GP['Month'], df_GP['POST'], "g-.")  
plt.legend(labels=["GET", "POST"], loc = 'best')            # 設定圖例，參數為標籤, 位置
plt.xlabel("YearMonth", fontweight = "bold")                # 設定x軸標題及粗體
plt.ylabel("Number of HTTP Method", fontweight = "bold")    # 設定y軸標題及粗體
# 設定標題、文字大小、粗體及位置
plt.title("HTTP Method", fontsize = 15, fontweight = "bold", y = 1.1)   
plt.xticks(rotation=45)   # 將x軸數字旋轉45度
```
![N|Solid](https://raw.githubusercontent.com/Liu-owo/109-2_NTUT_299444_HOMEWORK/main/0623_Final%20Homework/plot_1.PNG)
### 實作困難點

##### csv檔存放在D槽
- 若擔心無法順利讀取csv檔，可以將檔案路徑全部打出來

```sh
df = pd.read_csv("D:\\Documents\weblog.csv")
```

##### 確認各資料的屬性
```sh
df.dtypes
```
##### 無法成功將String轉換成數值型資料

- 分析資料前，要先檢查資料有沒有空值，或出現不同形式的資料，若有出現，可能需要刪除或變更資料。
- 在轉換時，因為有出現不同形式的資料，導致無法順利轉換，我選擇將關鍵字詞存在indexNames，然後用drop將包含字詞的那些列刪除。

```sh
indexNames = df[ (df['IP'] == 'a.out:') | (df['IP'] == 'chmod:') | (df['IP'] == 'rm:') | 
(df['IP'] == 'sh:') | (df['IP'] == 'timeout:') | (df['IP'] == '[Fri') |
(df['IP'] == '[Mon') |(df['IP'] == '[Sat') | (df['IP'] == '[Thu') | (df['IP'] == '[Tue') | (df['IP'] == '[Wed')].index
df.drop(indexNames , inplace=True)
```
- 轉換成數值型資料或是object
```sh
df['欄'] = pd.to_numeric(df['欄'])    #object轉換成數值型資料
df['欄'] = df['欄'].astype(str)       #int轉換成object
```

##### 依照月份加總資料
可以指派給某個變數，指派不會自動印出表格，有指派則會自動印出。
```sh
df.groupby('Month').sum()
```

##### 資料時間有跨年，在輸出為Bar Chart時，會發生時間順序問題。
- y軸的資料是對著x軸跑，直接再新增一個串列去代換x

##### bar chart 所需資料及顯示圖例
- 表格所需資料： plt.bar(x軸資料, y軸資料,bar寬度,alpha=.4(顏色),)
- plt.legend()可印出圖例，()裡沒打的話，要在新增圖表時就指定，或之後在()裡指定
```sh
import matplotlib.pyplot as plt
import numpy as np
Months = ['2017.11','2017.12','2018.01','2018.02','2018.03']
plt.bar(Months,results['Status OK'], alpha = 0.4, label = 'OK ') 
plt.legend()
plt.show()
```
##### bar chart的bar有顯示資料數字
[有使用到plt.text()的參考資料](https://stackoverflow.com/questions/40489821/how-to-write-text-above-the-bars-on-a-bar-plot-python)
```sh
def createLabels(data): #每個bar顯示數值
    for item in data:
        height = item.get_height()
        plt.text(
            item.get_x()+item.get_width()/2., 
            height*1.05, 
            '%d' % int(height),
            ha = "center",
            va = "bottom",
        )
```
##### 類別型資料無法直接用在折線圖上
- 我選擇加總完後，直接新增一個表格，用新增的表格去畫折線圖

```sh
df.groupby('Month').sum() #加總目前的數值型資料

#自己再建一個表格
dataGP = {'Month':['201711', '201712', '201801', '201802', '201803'],
        'GET':[6772, 1821, 5239, 1102, 164], 'POST':[482, 108, 12, 70, 10]} #資料欄列內容
df_GP = pd.DataFrame(dataGP) #顯示表格
df_GP.head() #顯示表格

#畫第一條線，plt.plot(x, y, c)分別為x軸資料、y軸資料、線顏色 = 紅色
plt.plot(df_GP['Month'], df_GP['GET'],c = "r")  
#畫第二條線，plt.plot(x, y, c)分別為x軸資料、y軸資料、線顏色 = 綠色及線條為 -.
plt.plot(df_GP['Month'], df_GP['POST'], "g-.") 