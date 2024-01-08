### AWS Capstone Project 
## Weather App by Clout Computing 

Project Developers: Lauren Johnson, Arbaaz Asim, Chris Hornick, and Nick Davis.

---
Project Overview: Users will engage with the Flask application by downloading it and inputting their city data. The python code will dynamically parse the data to present real time weather updates directly on their web browser. 

Table of Contents
- [AWS CDK](#aws-cdk)
- [Docker](#docker)
- [Python](#python)
- [AWS CloudWatch](#aws-cloudwatch)
  
---
Tools: AWS Application Load Balancer, Docker Containers, and Flask Applications. 

---

### AWS CDK 
Building the architecture via code. 

```cdk
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as ec2 from "aws-cdk-lib/aws-ec2";
import * as ecs from "aws-cdk-lib/aws-ecs"
import * as ecs_patterns from "aws-cdk-lib/aws-ecs-patterns"
import { ApplicationProtocol } from 'aws-cdk-lib/aws-elasticloadbalancingv2';

export interface FrontendProps extends cdk.StackProps {
   autoscaling: {
        minCapacity: number;
        maxCapacity: number;
        instanceType: ec2.InstanceType,
   },
}
export class CdkWorkshopStack extends cdk.Stack {

   constructor(scope: Construct, id: String, env: string,  frontendProps: FrontendProps) {
    super(scope, id, frontendProps);

   const vpc = new ec2.Vpc(this, 'clout-vpc', {
}) //creates a VPC

 const cluster = new ecs.Cluster(this, "Clout Cluster", {
      enableFargateCapacityProviders; true,
      vpc: vpc
});
new ecs_patterns.ApplicationLoadBalancedFargateService(this, 'cloud-service', {
cluster: cluster,
memoryLimitMib: 512,
desiredCount: frontendProps.autoscaling.maxCapacity,
cpu: 256,
protocol: ApplicationProtocol.HTTP
```
### Docker

```docker
FROM python:3.8-silm-buster
RUN pip install requests
RUN pip install flask
COPY app.py .
COPY templates/data.html templates/form.html templates/
EXPOSE 80
CMD ["python3", "app.py"]
```

---

### Python
The @app.route('/') is where users will enter their location. That will then load to the data page which will make the API call, parse the data and display it for the user. 

```python
app = Flask(_name_)
=app.config["DEBUG"] = True

@app.route('/')
def form():
    return render_template('form.html')

@app.route('/data/', methods=['POST'])
def data():

 # create a variable containing the value of the city field on the html form
 form_data = request.form['City']

# create the url containing the city variable, proces the response
url = "https://api.openweathermap.org/data/2.5/weather?q=" + \
       form_data + "&appid=" + api_key = '&units=imperial'
response = requests.get(url)
data = response.json()

# create an empty list and append the date from the request using a for loop
info = list()
for i in data ['main']:
    val = str(data['main'][i])
    info.append(i + ': ' + val)

# render template displays info to users
return render_template('data.html', info=info)
```

---

### AWS CloudWatch

Health Checks monitoring container cpu/memory utilization using to ensure usage stays below 80%. If usage increases above 80% we'll consider increasing Fargate instance size or adding more instances. Also monitoring the number of active containers. Our app has two, so we're alerted when we have fewer than two instances which will indicate a potential issue that needs to be addressed. 

---

### Threat Modeling 

When identifying threats, we realize a bad actor within the company can alter the Prod environment. Since we take requests from an end user, a denial of service attack is possible which would render our product unstable.
