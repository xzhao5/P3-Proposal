# _**思路2.0-解析P3项目流程图**_

![sumary_process][process]

## **1.组成部分**

### 按照功能为基础，我在图上划出了5个主要区域：**Ingetition** ,  **Prcossing** , **Storage**, **Machine Learning** , **Consumption**, **Security & Monitoring**

### **Ingetition**：这一步部分主要是注入大量的原始数据，主要考虑有两个方向，batch或者stream，batch注入我想使用snowfalke，stream注入主要是用amazon kinests，产出的数据是5个raw .csv文件： aisle.csv, department.csv, order.csv, product.csv, order_prodect_prior.csv, order_prodect_train.csv - 这些是Bronze 数据。

### **Prcossing**：这是一个对原始数据的预处理的过程，应该叫做Pre-data processing（ELT），相当于是一个warehou的过程，我设计两个数据预处理的过程：第一个过程是Validate,Clean, Standardize, Norm，也就把多余的，错误的信息进行处理；第二个过程是Transform，把第一个过程处理的干净数据，进行变形，变形成我们想要的数据，或者说，有商业价值的数据。这两个过程都是需要lambda function作为一个trigger来启动。产出的数据为4张整洁的表： user_feature_1, user_feature_2, up_features, prd_features - 这些是Silver数据。

### **Storage**：这个就是data lake的结果设计，我设计了四个bucket，分别是Raw - 存储源数据， Clean Data - Processing第一过程之后的干净数据， Curated/Consumption - Processing 第二过程之后的变形的数据， Prepared Machine Learning Data - 机器学习预处理之前的数据。

### **Machine Learning**： 这一部分主要是展示机器学习的功能，主要使用Amazon SegaMarker， 当四个表数据进入到这个功能区域后，通过SQS的access key的链接，通过Lambda function触发Glue去预处理这个数据，来满足可以使用machine learning model的数据条件，然后通过Training来形成model，最后配置model，这一功能的产出是一个Predict_Prodct的数据 - 负责预测用户下一个会买的产品，也就是Gold数据。

### **Consumption**：这一步是这个项目的最后流向，也就是产出，它需要把machine learning产出的gold 数据，通过Lambda function， 接入一个API 可以连接客户端，达到实时分析的效果。

### **Security & Monitoring**: 这个部分主要是安全和监控，安全部分主要靠IAM Role和Key进行实现，Cloud Watch进行监控。

### **其他**： 我们还可以加入github进行vesrion control， python 插件 BOTO3，pandas等等进行辅助。

## **2.数据的流向**

### **起源**： 来自客户提供额第三方数据 或者stream 数据。
### **Step1**： 第三方数据流向snowfalke进行batch处理。 stream data 流向Amzon Kinesis。产出6个csv文件。
### **Step2**： 6个csv文件存入‘RAW’ S3 Bucket。
### **Step3**： AWS Lambda fucntion triggr ELT job 1， 同时把csv文件进行第一次处理 - Validate,Clean, Standardize, Norm。处理后的数据存入‘Clean Data’ S3 Bucket.
### **Step4**： AWS Lambda fucntion triggr ELT job 2， 从‘Clean Data’提取第一次处理后的数据，进行第二次处理 - Transform，处理后的数据存入‘Curated/Consumption’ S3 Bucket.
### **Step5**： 从‘Curated/Consumption’提出数据，通过SQS，lambda fucntion 'SMPiplienQueueHandler' trigger之后，再由Glue产出预处理的machine learning 数据， 这个出处理后的数据要存到 ‘Prepared Machine Learning Data’ S3 Bucket。
### **Step6**： 从 ‘Prepared Machine Learning Data’ S3 Bucket提取数据，基于Amazon SegaMaker部件，通过机器学习的建模，训练和配置过程后，产出数据‘Predict_Product’。
### **Step7**： 最后预测的数据‘Predict_Prodcut’， 通过lambda function ‘API GATEWAY CONNECTION’连接到客户端进行展示。
