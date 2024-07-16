
# AsyncTester

AsyncTester is a Python-based tool designed to test the performance of an asynchronous endpoint by sending concurrent HTTP requests and measuring various metrics such as latency and throughput.

## Table of Contents

- [Introduction](#introduction)
- [Features](#features)
- [Installation](#installation)
- [Usage](#usage)
- [Configuration](#configuration)
- [Error Handling](#error-handling)
- [Contributing](#contributing)
- [License](#license)

## Introduction

AsyncTester is built using `asyncio`, `httpx`, and `PyYAML`. It allows users to load a payload from a YAML file and send concurrent HTTP POST requests to a specified endpoint. The results, including latency and responses, are stored and can be analyzed for performance benchmarking.

## Features

- Asynchronous HTTP requests
- Concurrent execution with configurable number of users
- Detailed logging of request statuses and responses
- Error handling for HTTP and JSON parsing errors
- Calculation of throughput (requests per second)
- YAML-based payload configuration

## Installation

1. Clone the repository:

git clone https://github.com/manichandana77/AsyncTester.git
cd AsyncTester

Create a virtual environment:

python -m venv env
source env/bin/activate  # On Windows, use `env\Scripts\activate`

Install the required packages:
pip install -r requirements.txt

Usage:
Create a YAML file with your payload data. 

Example:
yaml
Payload:
  - index: 1
    prompt: "How many resumes are there in your context?"
  - index: 2
    prompt: "May I know the email address of Siva?"
  - index: 3
    prompt: "Tell me about the skills that Vamsi has?"
  - index: 4
    prompt: "What project has Mani done?"
  - index: 5
    prompt: "What are the skills that Siva has?"

Create the main.py script to run the AsyncTester:
python

from fastapi import FastAPI, HTTPException, status
from pydantic import BaseModel
from async_tester import AsyncTester
from storejson import JSONHandler
from Excel import ExcelHandler
import uvicorn
import logging
import os

# Set up logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

app = FastAPI()

# Define the payload model for the request body
class Payload(BaseModel):
    payload_file_path: str
    number_of_users: int
    file_name: str
    excel_file_name: str

@app.post("/test-async-endpoint")
async def test_async_endpoint(payload: Payload):
    try:
        endpoint = "http://172.25.1.228:4001/accelerator/server"
        
        # Verify that the payload file path exists
        if not os.path.isfile(payload.payload_file_path):
            raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail=f"Payload file does not exist: {payload.payload_file_path}")
        
        # Initialize AsyncTester with provided payload
        tester = AsyncTester(payload.payload_file_path, payload.number_of_users, endpoint)
        # Run the test_async_endpoint method
        results = await tester.test_async_endpoint()
    
        # Save results to JSON
        handler = JSONHandler()
        json_data = handler.save_to_json(results, payload.file_name)
        
        # Convert JSON to Excel
        handler1 = ExcelHandler(payload.excel_file_name)
        try:
            handler1.json_to_excel(json_data)
        except HTTPException as e:
            print(f"HTTP Exception: {e}")
        except Exception as e:
            print(f"Unexpected Exception: {e}")
        
        return {"status": "success", "message": "Requests processed successfully"}
    
    except HTTPException as http_exc:
        logger.error(f"HTTP Exception: {http_exc.detail}")
        raise http_exc
    
    except ValueError as value_err:
        logger.error(f"Value Error: {value_err}")
        raise HTTPException(status_code=status.HTTP_400_BAD_REQUEST, detail=f"Invalid input: {value_err}")
    
    except FileNotFoundError as file_not_found_err:
        logger.error(f"File not found: {file_not_found_err}")
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail=f"File not found: {file_not_found_err}")
    
    except Exception as e:
        logger.error(f"Unexpected error: {e}")
        raise HTTPException(status_code=status.HTTP_500_INTERNAL_SERVER_ERROR, detail=f"Internal server error: {e}")

if __name__ == '__main__':
    uvicorn.run(app, host='0.0.0.0', port=5002)

Run the FastAPI application:

uvicorn main:app --reload

Send a POST request to the /test-async-endpoint endpoint with the following JSON payload:
json

{
  "payload_file_path": "path/to/your/payload.yml",
  "number_of_users": 10,
  "file_name": "output.json",
  "excel_file_name": "path/to/excel_directory"
}

Configuration:

payload_file_path: Path to the YAML file containing the payload data.
number_of_users: Number of concurrent users to simulate.
endpoint: The HTTP endpoint to send requests to.
file_name: Name of the JSON file to save the results.
excel_file_name: Path to the directory where the Excel file will be saved.

Error Handling
The AsyncTester script includes extensive error handling:

File Handling: Checks if the payload file exists and can be parsed correctly.
HTTP Requests: Handles request errors, HTTP status errors, and unexpected errors during the request.
JSON Parsing: Catches and logs JSON decoding errors.
Logging: Logs errors and important information for easier debugging.

Contributing:
Contributions are welcome! Please open an issue or submit a pull request for any improvements or bug fixes.

License:
This project is licensed under the MIT License.

