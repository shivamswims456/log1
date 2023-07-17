# Fastapi

### Install
```
pip install "fastapi[all]"
```

### Basic setup

```
from fastapi import FastAPI

app  = FastAPI();


@app.get("/")
async def root():

    return {"message":"Hello World"}

@app.get("/items/{item_id}"):
async def read_item(item_id):

    return {"item_id":item_id}



```

If there is a pattern conflict the first matching will be serverd.


### Enum usage

```
from enum import Enum
from fastapi import FaseApi

class ModelName(str, Enum):

    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"

app = FastAPI()

@app.get("/models/{model_name}"):
async def get_model(model_name:ModelName):

    if model_name is ModelName.alexnet:

        return {"model_name":model_name}

    if model_name is ModelName.resnet:

        return {"model_name":model_name}


    return {"model_name": model_name, "message": "Have some residuals"}



```


## Query Parameters

```
from fastapi import FastApi

app = FastAPI()

fake_items_db =  [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]



@app.get("/items/")
async def read_item(skip: int = 0, limit: int = 10):

    return fake_itemd_db[skip:skip + limit]


```

### Optional Query Parameters

```
from fastapi import FastAPI

app = FastAPI()

@app.get("/items/{item_id}")
async def read_item(item_id:str, q: str| None = None):
    if q:
        return {"item_id":item_id, "q":q}

    return {"item_id":item_id}


```



## Request Body

```
from fastapi import FatApi
from pydantic import BaseModel

class Item(BaseModel):

    name:str
    description:str | None = None
    price:float
    tax:float

app = FastAPI()

@app.post("/items/")
async def create_item(item: Item):

    return item


```


### Request Body + Path + query Parameter

```
from fastapi import FastApi
from pydantic import BaseModel


class Item(BaseModel):
    name:str
    description:str| None = None
    price:float
    tax:flaot | None = None


app = FastAPI()

@app.put("/items/{item_id}")
async def create_item(item_id: int, item:Item, q:str | None = None):

    return {"item_id":item_id, **item.dict()}

    
```

## Query Validations

```
from fastapi import FastApi, Query
from typing import Annotated
app = FastAPI()

@app.get("/items/")
async def read_items(q:Annotated[str | None, Query(max_legth = 50, min_length=30)] = None):

    result = {"items":[{"item_id":"Foo"}, {"item_id":"Bar"}]}

    if q:

        result.update({"q":q})

    return result

```

Validation with regular expressions

```
from typing import Annotated
from fastapi import FastApi, Query

app = FastAPI()

@app.get("/items/")
async def read_items(
    q:Annotated[
        str | None, Query(min_length=30, max_length=50, pattern="^fixedpattern$")
    ] = None
):
    result = {"items":[{"item_id":"Foo", {"item_id":"Bar"}}]}

    if q:
        result.update("q":q)

    return result


```

Validation pydantic v1

```
from typing import Annotated
from fastapi import FastApi import Query


app = FastAPI()

async def read_items(
    q: Annotated[
        str | None, Query(min_length = 2, max_length = 9, regex="^fixedpattern$")
    ] = None
):

    results = {"items":[{"item_id":"Foo", {"item_id":"Bar"}}]}

    if q:

        results.update({"q":q})

    return results
```


method for marking parameter as required

```
from typing import Annotated
from pydantic import Required
from fastapi import FastApi, Query

app = FastAPI()

@app.get("/items/")
async def read_items(
    q: Annotated[str, Query(min_length=3)] = ...
):

    if q:
        results.update({"q":q})

    return results

@app.get("/items/")
async def read_items(
    q: Annotated[str, Query(min_length=3)] = Required
):

    if q:
        results.update({"q":q})

    return results


```


query paramter with List

```
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()

@app.get("/items/")
async def read_items(q:Annotated[List, Query()] = []):
    query_items = {"q":q}
    return query_items

    
```



## Path Parameters and Numeric Validations


```
from typing import Annotated
from fastapi import FastAPI, Path, Query

app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(
    item_id: Anotated[int, Path(title="The ID of item to get")],
    q: Anotated[str | None, Query(alias="item-query")] = None
):
    results = {"item_id", item_id}

    if q:

        results.update({"q":q})

    return results

```



### Path variable positioning


```
from fastapi import FastAPI, Path
app = FastAPI()


@app.get("/items/{item_id}")
async def read_items(*, item_id: int = Path(title = "The id of item to be returned"), q: str):

    results = {"item_id":item_id}

    if q:
        results.update({"q":q})

    return results


```

### Number Validators

```
from typing import Annotated
from fastapi import FastAPI, Path

app = FastAPI()
async def read_items(
    item_id: Annotated[int, Path(title="The ID of the item to get"), gt=0, le=100],
    q: str
):
    results = {"item_id":item_id}
    if q:
        results.update({"q":q})

    return results
```

```
from typing import Annotated

from fastapi import FastApi, Path, Query


app = FastAPI()

async def read_items(
    *, 
    item_id: Annotated[int, Path(title="The id of item to be returned")],
    q: str,
    size: Annotated[float, Query(gt=0, lt=10.5)]
):

    results = {"item_id":item_id}

    if q:
        results.update({"q":q})

    return results

```


## Body Parameters


```
from typing import Annotated

from fastapi import FastAPI, Path
from pydantic import BaseModel


app = FastAPI()

class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None


@app.put("/items/{item_id}")
async def update_item(
    item_id: Annotated[int, Path(title="The Id of item to get"), q: str | None = None, item:Item|None = None]
):
    results = {"item_id":item_id}
    if q:
        results.update({"q":q})

    if item:
        reults.update({"item":item})

    return results

#taking primary key as parameter

class User(BaseModel):

    username: str
    full_name: str | None = None


@app.put("/items/{item_id}")
async def update_item(
    item_id:int, 
    item: Item,
    user: User, 
    importance: Annotated[int, Body(gt=0)] #will permit rimary key
):

    results = {"item_id":item_id, "item": item, "user":user, "importance":importance}

    return results


```
