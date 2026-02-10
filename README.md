# How-to-Deploy-a-Python-Machine-Learning-Model-on-OCI-and-Integrate-with-Oracle-APEX
How to Deploy a Python Machine Learning Model on OCI and Integrate with Oracle APEX
Introduction
Oracle APEX is a powerful low-code platform, but when combined with Python Machine Learning models deployed on Oracle Cloud Infrastructure (OCI), it becomes a truly intelligent application platform.
In this blog, I will explain how I deployed a Python ML model on OCI and integrated it with Oracle APEX using REST APIs to enable real-time predictions inside an APEX application.

Oracle APEX is a powerful low-code platform, but when combined with Python-based Machine Learning models deployed on Oracle Cloud Infrastructure (OCI), it enables truly intelligent, data-driven enterprise applications.
In this blog, I share a hands-on, real-world implementation where a Python ML model is deployed on OCI and consumed inside an Oracle APEX application using REST APIs to provide real-time responses and predictions.
This approach demonstrates how APEX can seamlessly integrate with modern AI/ML services while maintaining security and scalability.

 
 
 

Issues Found and Fixed:
Process Configuration Issue: The process "New_1" was configured 
to use "PL/SQL Procedure or Function" type that was calling 
CALL_TEST_REST procedure, which was causing the HTTP request failure.

Button Binding Issue: The process was not bound to the P1_SUBMIT button initially (it was set to "- Select -").

Changes Made:
Changed Process Type: Modified the process from "PL/SQL Procedure or Function" to "REST Source"


Set REST Source to: OPENAI_CHAT_API

Set Operation to: POST_CHAT
Bound Process to Button: Configured the "When Button Pressed" server-side condition to P1_SUBMIT

Saved Changes: Successfully saved all changes to the page

Why This Fixes the Error:

The ORA-29273 error typically occurs when PL/SQL procedures attempt to make HTTP calls without proper configuration or network access. By changing the process to use Oracle APEX's built-in REST Source mechanism, the application now properly handles HTTP requests through the OPENAI_CHAT_API REST Data Source, which includes proper authentication, SSL certificates, and network access configuration.

 
begin
  call_test_rest(
      p_input    => :P1_INPUT,
      p_response => :P1_RESPONSE
  );
end;

now work correctly when you click the P1 Submit button - it will call the OpenAI Chat API through the properly configured REST Source instead of the failing PL/SQL procedure.

Architecture Overview
The solution consists of:
•	Python ML model hosted on OCI Compute
•	REST API built using Flask
•	Oracle APEX application consuming the API
•	Secure communication between APEX and OCI
This architecture enables scalable, real-time AI features inside APEX apps.

Step 1: Build the Python ML Model
I created a simple machine learning model using Python and scikit-learn.
The model was trained locally and saved using joblib.
Example:
•	Input: Business parameters (numeric values)
•	Output: Prediction score

Step 2: Deploy Model on OCI
Steps followed:
1.	Created OCI Compute instance
2.	Installed Python, Flask, and required libraries
3.	Uploaded trained ML model
4.	Exposed prediction logic using Flask REST API
The API returns predictions in JSON format.

Step 3: Secure the REST API
To ensure security:
•	API runs on a private port
•	OCI Security List updated
•	Authentication handled via headers
This keeps the ML service safe while accessible to APEX.

Step 4: Integrate with Oracle APEX
In Oracle APEX:
•	Created REST Data Source
•	Configured POST request to OCI endpoint
•	Passed input values from APEX page items
•	Parsed JSON response
The prediction result is displayed instantly on the APEX UI.

Step 5: Business Use Case
This solution can be used for:
•	Financial risk prediction
•	Demand forecasting
•	Employee performance insights
•	Intelligent ERP dashboards
APEX + OCI ML opens the door for AI-driven enterprise applications.

Challenges Faced & Solutions
Challenge: Network access between APEX and OCI
Solution: Proper OCI security rules and endpoint testing
Challenge: JSON parsing in APEX
Solution: Used REST Data Source auto-mapping

Conclusion
By integrating Python Machine Learning models deployed on OCI with Oracle APEX, developers can build intelligent, scalable, and future-ready applications.
Oracle APEX is not just low-code—it’s a powerful platform for AI-enabled enterprise solutions.
 

 
 

 

create or replace procedure call_test_rest (
    p_input    in  varchar2,
    p_response out clob
) as
    l_req   utl_http.req;
    l_resp  utl_http.resp;
    l_line  varchar2(32767);
    l_body  clob;
    l_url   varchar2(4000);
