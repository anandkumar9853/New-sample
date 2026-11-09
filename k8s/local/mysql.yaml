import json
import os

import boto3
import pymysql
from flask import Flask, jsonify, render_template

app = Flask(__name__)


def get_db_config():
    db_mode = os.environ.get("DB_MODE", "aws").lower()

    if db_mode == "local":
        return {
            "host": os.environ["DB_HOST"],
            "user": os.environ["DB_USER"],
            "password": os.environ["DB_PASSWORD"],
            "database": os.environ["DB_NAME"],
        }

    if db_mode == "aws":
        secret_name = os.environ["SECRET_NAME"]
        region_name = os.environ.get("AWS_REGION", "us-east-2")

        client = boto3.client("secretsmanager", region_name=region_name)
        response = client.get_secret_value(SecretId=secret_name)

        return json.loads(response["SecretString"])

    raise ValueError(f"Unsupported DB_MODE: {db_mode}")


def get_inventory():
    config = get_db_config()

    connection = pymysql.connect(
        host=config["host"],
        user=config.get("username", config.get("user")),
        password=config["password"],
        database=config.get("database", config.get("dbname")),
        cursorclass=pymysql.cursors.DictCursor,
        connect_timeout=5,
    )

    with connection:
        with connection.cursor() as cursor:
            cursor.execute(
                "SELECT id, item, category, quantity FROM inventory;"
            )
            return cursor.fetchall()


@app.route("/healthz")
def health():
    return jsonify({"status": "healthy"}), 200


@app.route("/")
def home():
    inventory = get_inventory()
    return render_template("index.html", inventory=inventory)


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)