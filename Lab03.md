# Lab session #3: Basic use of the cloud (1/2)

#  Tasks for Lab session #3

Go to the AWS Academy course named "AWS Academy Cloud Foundations" and follow the laboratories shown below.

Provide screeenshots of the major milestones and your explanation about what you have learned or observed. It does not have to be a repetition of the course but your own thoughts and conclusions. Write your feedback in the README.md file linking the images appropiatedly.

## Task 3.1: Introduction to AWS IAM

Follow the lab session: **Module 4 - AWS Cloud Security** *Lab 1 - Introduction to AWS IAM*

## Task 3.2: Build your VPC and Launch a Web Server

Follow the lab session: **Module 5 - Networking and Content Delivery** *Lab 2 - Build your VPC and Launch a Web Server*

## Task 3.3: Introduction to Amazon EC2

Follow the lab session: **Module 6 - Compute** *Lab 3 - Introduction to Amazon EC2*

## Task 3.4: use AWS Comprehend 

Go to the course "AWS Academy Learner Lab", open the modules and open the "Learner Lab". Click the button "Start Lab", wait until the environment is up and then click "AWS" at the top of the window and open the AWS Console.

You may want to check the module **Sandbox** *Sandbox Environment* to find some instructions on how to use the sandbox environment.

It may be the case that the lab is not visible for the browser that you may be using. Try using a different browser.

<p align="center"><img src="./images/Lab03-LearnerLab.jpeg " alt="Learner lab" title="Learner lab"/></p>

Once the AWS Console is open find the service "Amazon Comprehend"

<p align="center"><img src="./images/Lab03-AWSConsole.jpeg " alt="Learner lab" title="Learner lab"/></p>

Launch the demo
<p align="center"><img src="./images/Lab03-AWSComprehend.jpeg " alt="Learner lab" title="Learner lab"/></p>

Play with the demo using the provided text and parts of the "First contact with tensor flow" book.

<p align="center"><img src="./images/Lab03-AWSComprehendPlay.jpeg" alt="Learner lab" title="Learner lab"/></p>

## Task 3.5: Create a python script that sends a text to AWS Comprehend and retrieves the analysis

Using the "AWS Academy Learner Lab" AWS console it is only posible to execute python code in the browser terminal of the environment.

### Create a Personal Access Token on GitHub

From your GitHub account, 

1. go to Settings 
2. Developer Settings 
3. Personal Access Token 
4. Generate New Token (Classic)
5. Fillup the form 
6. Click Full control of private repositories
7. Click Generate token
8. Copy the generated Token, it will be something like ``ghp_sFhFsSHhTzMDreGRLjmks4Tzuzgthdvfsrta``

Keep the token on a safe place for future use.


### Clone your repository

Use the following command to clone your repository inside the AWS command line environment. For the password use the Personal Access Token created before.

```bash
_$ git clone https://github.com/CCBDA-UPC/2024-3-XX.git Lab3
Cloning into 'Lab3'...
Username for 'https://github.com': YOUR@EMAIL.COM
Password for 'https://YOUR@EMAIL.COM@github.com': ghp_sFhFsSHhTzMDreGRLjmks4Tzuzgthdvfsrta
remote: Enumerating objects: 9, done.
remote: Counting objects: 100% (9/9), done.
remote: Compressing objects: 100% (7/7), done.
remote: Total 9 (delta 1), reused 3 (delta 0), pack-reused 0
Unpacking objects: 100% (9/9), done.
Checking connectivity... done.
```

### Add a script to interact with AWS Comprehend

Insert the following code into `Comprehend1.py`.
```python
import json
import boto3

TEXT = 'The area of Machine Learning has shown a great expansion thanks to the co-development of key areas such as computing, massive data storage and Internet technologies. Many of the technologies and events of everyday life of many people are directly or indirectly influenced by automatic learning. Examples of technologies such as speech recognition, image classification on our phones or detection of spam emails, have enabled apps that a decade ago would have only sounded possible in science fiction. The use of learning in stock market models or medical models has impacted our society massively. In addition, cars with cruise control, drones and robots of all types will impact society in the not too distant future. Deep Learning, a subtype of Machine Learning, has undoubtedly been one of the fields which has had an explosive expansion since it was rediscovered in 2006. Indeed, many of the startups in Silicon Valley specialize in it, and big technology companies like Google, Facebook, Microsoft or IBM have both development and research teams. Deep Learning has generated interest even outside the university and research areas: a lot of specialized magazines (like Wired) and even generic ones (such as New York Times, Bloomberg or BBC) have written many articles about this subject. This interest has led many students, entrepreneurs and investors to join Deep Learning. Thanks to all the interest generated, several packages have been opened as "Open Source". Being one of the main promoters of the library we developed at Berkeley (Caffe) in 2012 as a PhD student, I can say that TensorFlow, presented in this book and also designed by Google (California), where I have been researching since 2013, will be one of the main tools that researchers and SME companies will use to develop their ideas about Deep Learning and Machine Learning. A guarantee of this is the number of engineers and top researchers who have participated in this project, culminated with the Open Sourcing. I hope this introductory book will help the reader interested in starting their adventure in this very interesting field. I would like to thank the author, whom I have the pleasure of knowing, the effort to disseminate this technology. He wrote this book (first Spanish version) in record time, two months after the open source project release was announced. This is another example of the vitality of Barcelona and its interest to be one of the actors in this technological scenario that undoubtedly will impact our future.'
comprehend = boto3.client('comprehend')
entity_list = comprehend.detect_entities(Text=TEXT, LanguageCode='en')
print(json.dumps(entity_list, indent=4))
for entity in entity_list['Entities']:
    print(f"{entity['Text']}: {entity['Type']}, {entity['Score']}")
exit()
```

