# popularity-based-recommender
recommends blogs to users based on the popularity of blogs on a website it uses the dtabase from mongodb.
!pip install pymongo[srv]
from datetime import datetime
import pymongo
from pymongo import MongoClient
import urllib.parse

username = urllib.parse.quote_plus('your username')
password = urllib.parse.quote_plus('your password')

url = "paste the url of you clustered database from mongodb".format(username, password)


cluster = MongoClient(url)
db = cluster['name of the cluster created in the database']
collection = db['blog posts']
for i in db.posts.find({}):
  print(i.keys())
  break
import pandas as pd
lst=[]
for i in db.posts.find({}):
  lst.append((i['_id'],i['num_of_views'],len(i['likes']),i['comments'],i['num_of_shares'],i['published_on'],divmod((datetime.now()-i['published_on']).total_seconds(),3600)[0]))
df=pd.DataFrame(lst,columns=['post_id','num_of_views','likes','comments','num_of_shares','date_of_publish','hours'])
df
post_data={} # dictionary of posts data
for i in db.posts.find({}):
  post_data[str(i['_id'])]={}
  post_data[str(i['_id'])]['num_of_views']=i['num_of_views']
  post_data[str(i['_id'])]['likes']=len(i['likes'])
  post_data[str(i['_id'])]['comments']=i['comments']
  post_data[str(i['_id'])]['num_of_shares']=i['num_of_shares']
  duration_in_s = (datetime.now()-i['published_on']).total_seconds()
  post_data[str(i['_id'])]['hours']=divmod(duration_in_s, 3600)[0]
post_data
def popularity_of_blogs():
  global post_data
  score={}
  for id in post_data.keys():
    score[id]=post_data[id]['num_of_views']*100+post_data[id]['likes']*75+post_data[id]['comments']*50+post_data[id]['num_of_shares']*25
    days=post_data[id]['hours']//72
    hours=post_data[id]['hours']%72
    if days!=0 and hours!=0:
      score[id]=round(score[id]/(days), 4)
      score[id]=round(score[id]/hours, 4)
    elif days==0 and hours!=0:
      score[id]=round(score[id]/hours, 4)
    elif days!=0 and hours==0:
      score[id]=round(score[id]/(days), 4)
  return sorted(score.items(), key=lambda x: x[1],reverse=True)
 popularity_of_blogs()
