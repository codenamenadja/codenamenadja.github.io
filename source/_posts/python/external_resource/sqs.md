---
title: django로 sqs자원 사용하기
p: python/external_resource/sqs
date: 2019-05-27 15:39:22
tags: ['sqs', 'aws']
---

1. que네임을 기반으로 url받아오기
    ```python
    def helper_get_que(key):
        sqs = boto3.client('sqs')
        response = sqs.list_queues()
        urlLists = response["QueueUrls"]
        for url in urlLists:
           if key in url:
               return url
    ```

2.  url을 엔트포인트로 메세지 발송
    ```python
    def helper_send_message(url, title, author, body):
        sqs = boto3.client('sqs')
        reponse = sqs.send_message(
            QueueUrl=url,
            DelaySeconds=10,
            MessageAttributes={
                'Title': {'DataType': 'String', 'StringValue': title},
                'Author': {'DataType': 'String', 'StringValue': author},
                'WeeksOn': {'DataType': 'Number', 'StringValue': '6'},
                           },
        MessageBody=(f"{body}")
        )
        return reponse
    ```
3. queue가 존재하지 않을때 만들기
    ```python
    def helper_create_queue(name):
        sqs = boto3.client('sqs')
        response = sqs.create_queue(
            QueueName=f"{name}",
            Attributes={'DelaySeconds': '60',
                        'MessageRetentionPeriod': '86400'
                        }
        )
        return response
    ```
4. 메세지 받아오기
    ```python
    def helper_get_message(endpoint):
        sqs = boto3.client('sqs')

        queue_url = helper_get_que(endpoint)
        response = sqs.receive_message(
            QueueUrl=queue_url,
            AttributeNames=[
                'SentTimestamp'
            ],
            MaxNumberOfMessages=1,
            MessageAttributeNames=[
                'All'
            ],
            VisibilityTimeout=0,
            WaitTimeSeconds=0
        )
        return response['Messages'][0]
    ```