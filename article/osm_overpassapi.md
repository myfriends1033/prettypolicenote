# *OSM and overpass api*

# OSM
元素是OpenStreetMap的物理世界概念數據模型的基本組成部分。其元素並非 點、線、面 資料，而是 **node**、**way**、**relation**，以上元素接可含有一個或多個關聯的 **tag**  
後兩種資料皆以 **node** 為基礎形成，也就是 way、relation 資料都是2個或以上的node組合而成：  

- node 節點： 空間中的一個點，該點至少由緯度、經度、ID組成。任一 node 可能為"**點資料**"、**線**的節點、**面**的節點
- way 路徑/形狀： 由2~2000個 node 組成，可以是"**線資料**"(線性特徵)或者"**面資料**"(區域邊界)。當一個區域超過2000個 node 時，就必須使用 relation資料來定義
- relation 關係： 是一個多用途的結構資料，定義 兩個或更多 資料之間的關係。在 relation 中，可能會有**點資料**、**線資料**、**面資料**，由 **"members"** 包在一起。
![members](C:\OSM\members.JPG)
當然也可能是分別的 node、way、relation ，最後被整合在同一個群組，就成了 relation 資料 ( 也就表示可以由兩個以上的 relation 再次組成一個 relation，也可以是 點與面 有所關聯而group在一起 )，常見的是 兩個以上的 way 組成一個 relation
- tag 標籤：OpenStreetMap represents physical features on the ground using tags attached to its basic data structures (its nodes, ways, and relations). Each tag describes a geographic attribute of the feature being shown by that specific node, way or relation.
- 因為 OSM 是開放的，且可以編輯，所以取得的資料**無時無刻都有可能浮動**，**尤其是數量**，搜尋同的條件下，此時得到的資料量，在未來也許會不同

這次範例，軍事機場的標籤 → "military"="airfield"


# Overpass API

以下只筆記我使用到的 overpass turbo、overpy


## [overpass turbo](https://overpass-turbo.eu/ )

是一個能即時展示在網路地圖上的一個API，並且該網站功可能從**地圖**切換到**資料**，直接查看搜尋到的資料全部內容。

基本操作：只要在網頁中使用 "**精靈模式**" 打上 "military"="airfield" 後就會自動生成以下

```QL

[out:json];
(
  node["military"="airfield"]({{bbox}});
  way["military"="airfield"]({{bbox}});
  relation["military"="airfield"]({{bbox}});
);
out body;
>;
out skel qt;

```

