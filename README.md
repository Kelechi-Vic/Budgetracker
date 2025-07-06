# Budgetracker
# --- File: app/__init__.py ---
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from app.config import Config

app = Flask(__name__)
app.config.from_object(Config)
db = SQLAlchemy(app)

from app import routes, models

# --- File: app/config.py ---
import os

class Config:
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'dev-key'
    SQLALCHEMY_DATABASE_URI = 'sqlite:///budget.db'
    SQLALCHEMY_TRACK_MODIFICATIONS = False

# --- File: app/models.py ---
from app import db

class Transaction(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    description = db.Column(db.String(100), nullable=False)
    amount = db.Column(db.Float, nullable=False)
    category = db.Column(db.String(50), nullable=False)
    type = db.Column(db.String(10), nullable=False)  # 'income' or 'expense'
    date = db.Column(db.String(20), nullable=False)

    def to_dict(self):
        return {
            'id': self.id,
            'description': self.description,
            'amount': self.amount,
            'category': self.category,
            'type': self.type,
            'date': self.date
        }

# --- File: app/routes.py ---
from flask import request, jsonify
from app import app, db
from app.models import Transaction

@app.route('/transactions', methods=['GET'])
def get_transactions():
    transactions = Transaction.query.all()
    return jsonify([t.to_dict() for t in transactions])

@app.route('/transactions', methods=['POST'])
def add_transaction():
    data = request.get_json()
    transaction = Transaction(
        description=data['description'],
        amount=data['amount'],
        category=data['category'],
        type=data['type'],
        date=data['date']
    )
    db.session.add(transaction)
    db.session.commit()
    return jsonify(transaction.to_dict()), 201

@app.route('/transactions/<int:id>', methods=['DELETE'])
def delete_transaction(id):
    transaction = Transaction.query.get_or_404(id)
    db.session.delete(transaction)
    db.session.commit()
    return jsonify({'message': 'Transaction deleted'})

# --- File: run.py ---
from app import app, db

if __name__ == '__main__':
    with app.app_context():
        db.create_all()
    app.run(debug=True)

# --- File: requirements.txt ---
Flask==2.3.2
Flask-SQLAlchemy==3.1.1

# --- File: README.md ---
# Budget Tracker API

A simple RESTful API built with Flask for tracking income and expenses.

## Features
- Add, retrieve, and delete transactions
- SQLite for lightweight storage
- Categorization of income/expense by type and date

## Setup Instructions

```bash
# Clone the repo
git clone https://github.com/yourusername/budget-tracker-api.git
cd budget-tracker-api

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Run the app
python run.py
```

## API Endpoints
- `GET /transactions` - List all transactions
- `POST /transactions` - Add a new transaction
- `DELETE /transactions/<id>` - Delete a transaction by ID
