<html><p><a href="https://loko-ai.com/" target="_blank" rel="noopener"> <img style="vertical-align: middle;" src="https://user-images.githubusercontent.com/30443495/196493267-c328669c-10af-4670-bbfa-e3029e7fb874.png" width="8%" align="left" /> </a></p>
<h1>Text Classification from Mongo</h1><br></html>

A simple example of training/predicting a text classification model using the **Predictor** and **MongoDB** component.

From the **Projects** tab, click on **Import from git** and copy and paste the URL of the current page 
(i.e. https://github.com/loko-ai/text_classification_from_mongo):
<p align="center"><img src="https://user-images.githubusercontent.com/30443495/230620716-c67e5e58-b71e-4817-9213-39a7e0e9cddd.gif" width="80%" /></p>


### STEP1: Mongo ingestion

From the **Applications** tab install and run the **mongo_extension** (See more here https://github.com/loko-ai/mongo_extension):

<p align="center"><img src="https://user-images.githubusercontent.com/30443495/236760265-d25d26ca-8484-4b3a-ba5a-df30300cede3.gif" width="80%" /></p>

You can then go back to your project and run it. 

In order to start the project remember to press the **play** button on the right of the project's name:

<p align="center"><img src="https://user-images.githubusercontent.com/30443495/231104375-fa1040ff-5b98-4ea3-9068-c523791d9e13.gif" width="60%" /></p>

The first flow use the **CSV Reader** component to read the *email_dataset.csv* and save records to a MongoDB collection named *email*.

<p align="center"><img src="https://user-images.githubusercontent.com/30443495/230619982-29889ac5-b7a8-439e-afc9-851623426179.png" width="80%" /></p>

In order to check if you correctly saved your data, you can run the second **Trigger** component, named *List*.

<p align="center"><img src="https://user-images.githubusercontent.com/30443495/230623001-031d5a3d-a545-437e-bd63-89acd8170adc.png" width="80%" /></p>

The expected output shows that you have a collection named email containing 1000 records. Otherwise, you can delete data
using the last flow and ingest again your dataset.

You can also visualize the collection content using the **db manager** GUI from the Applications tab:

<p align="center"><img src="https://user-images.githubusercontent.com/30443495/236760289-40a50da4-5960-4574-8655-1711bb5f3c82.gif" width="80%" /></p>

### STEP2: Model training

Now you have to create a new predictor from the **Predictors** tab using an auto model and auto transformer: 

<p align="center"><img src="https://user-images.githubusercontent.com/30443495/236760307-a1e57481-77c2-4634-a94a-8e54f824b841.gif" width="80%" /></p>

You can go back to the project and fit the email_clf model:

<p align="center"><img src="https://user-images.githubusercontent.com/30443495/230628094-643e9bee-c454-4eb6-b70a-effc6b0a2a7d.png" width="80%" /></p>

The email_clf is an AutoML classifier (i.e. it automatically chose the best model and hyperparameters) which uses the text
of the emails given in input to predict the associated labels.

The **MongoDB** component reads data from the *email* collection, **Selector** component selects only *label* and *text* fields 
and finally **Predictor** component fits the *email_clf* predictor using field *label* as the target. 

Ones the model is fitted, you can open the Predictors tab and visualize the generated report:

<p align="center"><img src="https://user-images.githubusercontent.com/30443495/236760331-045534a0-56d0-46dc-aa3a-38d4c3f6a76a.gif" width="80%" /></p>

### STEP3: Expose service

Finally, you can expose a service to obtain the email predictions. 

<p align="center"><img src="https://user-images.githubusercontent.com/30443495/230631548-aa1ef38c-d75c-4e3a-8bfd-b44a4011cd08.png" width="80%" /></p>

Use **Route** and **Response** components at the beginning and at the end of the flow producing the output. In the 
**Route** configuration name the service *predict* and copy the complete url (i.e. http://localhost:9999/routes/orchestrator/endpoints/text_classification_from_mongo/predict).
In the **Response** component set the *Response Type* to json.

The **Function** component extracts the body from the received request and returns it to the **Predictor** component 
which predicts the output.

You can test it using:

```commandline
curl -d "\"prova di un testo email\"" -X POST http://localhost:9999/routes/orchestrator/endpoints/text_classification_from_mongo/predict
```

### STEP4: Test service

You can test the *predict* service directly in your flow using the **HTTP Request** component:

<p align="center"><img src="https://user-images.githubusercontent.com/30443495/230633240-800dffa3-b1da-4cad-b926-87fa90800222.png" width="80%" /></p>

In this case you have change http://localhost:9999 to http://gateway:8080 since the request will be executed in one of the containers inside the *loko* network (i.e. http://gateway:8080/routes/orchestrator/endpoints/text_classification_from_mongo/predict). 