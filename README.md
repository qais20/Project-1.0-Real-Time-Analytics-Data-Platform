# Project: Real-Time Analytics Data Platform

## **Goal**
Build a scalable data platform that:
1. Ingests real-time user activity logs.
2. Processes the data.
3. Stores it for analytics.
4. Trains an ML model.
5. Exposes the results via APIs.

---

## **Step 1: Running Locally**

### **Tools**
- **Makefiles**: Simplify and automate local setup tasks.
- **Kind**: Create a local Kubernetes cluster.
- **Strimzi**: Deploy Kafka locally on Kubernetes.

### **Steps**
1. **Set Up Kind Cluster**:
   - Create a local Kubernetes cluster using Kind:
     ```bash
     kind create cluster --name data-platform
     ```
2. **Deploy Kafka with Strimzi**:
   - Use Strimzi Helm charts to deploy Kafka:
     ```bash
     helm repo add strimzi https://strimzi.io/charts/
     helm install kafka strimzi/strimzi-kafka-operator --namespace kafka --create-namespace
     ```
3. **Build and Deploy Locally**:
   - Use Makefiles for common tasks like building Docker images and deploying to Kind:
     ```bash
     make build
     make deploy
     ```
4. **Test Locally**:
   - Verify the local setup by running integration tests.

---

## **Step 2: Infrastructure Setup**

### **Tools**
- **AWS Services**: S3, RDS (PostgreSQL), SageMaker, EKS
- **Terraform**: Automate infrastructure provisioning
- **Kubernetes**: Deploy applications

### **Setup AWS Services**
Use Terraform to create:
- A **VPC** with public/private subnets, route tables, and an Internet Gateway (IGW).
- An **EKS cluster** for container orchestration.
- An **S3 bucket** for data storage.
- An **RDS PostgreSQL instance** for storing processed data.
- **SageMaker** for ML training and inference.

### **Deploy Kafka on Kubernetes**
1. Use Helm to deploy Kafka.
2. Create topics for real-time log ingestion (e.g., `user-activity-logs`).

### **GitHub Repository**
- Create a new repository: `github.com/<your-username>/data-platform-project`.
- Add a `terraform/` folder containing the complete AWS setup.

---

## **Step 3: Data Ingestion**

### **Tools**
- **Kafka**: For real-time data streaming
- **Python**: For producers and consumers

### **Producer**
1. Simulate user activity logs with a Python Kafka producer.
2. Example logs:
    ```json
    {
      "user_id": 123,
      "action": "view_product",
      "timestamp": "2024-11-25T12:00:00Z"
    }
    ```
3. Write the producer script in `src/ingestion/producer.py`.

### **Consumer**
1. Consume logs from the Kafka topic.
2. Perform minimal processing.
3. Write raw data to S3.
4. Write the consumer script in `src/ingestion/consumer.py`.

### **Testing**
- Add unit tests in `tests/ingestion_tests.py`.

---

## **Step 4: Data Processing**

### **Tools**
- **PySpark**: Batch and streaming data processing

### **Batch Processing**
1. Read raw data from S3.
2. Clean and transform data using PySpark.
3. Store processed data back in S3 and RDS.

### **Streaming Processing**
- Use PySpark Structured Streaming to process logs in real time from Kafka.

### **Testing**
- Write test cases to validate transformations.

---

## **Step 5: SQL Integration**

### **Use Cases for SQL**
- **Join tables**: For example, join user data with event logs.
- **Aggregate data**: Generate reports such as monthly user activity.
- **Transformations**: Perform operations not feasible in PySpark.

### **Example Queries**
1. **Basic Aggregation**:
   ```sql
   SELECT user_id, COUNT(*) AS total_purchases
   FROM user_activity
   WHERE action = 'purchase'
   GROUP BY user_id
   LIMIT 10;
   ```

2. **Relational Database Schema**:
   ```sql
   CREATE TABLE users (
       user_id SERIAL PRIMARY KEY,
       name VARCHAR(100),
       email VARCHAR(100),
       signup_date TIMESTAMP
   );

   CREATE TABLE user_activity (
       id SERIAL PRIMARY KEY,
       user_id INT REFERENCES users(user_id),
       action VARCHAR(50),
       event_timestamp TIMESTAMP,
       details JSONB
   );

   CREATE TABLE aggregated_metrics (
       metric_id SERIAL PRIMARY KEY,
       metric_name VARCHAR(50),
       metric_value NUMERIC,
       created_at TIMESTAMP
   );
   ```

