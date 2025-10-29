
 
 @client_bp.route('/assign', methods=['POST'])
def assign_architecte():
    """
    🔹 Assigne un architecte à un client
    📥 JSON attendu :
    {
        "client_id": 6,
        "architecte_id": 6
    }
    """
    data = request.get_json()
    client_id = data.get('client_id')
    architecte_id = data.get('architecte_id')

    if not client_id or not architecte_id:
        return jsonify({'error': 'client_id and architecte_id are required'}), 400

    result = ClientService.assign_architecte_to_client(client_id, architecte_id)

    if result is None:
        return jsonify({'error': 'Client or Architecte not found'}), 404

    return jsonify(result), 200

 @staticmethod
    def assign_architecte_to_client(client_id, architecte_id):
        client = Client.query.get(client_id)
        architecte = Architecte.query.get(architecte_id)
        if not client or not architecte:
            return None
        client.architecte_id = architecte_id
        db.session.commit()
        return client.to_dict()

@client_bp.route('/assign', methods=['POST'])
def assign_architecte():
    # Récupérer les données de la requête
    data = request.get_json()
    client_id = data.get('client_id')
    architecte_id = data.get('architecte_id')

    # Vérifier que les IDs sont fournis
    if not client_id or not architecte_id:
        return jsonify({'error': 'client_id and architecte_id are required'}), 400

    # Appeler la méthode statique pour assigner l'architecte au client
    result = ClientService.assign_architecte_to_client(client_id, architecte_id)

    # Vérifier le résultat
    if result is None:
        return jsonify({'error': 'Client or Architecte not found'}), 404

    # Retourner le résultat
    return jsonify(result.to_dict()), 200

### Assign an architect to a client
POST http://localhost:5000/api/clients/assign
Content-Type: application/json

{
  "client_id": 6,
  "architecte_id": 6
}

erreur 404

# Structure d'un projet Flask pour une API avec plusieurs tables et relations

## Introduction
Ce document décrit une structure modulaire pour développer une API avec **Flask**, adaptée à la gestion de plusieurs tables et relations (One-to-Many, Many-to-Many) avec **Flask-SQLAlchemy**. Il inclut également l’utilisation d’**Alembic** pour gérer les migrations de base de données.

## Structure du projet

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
├── migrations/
│   ├── env.py
│   ├── script.py.mako
│   ├── README
│   └── versions/
│       ├── <migration_id>_initial.py
│       └── <migration_id>_add_phone_to_user.py
├── requirements.txt
├── run.py
├── .env
├── .gitignore
└── README.md

## Pourquoi utiliser Alembic ?

1. **Gestion des migrations** : Applique les modifications du schéma (ex. : ajout de colonnes, tables) sans perte de données.
2. **Versionnement** : Suit l’historique des changements avec des scripts versionnés.
3. **Synchronisation** : Assure la cohérence du schéma entre les environnements (dev, test, prod).
4. **Automatisation** : Génère automatiquement des scripts de migration avec `--autogenerate`.
5. **Support des relations** : Gère les clés étrangères et les tables d’association (ex. : Many-to-Many).

## Configuration d’Alembic

1. **Installer Alembic** :
   ```bash
   pip install alembic

Initialiser Alembic :bash

alembic init migrations

Configurer migrations/env.py :python

from app import create_app, db
from app.models.user import User
from app.models.product import Product
from app.models.order import Order

app = create_app()
connectable = app.config["SQLALCHEMY_DATABASE_URI"]

with app.app_context():
    from alembic import context
    context.configure(
        connection=db.engine,
        target_metadata=db.metadata,
        compare_type=True,
        dialect_opts={"paramstyle": "named"},
    )

    with context.begin_transaction():
        context.run_migrations()

Configurer alembic.ini :

[alembic]
sqlalchemy.url = sqlite:///app.db

Créer et appliquer une migration :bash

alembic revision --autogenerate -m "Initial migration"
alembic upgrade head

Exemple de code1. app/models/user.pypython

from ..database.db import db

class User(db.Model):
    __tablename__ = "users"
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    phone = db.Column(db.String(20), nullable=True)
    orders = db.relationship("Order", backref="user", lazy="select")

    def to_dict(self):
        return {
            "id": self.id,
            "name": self.name,
            "email": self.email,
            "phone": self.phone,
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

6. app/__init__.pypython

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

    return app

7. requirements.txt

flask
flask-sqlalchemy
python-dotenv
pytest
marshmallow
alembic

Étapes pour lancer le projetCréez un environnement virtuel :bash

python -m venv venv
source venv/bin/activate  # Linux/Mac
venv\Scripts\activate     # Windows

Installez les dépendances :bash

pip install -r requirements.txt

Configurez les variables d'environnement dans .env :

DATABASE_URL=sqlite:///app.db
SECRET_KEY=your-secret-key

Initialisez et appliquez les migrations :bash

alembic init migrations
alembic revision --autogenerate -m "Initial migration"
alembic upgrade head

Lancez l'application :bash

python run.py

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

Bonnes pratiquesModularité : Séparez les modèles, routes, services, et schémas par entité.
Relations : Utilisez db.relationship pour One-to-Many et db.Table pour Many-to-Many.
Alembic : Vérifiez les scripts de migration générés et testez-les avant production.
Validation : Utilisez Marshmallow pour valider et sérialiser les données.
Tests : Écrivez des tests pour les relations et les migrations.



