# Structure d'un projet Flask pour une API avec plusieurs tables

## Introduction
Ce document décrit une structure modulaire et maintenable pour développer une API avec Flask, adaptée à la gestion de plusieurs tables dans une base de données. Chaque table est organisée avec ses propres modèles, routes, services, et tests pour garantir la scalabilité et la clarté du code.

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
├── requirements.txt
├── run.py
├── .env
├── .gitignore
└── README.md

## Explication des dossiers et fichiers

1. **Racine du projet (`my_flask_api/`)**
   - `requirements.txt` : Liste des dépendances Python (ex. : flask, flask-sqlalchemy).
   - `run.py` : Point d'entrée pour lancer l'application.
   - `.env` : Variables d'environnement (ex. : DATABASE_URL, SECRET_KEY).
   - `.gitignore` : Ignore les fichiers inutiles pour le versionnement.
   - `README.md` : Documentation du projet.

2. **Dossier `app/`**
   - Contient le code principal, organisé de manière modulaire.
   - `__init__.py` : Initialise l'application Flask et enregistre les Blueprints.

3. **Dossier `models/`**
   - Chaque table a son propre fichier (ex. : `user.py`, `product.py`, `order.py`).
   - Définit les modèles SQLAlchemy avec les relations entre tables.

4. **Dossier `routes/`**
   - Contient les endpoints de l'API, un fichier par entité (ex. : `user_routes.py`).
   - Utilise des Blueprints Flask pour modulariser les routes.

5. **Dossier `services/`**
   - Contient la logique métier pour chaque entité (ex. : `user_service.py`).

6. **Dossier `schemas/` (optionnel)**
   - Contient les schémas Marshmallow pour la sérialisation/désérialisation.

7. **Dossier `database/`**
   - Gère la connexion à la base de données via `db.py`.

8. **Dossier `utils/`**
   - Contient des fonctions utilitaires (ex. : gestion des erreurs).

9. **Dossier `tests/`**
   - Contient les tests unitaires et d'intégration pour chaque entité.

## Exemple de code

### 1. `app/models/user.py`
```python
from ..database.db import db

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)
    orders = db.relationship("Order", backref="user", lazy=True)

    def to_dict(self):
        return {"id": self.id, "name": self.name, "email": self.email}

2. app/models/product.pypython

from ..database.db import db

class Product(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    price = db.Column(db.Float, nullable=False)

    def to_dict(self):
        return {"id": self.id, "name": self.name, "price": self.price}

3. app/models/order.pypython

from ..database.db import db

class Order(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey("user.id"), nullable=False)
    product_id = db.Column(db.Integer, db.ForeignKey("product.id"), nullable=False)
    quantity = db.Column(db.Integer, nullable=False)

    def to_dict(self):
        return {
            "id": self.id,
            "user_id": self.user_id,
            "product_id": self.product_id,
            "quantity": self.quantity
        }

4. app/routes/user_routes.pypython

from flask import Blueprint, jsonify, request
from ..services.user_service import get_all_users, create_user

user_bp = Blueprint("user", __name__, url_prefix="/api/users")

@user_bp.route("/", methods=["GET"])
def get_users():
    users = get_all_users()
    return jsonify([user.to_dict() for user in users])

@user_bp.route("/", methods=["POST"])
def add_user():
    data = request.get_json()
    user = create_user(data["name"], data["email"])
    return jsonify(user.to_dict()), 201

5. app/__init__.pypython

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

6. run.pypython

from app import create_app

app = create_app()

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=True)

7. requirements.txt

flask
flask-sqlalchemy
python-dotenv
pytest
marshmallow

Bonnes pratiquesModularité : Séparez les modèles, routes, et services par entité.
Relations : Définissez les relations entre tables dans les modèles (ex. : one-to-many).
Sérialisation : Utilisez Marshmallow pour valider et formater les données JSON.
Tests : Écrivez des tests unitaires pour chaque entité (ex. : test_user_routes.py).
Documentation : Documentez les endpoints dans README.md ou avec Swagger.

Étapes pour lancer le projetCréez un environnement virtuel :bash

python -m venv venv
source venv/bin/activate  # Linux/Mac
venv\Scripts\activate     # Windows

Installez les dépendances :bash

pip install -r requirements.txt

Configurez les variables d'environnement dans .env :

DATABASE_URL=sqlite:///app.db
SECRET_KEY=your-secret-key

Lancez l'application :bash

python run.py

Testez les endpoints avec Postman ou curl :bash

curl -X POST http://localhost:5000/api/users -H "Content-Type: application/json" -d '{"name": "John Doe", "email": "john@example.com"}'

ConclusionCette structure est idéale pour gérer plusieurs tables, car elle sépare les responsabilités, facilite l'ajout de nouvelles entités, et permet une maintenance aisée. Pour des besoins spécifiques (ex. : authentification, pagination), ajoutez des modules supplémentaires.Generated on October 24, 2025