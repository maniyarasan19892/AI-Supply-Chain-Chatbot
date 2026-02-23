import pyodbc
import ollama

# Connect to SQL Server
conn = pyodbc.connect(
    "DRIVER={ODBC Driver 17 for SQL Server};"
    "SERVER=localhost\\SQLEXPRESS;"
    "DATABASE=supply_chain;"
    "Trusted_Connection=yes;"
)

cursor = conn.cursor()

print("📦 AI Supply Chain Chatbot (Local LLM)")
print("Type 'exit' to quit\n")

while True:
    user_question = input("You: ")

    if user_question.lower() == "exit":
        print("Chatbot: Goodbye 👋")
        break

    # Prompt for LLM
    prompt = f"""
You are an SQL Server expert.

Database:
FactSupplyChain(ProductKey, SupplierKey, LogisticsKey, Revenue, Cost, DefectRate, Quantity, ShippingCost)

Generate ONLY a valid SQL Server query.
Do not explain anything.
Question: {user_question}
"""

    response = ollama.chat(
        model="llama3",
        messages=[{"role": "user", "content": prompt}]
    )

    sql_query = response["message"]["content"]

    print("\nGenerated SQL:\n", sql_query)

    try:
        cursor.execute(sql_query)
        rows = cursor.fetchall()

        print("\nResults:")
        for row in rows:
            print(row)

    except Exception as e:
        print("SQL Execution Error:", e)

    print("\n" + "-"*50 + "\n")

conn.close()