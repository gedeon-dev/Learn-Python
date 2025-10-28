my_flask_api/
├── app/
│   ├── init.py
│   ├── routes/
│   │   ├── init.py
│   │   ├── user_routes.py
│   │   ├── product_routes.py
│   │   └── order_routes.py
│   ├── models/
│   │   ├── init.py
│   │   ├── user.py
│   │   ├── product.py
│   │   └── order.py
│   ├── services/
│   │   ├── init.py
│   │   ├── user_service.py
│   │   ├── product_service.py
│   │   └── order_service.py
│   ├── config/
│   │   ├── init.py
│   │   └── config.py
│   ├── database/
│   │   ├── init.py
│   │   └── db.py
│   ├── utils/
│   │   ├── init.py
│   │   └── helpers.py
│   ├── schemas/
│   │   ├── init.py
│   │   ├── user_schema.py
│   │   ├── product_schema.py
│   │   └── order_schema.py
│   └── tests/
│       ├── init.py
│       ├── test_user_routes.py
│       ├── test_product_routes.py
│       └── test_order_routes.py
├── requirements.txt
├── run.py
├── .env
├── .gitignore
└── README.md

## Explication des dossiers et fichiers

1. **Racine du projet (`my_flask_api/`)**
   - `requirements.txt` : Liste des dépendances Python (ex. : flask, flask-sqlalchemy, python-dotenv, pytest, marshmallow).
   - `run.py` : Point d'entrée pour lancer l'application.
   - `.env` : Variables d'environnement (ex. : DATABASE_URL, SECRET_KEY).
   - `.gitignore` : Ignore les fichiers inutiles (ex. : __pycache__, .env).
   - `README.md` : Documentation du projet.

2. **Dossier `app/`**
   - `__init__.py` : Initialise l'application Flask, configure SQLAlchemy, et enregistre les Blueprints.

3. **Dossier `models/`**
   - Contient les modèles SQLAlchemy pour chaque table (ex. : `user.py`, `product.py`, `order.py`).
   - Définit les relations entre tables (One-to-Many, Many-to-Many).

4. **Dossier `routes/`**
   - Contient les endpoints REST, organisés par entité avec des Blueprints (ex. : `user_routes.py`).

5. **Dossier `services/`**
   - Contient la logique métier pour chaque entité (ex. : `user_service.py`).

6. **Dossier `schemas/` (optionnel)**
   - Contient les schémas Marshmallow pour sérialiser/désérialiser les données.

7. **Dossier `database/`**
   - Gère la connexion à la base de données via `db.py`.

8. **Dossier `utils/`**
   - Contient des fonctions utilitaires (ex. : gestion des erreurs).

9. **Dossier `tests/`**
   - Contient les tests unitaires et d'intégration pour chaque entité.

## Gestion des relations entre tables

### Relations One-to-Many
- Exemple : Un `User` peut avoir plusieurs `Order` (un utilisateur peut passer plusieurs commandes).
- Définie avec `db.relationship` et une clé étrangère (`ForeignKey`).

### Relations Many-to-Many
- Exemple : Une `Order` peut inclure plusieurs `Product`, et un `Product` peut apparaître dans plusieurs `Order`.
- Nécessite une table d'association (ex. : `order_product`).

## Exemple de code