Commit the file to the session repo and push it. At the AWS console pull the code and execute it.

```bash
_$ git pull
_$ python Comprehend1.py
{
    "Entities": [
        {
            "Score": 0.9569487571716309,
            "Type": "DATE",
            "Text": "a decade ago",
            "BeginOffset": 435,
            "EndOffset": 447
        },
        ......
        {
            "Score": 0.521251380443573,
            "Type": "QUANTITY",
            "Text": "actors",
            "BeginOffset": 2437,
            "EndOffset": 2443
        }
    ],
    "ResponseMetadata": {
        "RequestId": "0a4374e6-5bf9-43f5-8f12-35bae9b1e1ec",
        "HTTPStatusCode": 200,
        "HTTPHeaders": {
            "x-amzn-requestid": "0a4374e6-5bf9-43f5-8f12-35bae9b1e1ec",
            "content-type": "application/x-amz-json-1.1",
            "content-length": "3120",
            "date": "Sun, 12 Mar 2024 12:10:14 GMT"
        },
        "RetryAttempts": 0
    }
}
a decade ago: DATE, 0.9569487571716309
one of the fields: QUANTITY, 0.69606614112854
2006: DATE, 0.9996651411056519
Silicon Valley: LOCATION, 0.9980552196502686
Google: ORGANIZATION, 0.9996868371963501
Facebook: ORGANIZATION, 0.9996793270111084
Microsoft: ORGANIZATION, 0.9996274709701538
IBM: ORGANIZATION, 0.999663233757019
both: QUANTITY, 0.9742820262908936
Deep Learning: TITLE, 0.616136372089386
Wired: TITLE, 0.9581007361412048
New York Times: ORGANIZATION, 0.9418124556541443
Bloomberg: ORGANIZATION, 0.973442554473877
BBC: ORGANIZATION, 0.9957347512245178
Deep Learning: TITLE, 0.9147940278053284
one: QUANTITY, 0.8547863364219666
Berkeley: ORGANIZATION, 0.712485671043396
Caffe: ORGANIZATION, 0.5989633202552795
2012: DATE, 0.9998225569725037
TensorFlow: ORGANIZATION, 0.814161479473114
Google: ORGANIZATION, 0.9990465044975281
California: LOCATION, 0.9177584052085876
2013: DATE, 0.9777125120162964
one: QUANTITY, 0.9882826209068298
Deep: TITLE, 0.7469157576560974
first: QUANTITY, 0.9794182777404785
Spanish: OTHER, 0.8972253203392029
two months: QUANTITY, 0.8186834454536438
Barcelona: LOCATION, 0.8517923951148987
one: QUANTITY, 0.9653879404067993
actors: QUANTITY, 0.521251380443573
```

### Play with the AWS Comprehend API

Using the [Boto3 Comprehen API](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/comprehend.html#client) and the `FirstContactWithTensorFlow.txt` text create some analysis bearing in mind the restrictions on document size that the API has.

- Explain what is that you want to obtain
- Add a file named ``Comprehend2.py`` to implement that idea



# How to submit this assignment:

How long have you been working on this session? What have been the main difficulties you have faced and how have you solved them? Add your answers to README.md.

Use the **private** repo named *https://github.com/CCBDA-UPC/2024-3-xx*. It needs to have, at least, two files `README.md` with your responses to the above questions and `authors.json` with both members email addresses:

```json5
{
  "authors": [
    "FIRSTNAME1.LASTNAME1@estudiantat.upc.edu",
    "FIRSTNAME2.LASTNAME2@estudiantat.upc.edu"
  ]
}
```


Make sure that you have updated your local GitHub repository (using the `git`commands `add`, `commit` and `push`) with all the files generated during this session. 

**Before the deadline**, all team members shall push their responses to their private *https://github.com/CCBDA-UPC/2024-3-xx* repository.