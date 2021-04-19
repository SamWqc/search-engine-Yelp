# search-engine-Yelp
 Search engine based on Pylucene

Index Build
--------------------------------------------------------------------------------

Index construction mainly completed in the **index_builder.ipynb**.

main codes
```
doc.add(Field("business_id", str(rest_info['business_id'][i]),StringField.TYPE_STORED))
doc.add(Field("name", str(rest_info['name'][i]),TextField.TYPE_STORED))
doc.add(Field("address", str(rest_info['address'][i]),TextField.TYPE_STORED))
doc.add(Field("categories", str(rest_info['categories'][i]),TextField.TYPE_STORED))
doc.add(Field("attributes", str(rest_info['attributes'][i]),TextField.TYPE_STORED))
doc.add(Field("city", str(rest_info['city'][i]),TextField.TYPE_STORED))
doc.add(Field("state", str(rest_info['state'][i]),TextField.TYPE_STORED))
doc.add(Field("postal_code", str(rest_info['postal_code'][i]),TextField.TYPE_STORED))
doc.add(Field("hours", str(rest_info['hours'][i]),TextField.TYPE_STORED))
doc.add(StringField("lat",str(rest_info['latitude'][i]),Field.Store.YES))
doc.add(StringField("long",str(rest_info['longitude'][i]),Field.Store.YES))
doc.add(LatLonPoint("location",float(rest_info['latitude'][i]),float(rest_info['longitude'][i])))
doc.add(FloatPoint("stars", float(rest_info['stars'][i]) ))
doc.add(StoredField('stars',float(rest_info['stars'][i]) ))
doc.add(IntPoint("review_count", int(rest_info['review_count'][i]) ))
doc.add(StoredField("review_count", int(rest_info['review_count'][i]) ))
doc.add(Field("review", str(rest_info['tip_text'][i]),TextField.TYPE_STORED))
```


Query 
--------------------------------------------------------------------------------

All query types are in the **query_total.ipynb**

**Boolean Query**

```
#query = city: portland ; categories: shopping and restaurant 
b2 = BooleanQuery.Builder()
b2.add(TermQuery(Term("city", "portland")), BooleanClause.Occur.MUST)
b2.add(TermQuery(Term("categories", "korean")), BooleanClause.Occur.MUST)
b2.add(TermQuery(Term("categories", "barbeque")), BooleanClause.Occur.MUST)
bq2 = b2.build()

for hit in rk1_hits:
    hitDoc = isearcher.doc(hit.doc)
    print(
          'Name:'+hitDoc['name']+'\t'+
          'Categories:'+hitDoc['categories']+'\t'+
          'Stars: '+hitDoc['stars']+'\t'+
          'Summary:'+ str(get_summary(hitDoc['review'],LANGUAGE,SENTENCES_COUNT)) )
    print('-'*100)
    
```

```
Business_id: 7lZBRWIBam0oAwFFOBvBhA	Name: Belmont Station	Address: 4500 SE Stark St	Categories: Nightlife, Bars, Beer Bar, Beer, Wine & Spirits, Beer Gardens, Food
----------------------------------------------------------------------------------------------------
Business_id: 3eGvOTVpmJ6B_dKwfGgXNw	Name: Hop City Craft Beer and Wine	Address: 99 Krog St NE	Categories: Beer Gardens, Food, Beer Bar, Beer, Wine & Spirits, Bars, Nightlife
----------------------------------------------------------------------------------------------------
Business_id: _eMpuJiJkBkG-j-I8CGOJg	Name: Imperial Bottle Shop & Taproom	Address: 2006 NE Alberta St	Categories: Beer Gardens, Nightlife, Beer, Wine & Spirits, Food, Bars, Beer Bar
----------------------------------------------------------------------------------------------------
```

**Ranking Query**

```
#query = 'portland salons with wifi'
multiFiled_query=  PythonMultiFieldQueryParser(['city','attributes','categories','review'],StandardAnalyzer())
multiFiled_query.setDefaultOperator(QueryParser.Operator.AND)
mrq = multiFiled_query.parse('portland salons with wifi',['city','attributes','categories','review'],
                     [BooleanClause.Occur.MUST, BooleanClause.Occur.SHOULD,BooleanClause.Occur.SHOULD,BooleanClause.Occur.SHOULD],
                     StandardAnalyzer())
                     
mrq_hits=isearcher.search(mrq,10).scoreDocs
for hit in mrq_hits:
    hitDoc = isearcher.doc(hit.doc)
    print('Business_id: '+hitDoc['business_id']+'\t'+
          'City: '+ hitDoc['city']+'\t'+
          'Name:'+hitDoc['name']+'\t'+
          'Categories:'+hitDoc['categories']+'\t'+
          'Stars: '+hitDoc['stars']+'\t'+
          'Score:%.2f'%(hit.score))
    print('-'*100)
    
```