### 1. `app/models/user.py`
```python
from ..database.db import db

class User(db.Model):
    __tablename__ = "users"
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    orders = db.relationship("Order", backref="user", lazy="select")

    def to_dict(self):
        return {
            "id": self.id,
            "name": self.name,
            "email": self.email,
            "orders": [order.to_dict() for order in self.orders]
        }

2. app/models/product.pypython

from ..database.db import db

class Product(db.Model):
    __tablename__ = "products"
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    price = db.Column(db.Float, nullable=False)

    def to_dict(self):
        return {"id": self.id, "name": self.name, "price": self.price}

3. app/models/order.pypython

from ..database.db import db

order_product = db.Table(
    "order_product",
    db.Column("order_id", db.Integer, db.ForeignKey("orders.id"), primary_key=True),
    db.Column("product_id", db.Integer, db.ForeignKey("products.id"), primary_key=True)
)

class Order(db.Model):
    __tablename__ = "orders"
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey("users.id"), nullable=False)
    products = db.relationship(
        "Product",
        secondary=order_product,
        backref=db.backref("orders", lazy="select"),
        lazy="select"
    )
    quantity = db.Column(db.Integer, nullable=False)

    def to_dict(self):
        return {
            "id": self.id,
            "user_id": self.user_id,
            "products": [product.to_dict() for product in self.products],
            "quantity": self.quantity
        }

4. app/services/order_service.pypython

from ..models.order import Order
from ..models.user import User
from ..models.product import Product
from ..database.db import db

def create_order(user_id, product_ids, quantity):
    user = User.query.get_or_404(user_id)
    products = Product.query.filter(Product.id.in_(product_ids)).all()
    if not products or len(products) != len(product_ids):
        raise ValueError("Un ou plusieurs produits n'existent pas")
    order = Order(user_id=user_id, quantity=quantity)
    order.products = products
    db.session.add(order)
    db.session.commit()
    return order

def get_all_orders():
    return Order.query.all()

def get_order_by_id(order_id):
    return Order.query.get_or_404(order_id)

5. app/routes/order_routes.pypython

from flask import Blueprint, jsonify, request
from ..services.order_service import create_order, get_all_orders, get_order_by_id
from ..schemas.order_schema import OrderSchema

order_bp = Blueprint("order", __name__, url_prefix="/api/orders")
order_schema = OrderSchema()
orders_schema = OrderSchema(many=True)

@order_bp.route("/", methods=["GET"])
def get_orders():
    orders = get_all_orders()
    return jsonify(orders_schema.dump(orders))

@order_bp.route("/", methods=["POST"])
def add_order():
    data = request.get_json()
    user_id = data.get("user_id")
    product_ids = data.get("product_ids")
    quantity = data.get("quantity")
    try:
        order = create_order(user_id, product_ids, quantity)
        return jsonify(order_schema.dump(order)), 201
    except ValueError as e:
        return jsonify({"error": str(e)}), 400

@order_bp.route("/<int:order_id>", methods=["GET"])
def get_order(order_id):
    order = get_order_by_id(order_id)
    return jsonify(order_schema.dump(order))

6. app/schemas/order_schema.pypython

from marshmallow import Schema, fields
from .user_schema import UserSchema
from .product_schema import ProductSchema

class OrderSchema(Schema):
    id = fields.Int(dump_only=True)
    user_id = fields.Int(required=True)
    user = fields.Nested(UserSchema, dump_only=True)
    products = fields.List(fields.Nested(ProductSchema), dump_only=True)
    quantity = fields.Int(required=True)

7. app/__init__.pypython

from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from .config.config import Config
from .database.db import db

def create_app():
    app = Flask(__name__)
    app.config.from_object(Config)
    db.init_app(app)

    from .routes.user_routes import user_bp
    from .routes.product_routes import product_bp
    from .routes.order_routes import order_bp
    app.register_blueprint(user_bp)
    app.register_blueprint(product_bp)
    app.register_blueprint(order_bp)

    with app.app_context():
        db.create_all()

    return app

8. app/database/db.pypython

from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

9. app/config/config.pypython

from os import environ
from dotenv import load_dotenv

load_dotenv()

class Config:
    SQLALCHEMY_DATABASE_URI = environ.get("DATABASE_URL") or "sqlite:///app.db"
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    SECRET_KEY = environ.get("SECRET_KEY") or "your-secret-key"

10. run.pypython

from app import create_app

app = create_app()

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=True)

11. requirements.txt

flask
flask-sqlalchemy
python-dotenv
pytest
marshmallow

12. .env

DATABASE_URL=sqlite:///app.db
SECRET_KEY=your-secret-key

Exemple d’utilisationCréer un utilisateurbash

curl -X POST http://localhost:5000/api/users \
-H "Content-Type: application/json" \
-d '{"name": "John Doe", "email": "john@example.com"}'

Créer des produitsbash

curl -X POST http://localhost:5000/api/products \
-H "Content-Type: application/json" \
-d '{"name": "Laptop", "price": 999.99}'

curl -X POST http://localhost:5000/api/products \
-H "Content-Type: application/json" \
-d '{"name": "Mouse", "price": 29.99}'

Créer une commande avec plusieurs produitsbash

curl -X POST http://localhost:5000/api/orders \
-H "Content-Type: application/json" \
-d '{"user_id": 1, "product_ids": [1, 2], "quantity": 3}'

Réponse JSON attenduejson

{
  "id": 1,
  "user_id": 1,
  "user": {
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com"
  },
  "products": [
    {"id": 1, "name": "Laptop", "price": 999.99},
    {"id": 2, "name": "Mouse", "price": 29.99}
  ],
  "quantity": 3
}

Bonnes pratiquesModularité : Séparez les modèles, routes, services, et schémas par entité.
Relations :Utilisez db.relationship pour One-to-Many et db.Table pour Many-to-Many.
Optimisez avec lazy="select" ou lazy="joined".

Validation : Utilisez Marshmallow pour valider les données et gérer les relations dans les réponses JSON.
Tests : Écrivez des tests unitaires pour chaque entité et leurs relations.
Gestion des erreurs : Centralisez la gestion des erreurs dans utils/helpers.py.
Documentation : Documentez les endpoints dans README.md ou avec Swagger.

Étapes pour lancer le projetCréez un environnement virtuel :bash

python -m venv venv
source venv/bin/activate  # Linux/Mac
venv\Scripts\activate     # Windows

Installez les dépendances :bash

pip install -r requirements.txt

Configurez les variables d'environnement dans .env.
Lancez l'application :bash

python run.py

Testez les endpoints avec Postman ou curl.

Tests unitairesExemple : app/tests/test_order_routes.pypython

import pytest
from app import create_app

@pytest.fixture
def client():
    app = create_app()
    app.config["TESTING"] = True
    with app.app_context():
        from ..database.db import db
        db.create_all()
        yield app.test_client()
        db.drop_all()

def test_create_order(client):
    client.post("/api/users", json={"name": "John Doe", "email": "john@example.com"})
    client.post("/api/products", json={"name": "Laptop", "price": 999.99})
    response = client.post("/api/orders", json={"user_id": 1, "product_ids": [1], "quantity": 2})
    assert response.status_code == 201
    assert response.json["products"][0]["name"] == "Laptop"

class Architecte(db.Model):
    __tablename__ = 'architectes'
    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    nom = db.Column(db.String, nullable=False)
    prenom = db.Column(db.String, nullable=False)
    tel = db.Column(db.Integer)
    email = db.Column(db.String)
    tarif_horaire = db.Column(db.Integer, nullable=False)
    type = db.Column(db.String, nullable=False)
    clients = db.relationship('Client', backref='architecte', lazy="select")