3. **Active Users**:
   ```sql
   SELECT COUNT(DISTINCT user_id) AS active_users
   FROM user_activity
   WHERE event_timestamp > NOW() - INTERVAL '30 days';
   ```

### **API SQL Example**
Using SQL queries in the API backend:
```python
from fastapi import FastAPI
from sqlalchemy import create_engine

app = FastAPI()
engine = create_engine('postgresql://username:password@host:port/dbname')

@app.get("/api/active-users")
def get_active_users():
    query = """
    SELECT COUNT(DISTINCT user_id) AS active_users
    FROM user_activity
    WHERE event_timestamp > NOW() - INTERVAL '30 days'
    """
    result = engine.execute(query).fetchone()
    return {"active_users": result["active_users"]}
```

### **Step-by-Step SQL Integration**
1. **Set Up PostgreSQL**:
   - Deploy an RDS PostgreSQL instance using Terraform.
   - Store credentials securely in AWS Secrets Manager.
2. **Write Data**:
   - Use PySpark or Python libraries (e.g., `sqlalchemy`, `psycopg2`) to write processed data into PostgreSQL.
   ```python
   from sqlalchemy import create_engine

   engine = create_engine('postgresql://username:password@host:port/dbname')
   data.to_sql('user_activity', engine, if_exists='append', index=False)
   ```
3. **Fetch Data**:
   - Use SQLAlchemy or psycopg2 for API or further processing.
   ```python
   query = "SELECT user_id, COUNT(*) FROM user_activity WHERE action = 'view_product' GROUP BY user_id"
   result = engine.execute(query)
   for row in result:
       print(row)
   ```
4. **Analytics and Aggregation**:
   - Perform complex data manipulations with SQL.

---

## **Step 6: Machine Learning**

### **Tools**
- **SageMaker**: For model training and deployment
- **Python**: For feature engineering

### **Steps**
1. **Feature Engineering**:
   - Use PySpark to create features from processed data.
   - Store features in S3.
2. **Train Model**:
   - Train a SageMaker model to predict user behaviour (e.g., purchase likelihood).
   - Save the model artifacts in S3.
3. **Deploy Model**:
   - Deploy the model using a SageMaker endpoint.
4. **Testing**: Test the model with sample predictions.

---

## **Step 7: APIs**

### **Tools**
- **Flask/FastAPI**: For building REST APIs
- **PostgreSQL**: Backend database

### **Steps**
1. **Create APIs**:
   - Expose endpoints to fetch processed data insights and query ML model predictions.
   - Example endpoint:
     ```bash
     GET /api/predictions?user_id=123
     ```
2. **Dockerize API**:
   - Create a Dockerfile for the Flask/FastAPI app.
   - Deploy it on Kubernetes using Helm.
3. **Testing**:
   - Add integration tests in `tests/api_tests.py`.

---

## **Step 8: Orchestration and CI/CD**

### **Tools**
- **Kubernetes**: Orchestrate containers
- **GitHub Actions**: Automate builds, tests, and deployments

### **Steps**
1. **Deploy on Kubernetes**:
   - Deploy Kafka, PySpark, and APIs using Kubernetes manifests in the `k8s/` folder.
2. **Set Up CI/CD**:
   - Use GitHub Actions to:
     - Lint Python code.
     - Run unit tests.
     - Build and push Docker images.
     - Deploy to Kubernetes.

---

## **Documentation**

### **Content**
1. **README.md**:
   - Overview of the project.
   - High-level architecture diagram.
   - Instructions to clone and run locally.
2. **Docs Folder**:
   - Add setup and deployment instructions in `docs/how-to-run.md`.
   - API documentation in `docs/api-documentation.md`.

---

## **Deliverables**

1. **GitHub Repository**:
   - Include all code, tests, and documentation.
   - Use GitHub Pages to host the project documentation.
2. **Demo**:
   - Add a video demo of the platform processing data and exposing predictions via APIs.

---

## **Resources**

### **Kafka and MSK Labs**
- [Kafka MSK Labs](https://github.com/moabukar/kafka-msk)
- [Kafka Kubernetes Labs](https://github.com/moabukar/kafka-k8s)

