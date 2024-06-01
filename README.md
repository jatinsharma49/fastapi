Project Structure

fastapi_project/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── models/
│   │   ├── __init__.py
│   │   ├── request.py
│   │   ├── response.py
│   ├── views/
│   │   ├── __init__.py
│   │   ├── addition.py
│   ├── utils/
│   │   ├── __init__.py
│   │   ├── multiprocessor.py
│   │   ├── logger.py
├── tests/
│   ├── __init__.py
│   ├── test_addition.py
├── requirements.txt: 


**
Create FastAPI Application**
app/main.py

from fastapi import FastAPI
from app.views import addition

app = FastAPI()

app.include_router(addition.router)

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
    
Create Pydantic Models for Request and Response Validation
app/models/request.py

from pydantic import BaseModel, Field
from typing import List

class AdditionRequest(BaseModel):
    batchid: str = Field(..., example="id0101")
    payload: List[List[int]] = Field(..., example=[[1, 2], [3, 4]])

app/models/response.py
from pydantic import BaseModel

from pydantic import BaseModel
from datetime import datetime
from typing import List

class AdditionResponse(BaseModel):
    batchid: str
    response: List[int]
    status: str
    started_at: datetime
    completed_at: datetime


the Logic for Addition Using Multiprocessing

app/utils/multiprocessor.py
from multiprocessing import Pool
import logging

logger = logging.getLogger(__name__)

def add_numbers(numbers):
    return sum(numbers)

def process_addition(payload: list):
    try:
        with Pool(processes=4) as pool:
            results = pool.map(add_numbers, payload)
            return results
    except Exception as e:
        logger.error(f"Error in processing addition: {e}")
        raise e

 Error Handling and Logging
app/utils/logger.py

import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def get_logger():
    return logger





    
