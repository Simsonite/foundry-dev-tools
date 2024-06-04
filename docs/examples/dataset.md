# Dataset

### Uploading a dataset to Foundry

Saves a Pandas or PySpark dataframe to Foundry.

````{tab} v2
```python
from foundry_dev_tools import FoundryContext
import pandas as pd

df = pd.DataFrame({"a": [0, 1, 2], "b": [1, 2, 3]})
ctx = FoundryContext()
dataset = ctx.get_dataset_by_path("/path/to/test_output_dataset", create_if_not_exist=True)
dataset.save_dataframe(df)
```
````

````{tab} v1
```python
from foundry_dev_tools import CachedFoundryClient

cached_client = CachedFoundryClient()
cached_client.save_dataset(df, '/path/to/test_output_dataset',
                           branch='master', exists_ok=True, mode='SNAPSHOT')
```
````
### Uploading a folder to a dataset in Foundry

Upload the complete content of a local folder to a dataset in Foundry
````{tab} v2
```python
from foundry_dev_tools import FoundryContext
from pathlib import Path

ctx = FoundryContext()
dataset = ctx.get_dataset_by_path("/path/to/test_folder_upload", create_if_not_exist=True)
dataset.upload_folder(Path("/path/to/folder-to-upload"))
```
````

````{tab} v1
```python
import os
from foundry_dev_tools import FoundryRestClient

upload_folder = "/path/to/folder-to-upload"
target_dataset_path = "/path/to/test_folder_upload"

filenames = os.listdir(upload_folder)
filepaths = list(map(lambda file: os.sep.join([upload_folder, file]), filenames))
dataset_paths_in_foundry = list(map(lambda file: file, filenames))
path_file_dict = dict(zip(dataset_paths_in_foundry, filepaths))
rest_client = FoundryRestClient()
dataset_rid = rest_client.get_dataset_rid(dataset_path=target_dataset_path)
transaction_rid = rest_client.open_transaction(dataset_rid=dataset_rid,
                                               mode='UPDATE',
                                               branch='master')
rest_client.upload_dataset_files(dataset_rid=dataset_rid,
                                 transaction_rid=transaction_rid,
                                 path_file_dict=path_file_dict)
rest_client.commit_transaction(dataset_rid, transaction_rid)
```
````
### Save model or other type of python object

````{tab} v2
```python
from foundry_dev_tools import FoundryContext
from pathlib import Path

model_obj = """<PMML xmlns="http://www.dmg.org/PMML-4_1" version="4.1"></PMML>"""
ctx = FoundryContext()
dataset = ctx.get_dataset_by_path("/path/to/playground/model1", create_if_not_exist=True)
dataset.save_model(model_obj,file_path_in_dataset="model.pickle")

```
````

````{tab} v1
```python
from foundry_dev_tools import CachedFoundryClient
import pickle

model_obj = """<PMML xmlns="http://www.dmg.org/PMML-4_1" version="4.1"></PMML>"""
cached_client = CachedFoundryClient()
cached_client.save_model(model_obj, dataset_path_or_rid='/path/to/playground/model1',
                         branch='master', exists_ok=True, mode='SNAPSHOT')
```
````

### Load model or other type of file (using temporary file)
````{tab} v1
```python
from foundry_dev_tools import FoundryContext
from pathlib import Path
import pickle

ctx = FoundryContext()
dataset = ctx.get_dataset_by_path("/path/to/playground/model1")
model_file = dataset.download_file(output_directory=Path("/tmp/model"),path_in_dataset="model.pickle")

with model_file.open("rb") as model:
    print(pickle.load(model))

```
````

````{tab} v2
```python
from foundry_dev_tools import FoundryRestClient
import pickle

rest_client = FoundryRestClient()
rid = rest_client.get_dataset_rid('/path/to/playground/model1')
model_file = rest_client.download_dataset_files(dataset_rid=rid, output_directory='/tmp/model', branch='master')[0]

with open(model_file, 'rb') as file:
    print(pickle.load(file))
```
````

### Load model or other type of file (in-memory)

````{tab} v2
```python
from foundry_dev_tools import FoundryContext
import pickle

ctx = FoundryContext()
dataset = ctx.get_dataset_by_path("/path/to/playground/model1")
model_file_bytes = dataset.get_file("model.pickle")
print(pickle.loads(model_file_bytes))

```
````

