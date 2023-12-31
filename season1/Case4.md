# Case4

まずはデータをインジェストしてテーブルを作るところから。
```
.execute database script <|
.create-merge table PrimeNumbers (PrimeNumber:int)
.ingest async into table PrimeNumbers (@'https://kustodetectiveagency.blob.core.windows.net/prime-numbers/prime-numbers.csv.gz')
```

special prime numberの定義を確認。`prev()`が鍵か。
```
PrimeNumbers
| sort by PrimeNumber asc
| extend newValue = PrimeNumber + prev(PrimeNumber) + 1
| join kind=inner
(
    PrimeNumbers
) on $left.newValue == $right.PrimeNumber
| top 1 by newValue
```
次のヒントにたどり着いた。

<!-- 
Well done, my friend.
It's time to meet. Let's go for a virtual sTREEt tour...
Across the Big Apple city, there is a special place with Turkish Hazelnut and four Schubert Chokecherries within 66-meters radius area.
Go 'out' and look for me there, near the smallest American Linden tree (within the same area).
Find me and the bottom line: my key message to you.

Cheers,
El Puente.

PS: You know what to do with the following:

----------------------------------------------------------------------------------------------

.execute database script <|
// The data below is from https://data.cityofnewyork.us/Environment/2015-Street-Tree-Census-Tree-Data/uvpi-gqnh 
// The size of the tree can be derived using 'tree_dbh' (tree diameter) column.
.create-merge table nyc_trees 
       (tree_id:int, block_id:int, created_at:datetime, tree_dbh:int, stump_diam:int, 
curb_loc:string, status:string, health:string, spc_latin:string, spc_common:string, steward:string,
guards:string, sidewalk:string, user_type:string, problems:string, root_stone:string, root_grate:string,
root_other:string, trunk_wire:string, trnk_light:string, trnk_other:string, brch_light:string, brch_shoe:string,
brch_other:string, address:string, postcode:int, zip_city:string, community_board:int, borocode:int, borough:string,
cncldist:int, st_assem:int, st_senate:int, nta:string, nta_name:string, boro_ct:string, ['state']:string,
latitude:real, longitude:real, x_sp:real, y_sp:real, council_district:int, census_tract:int, ['bin']:int, bbl:long)
with (docstring = "2015 NYC Tree Census")
.ingest async into table nyc_trees ('https://kustodetectiveagency.blob.core.windows.net/el-puente/1.csv.gz')
.ingest async into table nyc_trees ('https://kustodetectiveagency.blob.core.windows.net/el-puente/2.csv.gz')
.ingest async into table nyc_trees ('https://kustodetectiveagency.blob.core.windows.net/el-puente/3.csv.gz')
// Get a virtual tour link with Latitude/Longitude coordinates
.create-or-alter function with (docstring = "Virtual tour starts here", skipvalidation = "true") VirtualTourLink(lat:real, lon:real) { 
	print Link=strcat('https://www.google.com/maps/@', lat, ',', lon, ',4a,75y,32.0h,79.0t/data=!3m7!1e1!3m5!1s-1P!2e0!5s20191101T000000!7i16384!8i8192')
}
// Decrypt message helper function. Usage: print Message=Decrypt(message, key)
.create-or-alter function with 
  (docstring = "Use this function to decrypt messages")
  Decrypt(_message:string, _key:string) { 
    let S = (_key:string) {let r = array_concat(range(48, 57, 1), range(65, 92, 1), range(97, 122, 1)); 
    toscalar(print l=r, key=to_utf8(hash_sha256(_key)) | mv-expand l to typeof(int), key to typeof(int) | order by key asc | summarize make_string(make_list(l)))};
    let cypher1 = S(tolower(_key)); let cypher2 = S(toupper(_key)); coalesce(base64_decode_tostring(translate(cypher1, cypher2, _message)), "Failure: wrong key")
}
https://msazure.club/kusto-detective-agency-case-4/
 -->
