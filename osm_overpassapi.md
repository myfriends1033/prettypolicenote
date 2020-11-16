# OSM and overpass api

# OSM
元素是OpenStreetMap的物理世界概念數據模型的基本組成部分。其元素並非 點、線、面 資料，而是 **node**、**way**、**relation**，以上元素接可含有一個或多個關聯的 **tag**  
後兩種資料皆以 **node** 為基礎形成，也就是 way、relation 資料都是2個或以上的node組合而成：  

- node 節點： 空間中的一個點，該點至少由緯度、經度、ID組成。任一 node 可能為"**點資料**"、**線**的節點、**面**的節點 <P>
- way 路徑/形狀： 由2~2000個 node 組成，可以是**線資料**(線性特徵)或者**面資料**(區域邊界)。當一個區域超過2000個 node 時，就必須使用 relation資料來定義 <p>
- relation 關係： 是一個多用途的結構資料，定義 兩個或更多 資料之間的關係。在 relation 中，可能會有**點資料**、**線資料**、**面資料**。可能是分別的 node、way、relation ，最後被整合在同一個群組，就成了 relation 資料 ( 也就表示可以由兩個以上的 relation 再次組成一個 relation，也可以是 點與面 有所關聯而group在一起 )，常見的是 兩個以上的 way 組成一個 relation <P>
- tag 標籤：OpenStreetMap represents physical features on the ground using tags attached to its basic data structures (its nodes, ways, and relations). Each tag describes a geographic attribute of the feature being shown by that specific node, way or relation.

這次範例，軍事機場的標籤 → "military"="airfield"


# Overpass API

以下只筆記我使用到的 overpass turbo  、 overpy


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
- ({{bbox}}) 表示在網頁中的**可視範圍**作為搜尋邊界，也可改成需要的範圍 (緯,經,緯,經)
- out body; : This prints all information necessary to use the data. These are also tags for all elements and the roles for relation members. see Print action. This first step will print all of the node details, the ways (without nodes(!)) and relations (without details about ways and nodes(!)) you requested.
- &gt; ; : This statement is called recurse down and resolves ways to its nodes, and relations to its ways and nodes.
- out skel qt; - in this last statement, we will now print out the results of the previous >; (recurse down) statement. To reduce the amount of unwanted by-catch, we use out skel instead of out body;. The idea here is to just return the geometry information, but leave out any tags. qt is a sorting option (by quadtile), which is also meant as a performance optimization.
- 內容很多，可參考 [這裡](https://gis.stackexchange.com/questions/187447/understanding-overpassturbo-query) 和 [這裡](https://wiki.openstreetmap.org/wiki/Overpass_API/Overpass_QL)


## [overpy](https://python-overpy.readthedocs.io/en/latest/ )

透過 Python 界接的 API，功能齊全， query() 來輸入 QL，：

```python
import overpy
api = overpy.Overpass()
result = api.query('''
[out:json];
(
  node["military"="airfield"];
  way["military"="airfield"];
  relation["military"="airfield"];
);
out body;
>;
out skel qt;
''')
```




