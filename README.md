# Final-Project_-6
## Flask Healthcare Application
│
├── app/
│   ├── templates/
│   │   └── form.html
│   ├── static/
│   ├── app.py
│   └── config.py
│
├── data/
│   └── users.csv
│
├── notebooks/
│   └── analysis.ipynb
│
├── requirements.txt
├── README.md
└── .env
## Form.Rhtml
<form method="POST">
  Age: <input type="number" name="age"><br>
  Gender: <select name="gender">
    <option value="Male">Male</option>
    <option value="Female">Female</option>
  </select><br>
  Income: <input type="number" name="income"><br>
  <h4>Expenses:</h4>
  {% for category in ['utilities', 'entertainment', 'school_fees', 'shopping', 'healthcare'] %}
    <input type="checkbox" name="{{ category }}" value="1">
    {{ category.capitalize() }}: <input type="number" name="{{ category }}"><br>
  {% endfor %}
  <input type="submit" value="Submit">
</form>
## Flask Web App (Data Collection).py
from flask import Flask, render_template, request, redirect
from pymongo import MongoClient
import os

app = Flask(__name__)
client = MongoClient(os.getenv("MONGO_URI"))
db = client["survey_db"]
collection = db["users"]

@app.route("/", methods=["GET", "POST"])
def survey():
    if request.method == "POST":
        data = {
            "age": request.form["age"],
            "gender": request.form["gender"],
            "income": float(request.form["income"]),
            "expenses": {
                category: float(request.form.get(category, 0))
                for category in ["utilities", "entertainment", "school_fees", "shopping", "healthcare"]
            }
        }
        collection.insert_one(data)
        return redirect("/")
    return render_template("form.html")

if __name__ == "__main__":
    app.run(debug=True)
## Import Numpy.py
import csv
from pymongo import MongoClient

class User:
    def __init__(self, mongo_uri):
        self.client = MongoClient(mongo_uri)
        self.collection = self.client["survey_db"]["users"]

    def export_to_csv(self, filepath):
        users = self.collection.find()
        with open(filepath, "w", newline="") as f:
            writer = csv.writer(f)
            writer.writerow(["Age", "Gender", "Income", "Utilities", "Entertainment", "School Fees", "Shopping", "Healthcare"])
            for user in users:
                expenses = user.get("expenses", {})
                writer.writerow([
                    user.get("age"),
                    user.get("gender"),
                    user.get("income"),
                    expenses.get("utilities", 0),
                    expenses.get("entertainment", 0),
                    expenses.get("school_fees", 0),
                    expenses.get("shopping", 0),
                    expenses.get("healthcare", 0)
                ])
## Jupiternotebook.py
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

df = pd.read_csv("../data/users.csv")

# Ages with highest income
top_income = df.groupby("Age")["Income"].mean().sort_values(ascending=False).head(10)
top_income.plot(kind="bar", title="Top Ages by Average Income")
plt.savefig("../data/top_income.png")

# Gender distribution across spending
expense_cols = ["Utilities", "Entertainment", "School Fees", "Shopping", "Healthcare"]
df_melted = df.melt(id_vars=["Gender"], value_vars=expense_cols, var_name="Category", value_name="Amount")
sns.boxplot(data=df_melted, x="Category", y="Amount", hue="Gender")
plt.title("Spending by Gender Across Categories")
plt.savefig("../data/gender_spending.png")
