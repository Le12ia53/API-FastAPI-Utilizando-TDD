# API-FastAPI-Utilizando-TDD
Instalação das Dependências com Poetry
poetry init --no-interaction
poetry add fastapi uvicorn pymongo pydantic pytest
Código Básico: Modelo, Schema e Rota
models.py (Pydantic models)
from pydantic import BaseModel, Field
from datetime import datetime
from bson import ObjectId

class PyObjectId(ObjectId):
    @classmethod
    def __get_validators__(cls):
        yield cls.validate

    @classmethod
    def validate(cls, v):
        if not ObjectId.is_valid(v):
            raise ValueError("Invalid object ID")
        return ObjectId(v)

class ProductModel(BaseModel):
    id: PyObjectId = Field(default_factory=PyObjectId, alias="_id")
    name: str
    price: float
    description: str = None
    updated_at: datetime = Field(default_factory=datetime.now)

    class Config:
        arbitrary_types_allowed = True
        json_encoders = {ObjectId: str}

routes/product.py (FastAPI routes)
from fastapi import APIRouter, HTTPException, Body
from ..models import ProductModel

router = APIRouter()

@router.post("/products/", response_model=ProductModel)
async def create_product(product: ProductModel):
    # Simulate DB insertion
    return product

@router.get("/products/")
async def list_products():
    # Simulate fetching from DB
    return [ProductModel(name="Example", price=1000)]

@router.get("/products/{product_id}", response_model=ProductModel)
async def get_product(product_id: str):
    # Simulate fetching specific product
    return ProductModel(name="Example", price=1000)

@router.put("/products/{product_id}", response_model=ProductModel)
async def update_product(product_id: str, product: ProductModel):
    # Simulate updating product
    return product

@router.delete("/products/{product_id}")
async def delete_product(product_id: str):
    # Simulate deleting product
    return {"message": "Product deleted successfully"}

Testes com Pytest
test_product.py

from fastapi.testclient import TestClient
from ..app.main import app

client = TestClient(app)

def test_create_product():
    response = client.post("/products/", json={"name": "New Product", "price": 5000})
    assert response.status_code == 200
    assert response.json()["price"] == 5000

def test_list_products():
    response = client.get("/products/")
    assert response.status_code == 200
    assert isinstance(response.json(), list)

def test_get_product():
    response = client.get("/products/1")
    assert response.status_code == 200

def test_update_product():
    response = client.put("/products/1", json={"name": "Updated Product", "price": 5500})
    assert response.status_code == 200
    assert response.json()["name"] == "Updated Product"

def test_delete_product():
    response = client.delete("/products/1")
    assert response.status_code == 200
    assert response.json() == {"message": "Product deleted successfully"}