```
Business_id: 63sw2U3K_CgimD8claSkAg	City: Portland	Name:Moda Studios-Ceanna Lee	Categories:Beauty & Spas, Men's Hair Salons, Hair Salons, Hair Stylists	Stars: 4.5	Score:7.83
----------------------------------------------------------------------------------------------------
Business_id: fFoJtbo7K_7-NjJubxD4SA	City: Portland	Name:Luxury Nails & Foot Massage	Categories:Hair Removal, Waxing, Massage, Beauty & Spas, Day Spas, Nail Salons	Stars: 4.0	Score:7.59
----------------------------------------------------------------------------------------------------
Business_id: y_xHlaTePE7qnf2AO1-OrA	City: Portland	Name:Color Treats	Categories:Waxing, Nail Salons, Massage, Hair Removal, Beauty & Spas, Nail Technicians	Stars: 5.0	Score:6.95
```

**Numerical Query**

```
#query = categories: bar  with stars higher than 4.5
range_q = FloatPoint.newRangeQuery('stars',4.5,5.)
rb = BooleanQuery.Builder()
rb.add(TermQuery(Term("categories", "bar")), BooleanClause.Occur.MUST)
rb.add(TermQuery(Term("city", "portland")), BooleanClause.Occur.MUST)
rb.add(range_q,BooleanClause.Occur.SHOULD)
rbq = rb.build()

rbq_hits = isearcher.search(rbq,10).scoreDocs
for hit in rbq_hits:
    hitDoc = isearcher.doc(hit.doc)
    print('City: '+hitDoc['city']+'\t'+
          'Name: '+hitDoc['name']+'\t'+
          'Star: '+hitDoc['stars']+'\t'+
          'Categories:'+hitDoc['categories']
    )
    print('-'*100)
```

```
City: Portland	Name: Beer O'Clock	Star: 4.5	Categories:Bars, Beer Bar, Nightlife
----------------------------------------------------------------------------------------------------
City: Portland	Name: Lombard House	Star: 5.0	Categories:Pubs, Beer Bar, Bars, Nightlife
----------------------------------------------------------------------------------------------------
City: Portland	Name: Barrio	Star: 5.0	Categories:Wine Bars, Bars, Beer Bar, Nightlife
```

**Spatial Query**

```
#spatial information 
cur_location = ( 45.588906,-122.593331)
radius = 5000.
geo_q=LatLonPoint.newDistanceQuery(
'location',
    cur_location[0],
    cur_location[1],
    5000.
)

#keyword information
key_q = BooleanQuery.Builder()
key_q.add(TermQuery(Term("categories", "chinese")), BooleanClause.Occur.SHOULD)
key_q.add(TermQuery(Term("categories", "restaurant")), BooleanClause.Occur.SHOULD)
key_q.add(geo_q,BooleanClause.Occur.MUST)
geo_key1 = key_q.build()

#query = categories: chinese and restaurant within 5000. meters away

b1_hits = isearcher.search(geo_key1,10).scoreDocs
for hit in b1_hits:
    hitDoc = isearcher.doc(hit.doc)
    
    print('City:'+hitDoc['city']+'\t'+
          'Name:'+hitDoc['name']+'\t'+
          'Address:'+hitDoc['address']+'\t'+
          'Categories:'+hitDoc['categories']+'\t'+
          'Distance:'+str(geodistance(hitDoc['long'],hitDoc['lat'] , cur_location[1],cur_location[0]))+ 'km'
    )
    print('-'*100)
          
```

```
City:Vancouver	Name:Lucky Garden Restaurant	Address:10204 NE Mill Plain Blvd	Categories:Chinese, Restaurants	Distance:4.092km
----------------------------------------------------------------------------------------------------
City:Vancouver	Name:Chopsticks Restaurant	Address:7601 E Mill Plain Blvd	Categories:Chinese, Restaurants	Distance:3.966km
----------------------------------------------------------------------------------------------------
City:Vancouver	Name:Ming's Restaurant	Address:11909 SE Mill Plain Blvd	Categories:Chinese, Restaurants	Distance:4.842km
```

Summary
--------------------------------------------------------------------------------

summary part is in the **summary.ipynb** with package sumy https://github.com/miso-belica/sumy

```
#query= 'chinese food with good servce'
r1 = QueryParser('review',StandardAnalyzer())
rk1 = r1.parse('portland korean barbeque')
rk1_hits=isearcher.search(rk1,10).scoreDocs

for hit in rk1_hits:
    hitDoc = isearcher.doc(hit.doc)
    print(
          'Name:'+hitDoc['name']+'\t'+
          'Categories:'+hitDoc['categories']+'\t'+
          'Stars: '+hitDoc['stars']+'\t'+
          'Summary:'+ str(get_summary(hitDoc['review'],LANGUAGE,SENTENCES_COUNT)) )
    print('-'*100)
    
```

```
Name:New Jang Su BBQ	Categories:Korean, Restaurants, Barbeque	Stars: 3.5	Summary:Pottery bi bim bop (rice) and chap chae (noodles).
----------------------------------------------------------------------------------------------------
Name:Wavey's Bar-B-Que	Categories:Food, Restaurants, Food Trucks, Barbeque	Stars: 3.5	Summary:Good old fashioned barbeque.
----------------------------------------------------------------------------------------------------
Name:D-Street  Noshery	Categories:Restaurants, Food Stands	Stars: 4.0	Summary:This is how the food cart scene needs to evolve .. D Street Noshery is closed for good.. Best pie in Portland
```
