from PIL import Image
import streamlit as st
import pandas as pd
import plotly.graph_objects as go
import mysql.connector
from datetime import datetime
import streamlit as st
from datetime import datetime
import streamlit as st
import threading
import time
import random

# Password protection function
def authenticate():
    password = st.text_input("Enter password:", type="password")
    return password

# Check password function
def check_password(password):
    # Replace 'your_password' with the actual password
    return password == "Ritika"

# Authenticate
password = authenticate()
if check_password(password):
    st.success("Authentication successful!")
else:
    st.error("Authentication failed. Please try again.")
    st.stop()  # Stop execution if authentication fails


# Function to execute MySQL queries
def execute_query(query):
    try:
        connection = mysql.connector.connect(host='your_host',
                                             database='your_database',
                                             user='your_username',
                                             password='your_password')
        if connection.is_connected():
            cursor = connection.cursor()
            cursor.execute(query)
            records = cursor.fetchall()
            st.write("Query executed successfully!")
            st.write("Local Address:", datetime.now().strftime("%Y-%m-%d %H:%M:%S"))
            if records:
                st.write("Query Result:")
                df = pd.DataFrame(records, columns=[i[0] for i in cursor.description])
                st.write(df)

                # Plotting graph
                fig = go.Figure()
                for col in df.columns:
                    if df[col].dtype == 'int64' or df[col].dtype == 'float64':
                        fig.add_trace(go.Scatter(x=df.index, y=df[col], mode='lines+markers', name=col))
                st.plotly_chart(fig, use_container_width=True)
            else:
                st.write("No records found.")
    except Exception as e:
        st.error(f"Error: {e}")
    finally:
        if connection.is_connected():
            cursor.close()
            connection.close()

# Function to insert data into MySQL database
def bulk_insert_to_sql(table_name, df):
    try:
        connection = mysql.connector.connect(host='your_host',
                                             database='your_database',
                                             user='your_username',
                                             password='your_password')
        if connection.is_connected():
            cursor = connection.cursor()
            columns = ", ".join(df.columns)
            for i, row in df.iterrows():
                sql = f"INSERT INTO {table_name} ({columns}) VALUES ({', '.join(['%s'] * len(row))})"
                cursor.execute(sql, tuple(row))
            connection.commit()
            st.success("Data inserted successfully!")
    except Exception as e:
        st.error(f"Error: {e}")
    finally:
        if connection.is_connected():
            cursor.close()
            connection.close()

# Function to retrieve and display all data from a SQL table
def display_all_data(table_name):
    query = f"SELECT * FROM {table_name}"
    execute_query(query)

# Function to find the maximum number
def find_max(numbers):
    max_num = max(numbers)
    st.write("Finding Maximum Number...")
    st.write(f"Input values: {numbers}")
    st.write(f"Maximum Number: {max_num}")
    fig = go.Figure(data=[go.Bar(x=["Max"], y=[max_num])])
    st.plotly_chart(fig, use_container_width=True)
    return fig

# Function to find the minimum number
def find_min(numbers):
    min_num = min(numbers)
    st.write("Finding Minimum Number...")
    st.write(f"Input values: {numbers}")
    st.write(f"Minimum Number: {min_num}")
    fig = go.Figure(data=[go.Bar(x=["Min"], y=[min_num])])
    st.plotly_chart(fig, use_container_width=True)
    return fig

# Function to perform addition
def addition(a, b):
    result = a + b
    st.write("Performing Addition...")
    st.write(f"Input values: a={a}, b={b}")
    st.write(f"Result: {result}")
    fig = go.Figure(data=[go.Bar(x=["Addition"], y=[result])])
    st.plotly_chart(fig, use_container_width=True)
    return fig

# Function to perform subtraction
def subtraction(a, b):
    result = a - b
    st.write("Performing Subtraction...")
    st.write(f"Input values: a={a}, b={b}")
    st.write(f"Result: {result}")
    fig = go.Figure(data=[go.Bar(x=["Subtraction"], y=[result])])
    st.plotly_chart(fig, use_container_width=True)
    return fig