````{tab} v1
```python
from foundry_dev_tools import FoundryRestClient
import pickle

rest_client = FoundryRestClient()
rid = rest_client.get_dataset_rid('/path/to/playground/model1')
model_file_bytes = rest_client.download_dataset_file(dataset_rid=rid,
                                                     output_directory=None,
                                                     foundry_file_path='model.pickle',
                                                     view='master')
print(pickle.loads(model_file_bytes))
```
````
### Download a dataset to a temporary folder

Downloads to a temporary folder and reading parquet dataset with pandas/pyarrow. When exiting the context, the temp files are automatically deleted.

````{tab} v2
```python
from foundry_dev_tools import FoundryContext
import pandas as pd

ctx = FoundryContext()
dataset = ctx.get_dataset("ri.foundry.main.dataset...")
with dataset.download_files_temporary() as tmp_dir:
    df = pd.read_parquet(tmp_dir)

print(df.shape)
```
````

````{tab} v1
```python
from foundry_dev_tools import FoundryRestClient
import pandas as pd

rest_client = FoundryRestClient()
rid = "ri.foundry.main.dataset.xxxxxxx-xxxx-xxx-xx-xxxxxxxxxx"
with rest_client.download_dataset_files_temporary(dataset_rid=rid, view='master') as temp_folder:
    df = pd.read_parquet(temp_folder)

print(df.shape)
```
````

### Download only few files from dataset

You can simply specify the list of files you want to download in download_dataset_files
````{tab} v2
```python
rid = "ri.foundry.main.dataset.xxxxxxx-xxxx-xxx-xxx-xxxxxxxxx"
ds = ctx.get_dataset(rid)
ds.download_files(output_directory=Path("/path/to/only_few_files"),paths_in_dataset={"file1.png","file2.png"})
```
````

````{tab} v1
```python
rid = "ri.foundry.main.dataset.xxxxxxx-xxxx-xxx-xxx-xxxxxxxxx"
rest_client.download_dataset_files(dataset_rid=rid, output_directory='/paht/to/only_few_files', files=['file1.png', 'file2.png'], branch='master')
```
````

### Polars DataFrame from Spark SQL dialect

Queries the Foundry SQL server with Spark SQL dialect, load arrow stream using [polars](https://www.pola.rs/).

````{tab} v2
```python
from foundry_dev_tools import FoundryContext
import polars as pl

ctx = FoundryContext()
ds = ctx.get_dataset_by_path("/path/to/test_dataset")
arrow_table = ds.query_foundry_sql("SELECT *",return_type="arrow")

df = pl.from_arrow(arrow_table)
print(df)
```
````

````{tab} v1
```python
from foundry_dev_tools import FoundryRestClient

rest_client = FoundryRestClient()
arrow_table = rest_client.query_foundry_sql(
    "SELECT * FROM `/path/to/test_dataset`",
    branch="master",
    return_type="arrow",
)
import polars as pl

df = pl.from_arrow(arrow_table)
print(df)
```
````

### DuckDB Table from Spark SQL dialect

Queries the Foundry SQL server with Spark SQL dialect, load arrow stream using [duckdb](https://duckdb.org/).

````{tab} v2
```python
from foundry_dev_tools import FoundryContext

ctx = FoundryContext()
ds = ctx.get_dataset_by_path("/path/to/test_dataset")
arrow_table = ds.to_arrow()

import duckdb

# Get an in-memory DuckDB database and create a new table from the result arrow table.
# Note that the python variable is automatically determined from the query string.
con = duckdb.connect()
con.execute("CREATE TABLE my_table AS SELECT * FROM arrow_table")
```
````

````{tab} v1
```python
from foundry_dev_tools import FoundryRestClient

rest_client = FoundryRestClient()
arrow_table = rest_client.query_foundry_sql(
    "SELECT * FROM `/path/to/test_dataset`",
    branch="master",
    return_type="arrow",
)
import duckdb

# Get an in-memory DuckDB database and create a new table from the result arrow table.
# Note that the python variable is automatically determined from the query string.
con = duckdb.connect()
con.execute("CREATE TABLE my_table AS SELECT * FROM arrow_table")
```
````