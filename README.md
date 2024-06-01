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


app/views/addition.py

from fastapi import APIRouter
from app.models.request import AdditionRequest
from app.models.response import AdditionResponse
from app.controllers.addition_controller import perform_addition

router = APIRouter()

def perform_addition(addition_request: AdditionRequest) -> AdditionResponse:
    started_at = datetime.now()
    results = process_addition(addition_request.payload)
    completed_at = datetime.now()
    return AdditionResponse(
        batchid=addition_request.batchid,
        response=results,
        status="complete",
        started_at=started_at,
        completed_at=completed_at
    )

@router.post("/add", response_model=AdditionResponse)
async def add_numbers(addition_request: AdditionRequest):
    return perform_addition(addition_request)

 Write Unit Tests
tests/test_addition.py

from fastapi.testclient import TestClient
from app.main import app
from datetime import datetime

client = TestClient(app)

def test_add_numbers():
    response = client.post("/add", json={"batchid": "id0101", "payload": [[1, 2], [3, 4]]})
    assert response.status_code == 200
    data = response.json()
    assert data["batchid"] == "id0101"
    assert data["response"] == [3, 7]
    assert data["status"] == "complete"
    assert "started_at" in data
    assert "completed_at" in data


def test_add_numbers_invalid_input():
    response = client.post("/add", json={"batchid": "id0101", "payload": [["a", "b"], [3, 4]]})
    assert response.status_code == 422




    