begin
    l_url := 'https://postman-echo.com/get?input=' ||
             utl_url.escape(p_input);

    l_req  := utl_http.begin_request(l_url, 'GET');
    l_resp := utl_http.get_response(l_req);

    loop
        utl_http.read_line(l_resp, l_line, true);
        l_body := l_body || l_line || chr(10);
    end loop;
exception
    when utl_http.end_of_body then
        utl_http.end_response(l_resp);
end;
/

 

Architecture Overview
The solution architecture consists of the following components:
•	Python Machine Learning model hosted on OCI Compute
•	REST API developed using Flask
•	Oracle APEX application consuming the REST API
•	Secure communication using HTTPS and OCI security rules
This architecture allows Oracle APEX applications to leverage ML capabilities without embedding complex ML logic inside the database.

Step 1: Build the Python Machine Learning Model
A simple machine learning model was created using Python.
The model was trained locally and serialized for deployment.
Input: Business-related numeric parameters
Output: Prediction or processed response
The trained model was saved using joblib so it can be reused by the REST API.

Step 2: Deploy the Model on OCI
The following steps were performed on OCI:
1.	Created an OCI Compute instance
2.	Installed Python and required libraries
3.	Uploaded the trained ML model
4.	Built a REST API using Flask
5.	Exposed the API endpoint over HTTPS
The REST API returns responses in JSON format, making it easy for Oracle APEX to consume.

Step 3: Python Flask REST API (OCI)
from flask import Flask, request, jsonify
import joblib

app = Flask(__name__)

model = joblib.load("model.pkl")

@app.route("/predict", methods=["POST"])
def predict():
    data = request.json
    input_value = data.get("input")

    prediction = model.predict([input_value])[0]

    return jsonify({
        "input": input_value,
        "prediction": prediction
    })

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
This REST API accepts JSON input and returns a prediction response that Oracle APEX can parse easily.

Step 4: Secure the REST API
To secure the deployment:
•	API runs on a restricted port
•	OCI Security Lists allow only required inbound traffic
•	HTTPS is used for communication
•	Headers are validated for authorized access
This ensures secure interaction between Oracle APEX and OCI services.

Step 5: Integrate REST API with Oracle APEX
Inside Oracle APEX, the integration was done using REST Data Sources.
Configuration Steps:
•	Created a REST Data Source pointing to the OCI endpoint
•	Configured a POST operation
•	Passed page item values as request payload
•	Parsed JSON response automatically using APEX mapping
The prediction result is displayed instantly on the APEX page.

Important Fix: ORA-29273 Error (Real Issue & Solution)
Initially, the application was failing with ORA-29273: HTTP request failed.
Root Cause:
•	Page process was configured as PL/SQL Procedure or Function
•	Procedure was using UTL_HTTP directly
•	Network ACL and SSL issues caused failures
Fix Applied:
•	Changed the process type from PL/SQL Procedure to REST Source
•	Bound the process to the correct button (P1_SUBMIT)
•	Used APEX REST Data Source instead of direct HTTP calls
Why This Works:
Oracle APEX REST Data Sources handle:
•	Network access
•	SSL certificates
•	Authentication
•	Error handling
This eliminates common UTL_HTTP failures and makes integration more reliable.

Reference PL/SQL (Before Fix – Not Recommended)
begin
  call_test_rest(
      p_input    => :P1_INPUT,
      p_response => :P1_RESPONSE
  );
end;
After switching to REST Source, this PL/SQL block is no longer required.

Business Use Cases
This architecture can be used for:
•	Financial risk prediction
•	Demand forecasting
•	Intelligent ERP dashboards
•	AI-powered decision support systems

Challenges Faced & Solutions
Challenge: Network connectivity and SSL issues
Solution: Used APEX REST Data Source instead of UTL_HTTP
Challenge: JSON parsing complexity
Solution: APEX REST auto-mapping feature

Conclusion
By integrating Python Machine Learning models deployed on OCI with Oracle APEX, developers can build intelligent, scalable, and future-ready enterprise applications.
Oracle APEX is not just a low-code tool—it is a powerful platform capable of supporting AI-driven solutions when combined with OCI services.

ACE DASHBOARD SUBMISSION DETAILS
Contribution Type: Blog Post
Tech Content: YES
Title:
How to Deploy a Python Machine Learning Model on OCI and Integrate with Oracle APEX
Description (copy-paste):
This blog explains a real-world implementation of deploying a Python machine learning model on Oracle Cloud Infrastructure and integrating it with Oracle APEX using REST APIs to enable real-time predictions

Reference:-
https://www.blogger.com/blog/post/edit/7852511296724383645/1082122175102608994




