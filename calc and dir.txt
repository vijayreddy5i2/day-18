from fastapi import FastAPI, HTTPException, Query
from pydantic import BaseModel
import os

app = FastAPI()

class CreateDirectoryRequest(BaseModel):
    path: str

class ListFilesRequest(BaseModel):
    path: str

class CalculatorRequest(BaseModel):
    operation: str
    num1: float
    num2: float

@app.post("/create_directory/")
async def create_directory(request: CreateDirectoryRequest):
    try:
        os.makedirs(request.path, exist_ok=True)
        return {"message": f"Directory '{request.path}' created successfully."}
    except Exception as e:
        raise HTTPException(status_code=400, detail=str(e))

@app.post("/list_files/")
async def list_files(request: ListFilesRequest):
    try:
        if not os.path.isdir(request.path):
            raise HTTPException(status_code=404, detail="Directory not found")
        files = os.listdir(request.path)
        return {"files": files}
    except Exception as e:
        raise HTTPException(status_code=400, detail=str(e))

@app.post("/calculate/")
async def calculate_post(request: CalculatorRequest):
    return calculate(request.num1, request.num2, request.operation)

@app.get("/calculate/")
async def calculate_get(
    num1: float = Query(..., description="The first number"),
    num2: float = Query(..., description="The second number"),
    operation: str = Query(..., description="The operation: add, sub, mul, div")
):
    return calculate(num1, num2, operation)

def calculate(num1: float, num2: float, operation: str):
    operation = operation.lower()
    if operation == "add":
        result = num1 + num2
    elif operation == "sub":
        result = num1 - num2
    elif operation == "mul":
        result = num1 * num2
    elif operation == "div":
        if num2 == 0:
            raise HTTPException(status_code=400, detail="Division by zero is not allowed.")
        result = num1 / num2
    else:
        raise HTTPException(status_code=400, detail="Invalid operation. Supported operations are add, sub, mul, and div.")
    return {"result": result}

# Main entry point for the application
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="127.0.0.1", port=8000)
