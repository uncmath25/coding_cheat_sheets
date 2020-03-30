# Python

### Web Scraping
``` python
import requests

r = requests.get('https://api.github.com/events')
json_data = r.json()
r = requests.post('http://httpbin.org/post', data = {'key': 'value'})

payload = {'key1': 'value1', 'key2': ['value2', 'value3']}
r = requests.get('http://httpbin.org/get', params=payload)
html_string = r.text

# stream response to file
with open('file', 'wb') as f:
    for chunk in r.iter_content(chunk_size=128):
        f.write(chunk)
```
Use the following for persistent html connections
``` python
session = requests.session()
r = session.get('https://api.github.com/events')
```
Authentication
``` python
HEAD = {'Authorization': 'Bearer {0}'.format(TOKEN)}
response = requests.get(url, headers=HEAD)
```

### Plotting
**Scatter Plot**
``` python
import matplotlib.pyplot as plt
import numpy as np

n = 10
amp = 1
x = list(range(n))
y = [z + amp*(2*np.random.random() - 1) for z in x]

plt.rcParams["figure.figsize"] = (16, 5)
plt.xlim(-amp, (n-1)+amp)
plt.ylim(-amp, (n-1)+amp)
plt.scatter(x, y, color='blue', alpha=0.8, label='data')
plt.plot(x, x, color='green', alpha=0.8, label='fit')
plt.xlabel('x Label')
plt.ylabel('y Label')
plt.title('Regression Line')
plt.legend(loc='best')
plt.xticks(np.linspace(0, n, n+1))
plt.savefig('output.png', format='png', dpi=100)
plt.show()
```
**Histogram**
``` python
import matplotlib.pyplot as plt
import numpy as np

x = list(np.random.normal(0, 1, 100))

plt.rcParams["figure.figsize"] = (16, 5)
BOUNDS = [-2, 2]
BIN_COUNT = 100
plt.xlim(BOUNDS[0], BOUNDS[1])
BINS = list(np.arange(BOUNDS[0], BOUNDS[1] + 1/BIN_COUNT, (BOUNDS[1] - BOUNDS[0])/BIN_COUNT))
plt.hist(x, bins=BINS, color='blue', alpha=0.5)
plt.xlabel('Standard Deviations')
plt.ylabel('Frequency')
plt.title('Normal Sample Distribution')
plt.show()
```
**Subplots**
``` python
import matplotlib.pyplot as plt
import numpy as np

fig = plt.figure()
sub_1 = fig.add_subplot(2, 2, 1)
sub_2 = fig.add_subplot(2, 2, 2)
sub_3 = fig.add_subplot(2, 2, 3)
sub_1.hist(np.random.randn(100), bins=20, color='b', alpha=0.3)
sub_2.scatter(np.arange(30), np.arange(30) + 3*np.arange(30))
sub_3.plot(np.arange(30), np.arange(30) + 3*np.arange(30), linestyle='--')
plt.show()
```
**3-D**
``` python
import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D
import numpy as np

fig = plt.figure()
ax = Axes3D(fig)
ax.scatter(list(np.random.normal(0, 1, 100)), list(np.random.normal(0, 1, 100)), list(np.random.normal(0, 1, 100)))
plt.show()
```

### MySQL
``` python
import pymysql

conn = pymysql.connect(user=SQL_USERNAME, passwd=SQL_PASSWORD, host=DATABASE_IP_ADDRESS, port=DATABASE_PORT_NUMBER, charset='utf8', local_infile=True)

cur = conn.cursor()
cur.execute(query)
data = cur.fetchall()

column_names = [str(row[0]) for row in cur.description] # gives column names from source table

cur.close()
conn.commit()
conn.close()
```

### AWS
**S3**
``` python
import json
import boto3
data = {'id': 1, 'name': 'colton'}
client = boto3.client('s3')
client.put_object(Body=json.dumps(data), Bucket=BUCKET, Key=PATH)
```

### Regex
``` python
import re
ws_regex = re.compile(r'\s+')
# substitute the token into the ws matched pattern
ws_regex.sub("replacement_token", "source_string")
# returns leftovers by splitting on ws
ws_regex.split('source_string')
# returns list of all ws matched patterns
ws_regex.findall('source_string')
# use without compile regex
re.split(r'\s+', 'source_string')
```

### Misc
Make directory if it doesn't exist
``` python
import os
if not os.path.exists(directory):
    os.makedirs(directory)
```
Suppress output to console
``` python
import os
import sys
sys.stderr = open(os.devnull, "w")
# CODE
sys.stdout = sys.__stderr__
```
JSON
``` python
import json
json_dict = json.loads('JSON_STRING')
json_string = json.dumps(json_dict, sort_keys=True, indent=4)
```
Format time
``` python
import datetime
import time
datetime.datetime.utcfromtimestamp(time.time()).strftime('%Y-%m-%d %H:%M:%S')
```
Download the tar file locally
``` python
import os
import tarfile
with tarfile.open('temp.tar.gz') as f:
    f.extractall('temp')
untar_files = os.listdir(os.path.join('temp', os.listdir('temp')[0]))
```
Pearson Correlation Coefficient
``` python
import numpy as np
import scipy.stats
from scipy.stats.stats import pearsonr
# returns (r, p) where p is the probability that an uncorrelated system produces results at least as extreme as these
pearsonr(x, y)
np.corrcoef(x, y)[0, 1]
```
Unzipping HTTP JSON
``` python
import json
import requests
import zlib
URL = 'http://files.tmdb.org/p/exports/movie_ids_02_25_2019.json.gz'
zipped_bytes = requests.get(URL).content
unzipped_bytes = zlib.decompress(zipped_bytes, 15 + 32)
data = [dict(json.loads(s)) for s in unzipped_bytes.decode('utf8').split('\n')[:-1]]
```
Reference file relative to script's location
``` python
import os
path = os.path.join(os.path.dirname(__file__), 'PATH_NAME')
```

### Virtual Environment
``` bash
pip3 install virtualenv
virtualenv --python=$(which python3) ENV # ENV is local directory
source ENV/bin/activate # activate environment
pip3 install PACKAGE_NAME" # install in environment
pip3 uninstall -y PACKAGE_NAME"
pip3 list
pip3 freeze > requirements.txt #cache the currently installed packages with version numbers
pip3 install -r requirements.txt
deactivate
```

### SQLAlchemy Example
``` python
from sqlalchemy import create_engine, Column, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

db_string = 'sqlite:///DATABASE'
db = create_engine(db_string)
base = declarative_base()

class Movie(base):
    __tablename__ = 'movies'

    title = Column(String, primary_key=True)
    director = Column(String)
    year = Column(String)

Session = sessionmaker(db)
session = Session()

base.metadata.create_all(db)

# CREATE
batman_begins = Movie(title='Batman Begins', director='Christopher Nolan', year='2005')
session.add(batman_begins)
session.commit()

# READ
movies = session.query(Movie)
for movie in movies:
    print(movie.title)

# UPDATE
batman_begins.title = 'Batman Ends'
session.commit()

# DELETE
session.delete(batman_begins)
session.commit()
```