# Function to perform multiplication
def multiplication(a, b):
    result = a * b
    st.write("Performing Multiplication...")
    st.write(f"Input values: a={a}, b={b}")
    st.write(f"Result: {result}")
    fig = go.Figure(data=[go.Bar(x=["Multiplication"], y=[result])])
    st.plotly_chart(fig, use_container_width=True)
    return fig

# Function to calculate percentage
def percentage(a, b):
    result = (a / b) * 100
    st.write("Calculating Percentage...")
    st.write(f"{a} is {result:.2f}% of {b}")
    fig = go.Figure(data=[go.Bar(x=["Percentage"], y=[result])])
    st.plotly_chart(fig, use_container_width=True)
    return fig
## Title with image
image = Image.open("logo.jpg.png")
st.image(image, use_column_width=True)
st.title("Welcome")

# Date selection form
with st.sidebar:
    st.write("## Date Selection")
    start_date = st.date_input("Start Date")
    end_date = st.date_input("End Date")
    submit_button = st.button('Submit')

# Sidebar for selecting algorithm
st.sidebar.title("Algorithm")
algorithm = st.sidebar.selectbox("Select an Operation", ("Max Number", "Min Number", "Addition", "Subtraction", "Multiplication", "Percentage"))

# Input fields based on selected algorithm
if algorithm in ["Addition", "Subtraction", "Multiplication"]:
    a = st.sidebar.number_input("Enter first number")
    b = st.sidebar.number_input("Enter second number")

elif algorithm == "Percentage":
    a = st.sidebar.number_input("Enter the value")
    b = st.sidebar.number_input("Enter the total value")

elif algorithm in ["Max Number", "Min Number"]:
    numbers = st.sidebar.text_input("Enter numbers separated by comma")

# Run algorithm
show_result = st.sidebar.button("Show Result")
result_fig = None
if show_result:
    if algorithm == "Max Number":
        numbers_list = [int(num.strip()) for num in numbers.split(",")]
        result_fig = find_max(numbers_list)
    
    elif algorithm == "Min Number":
        numbers_list = [int(num.strip()) for num in numbers.split(",")]
        result_fig = find_min(numbers_list)
    
    elif algorithm == "Addition":
        result_fig = addition(a, b)
    
    elif algorithm == "Subtraction":
        result_fig = subtraction(a, b)
    
    elif algorithm == "Multiplication":
        result_fig = multiplication(a, b)
    
    elif algorithm == "Percentage":
        result_fig = percentage(a, b)

if result_fig:
    st.write("### Result Visualization")
    st.plotly_chart(result_fig, use_container_width=True)

# MySQL Query Section
st.sidebar.header("MySQL Query")
query = st.sidebar.text_area("Enter your SQL query here")

if st.sidebar.button("Execute Query"):
    if query:
        execute_query(query)

# Data Visualization and Bulk Insert Section
uploaded_file = st.file_uploader("Upload a CSV file")

if uploaded_file is not None:
    data = pd.read_csv(uploaded_file)
    
    st.write("### Uploaded CSV Data")
    st.write(data)
    
    if st.button("Bulk Insert CSV Data to MySQL"):
        table_name = st.text_input("Enter table name")
        if table_name:
            bulk_insert_to_sql(table_name, data)
        else:
            st.error("Please provide a table name.")
    
    # Button to display stored data from MySQL
    if st.button("Show Stored Data from MySQL"):
        table_name = st.text_input("Enter table name for displaying data")
        if table_name:
            display_all_data(table_name)
        else:
            st.error("Please provide a table name.")
    
    # Select visual chart type
    chart_visual = st.selectbox('Select Charts/Plot type', ('Line Chart', 'Bar Chart', 'Bubble Chart'))

    fig = go.Figure()

    if chart_visual == 'Line Chart':
        fig.add_trace(go.Scatter(x=data.index, y=data['Division'], mode='lines', name='Division'))

    elif chart_visual == 'Bar Chart':
        fig.add_trace(go.Bar(x=data.index, y=data['Division'], name='Division'))

    elif chart_visual == 'Bubble Chart':
        fig.add_trace(go.Scatter(x=data.index, y=data['Division'], mode='markers', marker_size=[40, 60, 80, 60, 40, 50], name='Division'))

    st.plotly_chart(fig, use_container_width=True)
else:
    st.sidebar.warning("Please upload a CSV file.")