- ["military"="airfield"] 中括號內放要搜尋的 **tag** 
- ({{bbox}}) 表示在網頁中的**可視範圍**作為搜尋邊界，也可改成需要的範圍(緯,經,緯,經)，可以在[這裡](http://tools.geofabrik.de/calc/) 取得，請注意該網頁取得的是(經 緯 經 緯)，如果沒有輸入，就是搜尋全球資料
- out body; : This prints all information necessary to use the data. These are also tags for all elements and the roles for relation members. see Print action. This first step will print all of the node details, the ways (without nodes(!)) and relations (without details about ways and nodes(!)) you requested.
- &gt; ; : This statement is called recurse down and resolves ways to its nodes, and relations to its ways and nodes.
- out skel qt; - in this last statement, we will now print out the results of the previous >; (recurse down) statement. To reduce the amount of unwanted by-catch, we use out skel instead of out body;. The idea here is to just return the geometry information, but leave out any tags. qt is a sorting option (by quadtile), which is also meant as a performance optimization.
- 內容很多，可參考 [這裡](https://gis.stackexchange.com/questions/187447/understanding-overpassturbo-query) 和 [這裡](https://wiki.openstreetmap.org/wiki/Overpass_API/Overpass_QL)


## [overpy](https://python-overpy.readthedocs.io/en/latest/ )

透過 Python 界接的 API，功能齊全， query() 來輸入 QL：

```python
import overpy
api = overpy.Overpass()
result = api.query('''
[out:json];
(
  node["military"="airfield"](30.82,128.82,34.67,132.81);
  way["military"="airfield"](30.82,128.82,34.67,132.81);
  relation["military"="airfield"](30.82,128.82,34.67,132.81);
);
out body;
>;
out skel qt;
''')
```
- 其中()內必須座標是(緯,經,緯,經)，可以在 [這裡](http://tools.geofabrik.de/calc/) 取得，請注意該網頁取得的是(經 緯 經 緯)，如果沒有輸入，就是搜尋全球資料
- out:json 也可以換成 out:xml


因為整個資料的結構(node、way、relation)，以至於必須分開實作：

### 利用 node 取得點

#### 先引入套件並設定 QL

```python
import overpy

api = overpy.Overpass()
result = api.query('''
[out:json];
(
  node["military"="airfield"];
);
out body;
>;
out skel qt;''')
```

- 在QL中只要求點，且不輸入經緯度，以求全球資料。而因為有後兩行的輸入，才能得到所需的點，不然會沒有資料
- 因為 overpy 沒辦法可視化，所以 QL 的輸入，可以先在overpass turbo中測試，或者利用 `print(len(result.nodes))` 來查看取得多少資料

#### 接下來擷取所需資料

```python
namep = []
xp = []
yp = []

for point in result.nodes:
    namep.append(point.tags.get("name", ""))
    xp.append(float(point.lon))
    yp.append(float(point.lat))
```

- 資料的結構可利用 overpass turbo 來觀看如下：

> 
> {  
> "type": "node",  
> "id": 30353161,  
> "lat": -51.8208779,  
> "lon": -58.4565304,  
> "tags": {  
> 			"aerodrome:type": "military/public",  
> 			"aeroway": "aerodrome",  
> 			"alt_name": "Mount Pleasant Airport",  
> 			"ele": "74",  
> 			"iata": "MPN",  
> 			"icao": "EGYP",  
> 			"is_in": "Stanley",  
> 			"is_in:country_code": "FK",  
> 			"military": "airfield",  
> 			"name": "RAF Mount Pleasant",  
> 			"name:es": "Base Aerea Monte Agradable",  
> 			"operator": "RAF",  
> 			"source": "wikipedia",  
> 			"wikidata": "Q2071294",  
> 			"wikipedia": "en:RAF Mount Pleasant"}  
> }
>

- 利用 for 迴圈取的每個點資料 "tags" 中的屬性，而 tags 是一個字典，所以利用 `.tag.get`，括號內最多可以設定兩個

  + ("name", " ") 是找尋 "name" 屬性，沒有的話就找尋下一個 " " 屬性，如果還是沒有就會取後面的雙引號裡的名，當作回傳的資料

- 而看了結構之後會發現，經緯度並不是在 tags 裡面，而是跟 tags 為同一層，所以直接利用 `.lon` `.lat` 直接取的經緯度

- 將要取得的屬性，存進個別的 list 裡，是為了將多個 list 存成 dataframe，方便等等一併加進資料庫

#### 連接 PostgreSQL 資料庫前，先打開終端機設定資料庫

1. 先導至選定的資料庫 (JLDB)，替該資料庫建立 postgis 套件

```terminal
CREATE EXTENSION postgis;
CREATE EXTENSION postgis_topology;
```

2. 再建立資料表 (airfield_point)

```terminal
CREATE TABLE airfield_point (
        point_id SERIAL PRIMARY KEY,
        name VARCHAR(144),
        x double precision,
        y double precision,
        geom geometry(point,4326)
        );
```

#### 連接資料庫並將結果存進資料庫

```python
import pandas as pd
import psycopg2

df = pd.DataFrame(
    {'name' : namep,
    'x' : xp,
    'y' : yp })
dfp = df.values.tolist()

connection = psycopg2.connect(host='localhost',port=5432,user='postgres',password='postgresql',database='JLDB')
cursor = connection.cursor()
sql =('''
INSERT INTO "public"."airfield_point"(name,x,y) values (%s,%s,%s);

UPDATE "public"."airfield_point" SET geom = ST_SetSRID(ST_MakePoint(foo.x,foo.y),4326) FROM(SELECT point_id,x,y FROM "public"."airfield_point") as foo WHERE "public"."airfield_point".point_id = foo.point_id;      
''')
cursor.executemany(sql, dfp)
connection.commit()

cursor.close()
connection.close()
```
- 將剛剛個別的 list 存成 dataframe，再轉成 list，使得每一筆資料變成  [name ,lon , lat]，就能利用批次 `.executemany` 加進資料庫裡

- %s 是 PostgreSQL 可支援的 format ，數量必須剛好對應到後面的 dfp 內每個子資料的資料數量： [ [name ,lon , lat] , [name1 ,lon1 , lat1] , ...] 

- `UPDATE "public"."airfield_point" SET geom ...` 是為了讓 x,y 欄位轉換成點資料，存進 geom 欄位

#### 最後利用 QGIS 連接 PostgreSQL 展示結果

### 利用 way 取得面與線

```python
import overpy

api = overpy.Overpass()
result = api.query('''
[out:json];
(
  way["military"="airfield"];
);
out body;
>;
out skel qt;
''')
```
1. 改為 way["military"="airfield"]

2.  必須微調 overpy 套件，打開 \python3\Lib\site-packages\overpy 目錄下的 \_\_init\_\_.py ，更改如下：  
![套件更改1](C:\OSM\套件更改.JPG)
如此一來，迴圈裡的 `.nodes` 就能得到一連串經緯度 list [(經 緯),(經 緯), ...]

```python
import pandas as pd

namew = []
xyw = []
xylw = []
idw = []

for way in result.ways:
    namew.append(way.tags.get("name", ""))
    xy = 'POLYGON({})'.format(tuple(way.nodes))
    xyw.append(xy)
    xyl = 'LINESTRING{}'.format(tuple(way.nodes))  
    xylw.append(xyl)
    idw.append(way.id)

df = pd.DataFrame(
    {'name' : namew,
    'polgyon' : xyw,
    'polgline' : xylw,
    'id' : idw}
    )
dfw = df.values.tolist()
```

- `result.ways` 中利用 `.tags.get` 迴圈取得 tags 中的屬性資料

#### 以下說明 POLYGON、POLYLINE 怎麼加入資料庫

PostgreSQL 如何加入面與線資料，如下：

```terminal
INSERT INTO areas (name, polygon) VALUES (
'紗帽山',
ST_GeometryFromText('POLYGON((121.538805 25.151908,121.545940 25.151121,121.547917 25.149118,121.548638 25.146912,121.547345 25.143964,121.545256 25.142366,121.544187 25.142074,121.542696 25.143244,121.540955 25.142907,121.539439 25.143672,121.539290 25.143964,121.537574 25.147363,121.538805 25.151908))',4326)
);
```
```terminal
INSERT INTO areas (name, polyline) VALUES (
'光華路',
ST_GeometryFromText('LINESTRING(121.542541 25.136911,121.543168 25.137378,121.544962 25.137951,121.545797 25.136813,121.546347 25.136966)',4326)
);
```
- 為了生成 `ST_GeometryFromText( __ ,4326)` 中的一連串經緯度 ，於是將 `way.nodes` 得到的結果設定為 `xy = 'POLYGON({ }).format(tuple(way.nodes))'`及`xyl = 'LINESTRING{}'.format(tuple(way.nodes))`，以便後續加入資料庫
- 一連串的經緯度最後必須封閉才能被 Postgis 設定為面資料，而 way 事實上還包含 **線資料**，也就表示遇到線就無法加入資料庫，所以接下來改用 `for`迴圈，並利用 `.execute` 一筆一筆加入，以 `try` 加入面資料，失敗的則 `except` 再 `try` 改成線資料加入，如果還是失敗則 `except` 印出該筆 **ID**

```python
import psycopg2

for dfwfor in dfw:
    try:
        connection = psycopg2.connect(host='localhost',port=5432,user='postgres',password='postgresql',database='JLDB')
        cursor = connection.cursor() 
        sql =('''
        INSERT INTO "public"."airfield_polygon" (name,geom) values (%s,ST_GeomFromText(%s,4326));
        ''')
        data = (dfwfor[0],dfwfor[1])
        cursor.execute(sql,data)
        connection.commit()
		
        cursor.close()
        connection.close()
    except:
	    try:
            connection = psycopg2.connect(host='localhost',port=5432,user='postgres',password='postgresql',database='JLDB')
            cursor = connection.cursor() 
            sql =('''
            INSERT INTO "public"."airfield_polygon" (name,geom) values (%s,ST_GeomFromText(%s,4326));
            ''')
            data = (dfwfor[0],dfwfor[2])
            cursor.execute(sql,data)
            connection.commit()
		    
            cursor.close()
            connection.close()
        except:
            print(dfwfor[3])
```
- 以空軍基地來說，way 的 polgline 沒有很多，比較完整的在圖上看起來是 polygon，但因為沒有封閉，而只能是 polgline，還分屬不同筆資料，如下圖：
![沒有封閉](C:\OSM\way_error2.JPG)
- 但有的甚至還沒編輯好，如下圖：
![尚未完成](C:\OSM\way_error.JPG)

### relation
relation 的資料很多元，，以空軍基地來說，有些是幾個相鄰的 polygon 組成的，有些是 **MultiPolygon** (甚至可能有 點、線、面 的  **GeometryCollection**，才組成一個 面資料)
![multi](C:\OSM\multipolygon.JPG)
- relation 下的元素是由 **"members"** 包在一起，如先前所說，大部分是多個 **way** 組成一個 relation，而上圖就是一個 **MultiPolygon** 。利用 overpass-turbo 查看數據時，可在 members 下發現如下圖：
![multidata](C:\OSM\multi_data.JPG)
也就是被挖空的部分是 **inner** ，所以取得資料時，無法簡單的將所有經緯度取得後，直接建立在資料庫內，甚至有的 **outer** 是 線資料，是由很多條線資料結合後封閉成完整的 面資料

- 除了 overpass-turbo 可查看資料，也可微調 overpy 套件來查看 outer、inner，打開 \python3\Lib\site-packages\overpy 目錄下的 \_\_init\_\_.py ，更改如下：  
![套件更改2](C:\OSM\套件更改2.JPG)

relation 是由 members 下的很多的元素(nodes、ways、relations)組成的，所有元素都有其經緯度，所以可能先獲取所有的空軍基地的 id 後，再想辦法組合 
```python
import overpy

api = overpy.Overpass()
idr = []
def get_idr():
    global idr
    result = api.query(
    '''
    [out:json];
    (
      relation["military"="airfield"];
    );
    out body;
    >;
    out skel qt;
    ''')
    
    for re in result.relations:
        idr.append(re.id)

print(get_idr())
```

- 找了一下資料，網路上說 relation 得到的數據，還是得定義如何將所有 members 組合，可能的步驟應該是撇除 node 後： 
1. 首先，先分出 outer 和 inner
```python
for id in idr:
    result = api.query('''
    [out:json];
    (
      relation(id:{});
    );
    out;
    >;
    out skel qt;
    '''.format(id))
    
    print(result.relations)
```
2. 再分是 way 的 面資料 組成面、way 的 線資料 封閉成面 (甚至 members 下可能還會有 **子relation**，但此案沒有)
3. 最後將 outer 生成的面 merge 後， erase inner 的部分

### [<<Back](https://myfriends1033.github.io/)

