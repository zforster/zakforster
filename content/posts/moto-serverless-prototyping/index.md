+++
date = "2025-06-29T20:00:00+01:00"
draft = false
title = "Build Rapid AWS Serverless Prototypes with Moto + Python 🚀"
categories = ['Python', 'Software Engineering', 'Moto', 'AWS']
+++

The majority of applications I build utilise AWS. I value how quickly I can spin up a new database or bucket; it enables me to respond to stakeholder requests rapidly. There are times, however, where I know I will deploy my application on AWS, but I am in an early prototyping stage. At such a stage, deploying real infrastructure isn't necessary.

When prototyping, I build a rough, locally running sample that stakeholders can interact with. My primary objective is to collate early feedback that guides me toward the correct implementation. The key here is speed and iteration. I do not want to be defining and re-defining infrastructure in response to feedback, or paying for services I might throw away tomorrow.

For early stage prototyping I want access to mocked implementations of the AWS services I will eventually use. I have found [Moto](https://docs.getmoto.org/en/latest/) to be an incredibly useful tool for this; it creates mocked implementations of AWS services, keeping momentum to iterate quickly without the overhead of deploying real infrastructure.

## Introducing Moto

[Moto](https://docs.getmoto.org/en/latest/) is a popular Python library used by developers to mock out calls to AWS services. Moto's functionality and ease of use have made it the preferred library for Python engineers seeking to unit test their applications without deploying real resources in AWS.

Moto doesn’t have full coverage of all AWS services. However, it is actively developed and supports much of the functionality within many of the most commonly used services. It especially shines for services commonly used in a serverless stack such as DynamoDB and S3, with 68% and 72% coverage respectively (at the time of writing). The full list of implementation details are [available here.](https://github.com/getmoto/moto/blob/master/IMPLEMENTATION_COVERAGE.md)

## Moto Server Mode

A lesser known feature of Moto is [server mode.](https://docs.getmoto.org/en/latest/docs/server_mode.html) Server mode makes Moto not only a great testing tool, but also a great prototyping tool.

The main purpose of server mode is to allow developers to use Moto with any of the official AWS SDKs, not just Python. However, I have found server mode to be useful when looking to rapidly prototype and validate ideas locally.

With server mode, you can rapidly iterate on a local API instance, sharing concepts with stakeholders early on, allowing them to experience something tangible and provide feedback without the overhead of deploying real infrastructure. I find this perfect for fast-paced working environments where time to deliver is critical and requirements can change.

The great thing about server mode is that unlike the Moto decorators, mocked AWS services are fully decoupled from your locally hosted API instance, meaning the data is persisted across API restarts, eliminating the need to re-seed upon restart, unless of course your data model changes.

## Exploring a Practical Example

In the below example, Moto server mode is used as part of a local development environment within a FastAPI application. Rather than calling real AWS services, Moto mocks them, allowing us to run a local DynamoDB instance that we can use to store a collection of user assets.

### Installing Moto & Initialising the Server

To get started with Moto, we need to install the required dependencies. Server mode is an optional dependency of Moto, so we need to explicitly specify we want to install it.

```shell
pip install moto[server]
```

Once installed, we simply start the server. We can also specify the port we want to run on with the argument `-p`. Our Moto server must be running before we initalise the FastAPI server as we create the tables on startup.

```shell
moto_server -p3000
```

### Using Moto Within FastAPI

My main entry point creates a FastAPI server instance bound to port 8000 using uvicorn. Before the server starts, a boto3 DynamoDB client is created, pointed to the Moto server instance created above. The default endpoint_url has been replaced with `http://localhost:3000`. This is to tell the client to use the local Moto server, not real AWS services.

Once we have our client, an in-memory table `AssetBasket` is created inside of our local Moto server instance. I've wrapped this in a `try / except` block as if I was to restart the FastAPI server whilst Moto server was running, a `botocore` exception would be raised as the table already exists on the server instance.

```python
import boto3
from botocore.exceptions import ClientError
import uvicorn

if __name__ == "__main__":
    try:
        dynamodb = boto3.client(
            "dynamodb",
            "eu-west-1",
            endpoint_url="http://localhost:3000",  # mocked Moto instance
        )
        table = dynamodb.create_table(
            TableName="AssetBasket",
            KeySchema=[
                {"AttributeName": "PK", "KeyType": "HASH"},
                {"AttributeName": "SK", "KeyType": "RANGE"},
            ],
            AttributeDefinitions=[
                {"AttributeName": "PK", "AttributeType": "S"},
                {"AttributeName": "SK", "AttributeType": "S"},
            ],
            ProvisionedThroughput={
                "ReadCapacityUnits": 5,
                "WriteCapacityUnits": 5
            },
        )
    except ClientError as e:
        pass

    uvicorn.run(
        "src.api.app:app",
        host="127.0.0.1",
        port=8000,
        reload=True,
    )
```

I then have an `AssetRepository` class responsible for all database interactions between our application code and the `AssetBasket` DynamoDB table. The class has a function `add_asset_meta`, which allows us to push new asset metadata into our Dynamo table. Again, we set the `endpoint_url` with the url of our mocked services running on our Moto server.

```python
import boto3
from boto3.dynamodb.conditions import Key

from domain.model.asset import AssetMeta


class AssetRepository:
    def __init__(self):
        self.dynamodb = boto3.resource(
            "dynamodb",
            "eu-west-1",
            endpoint_url="http://localhost:3000",
        )
        self.table = self.dynamodb.Table("AssetBasket")

    def add_asset_meta(self, asset_meta: AssetMeta) -> None:
        self.table.put_item(Item=asset_meta.to_dynamo())
```

Finally, I create a file `api/asset/app.py` containing the `/asset` API routes. The API route is de-coupled from our application logic with a service layer function responsible for converting our input model to an `AssetMeta` object and orchestrating the call to the `AssetRepository` class. A service layer acts as a bridge between the API and repository layer to keep responsibilities cleanly segregated.

```python
from fastapi import APIRouter, status

import api.asset.model.input as api_model_input
import api.asset.model.output as api_model_output
import api.asset.service_layer as service_layer
from domain.repository.asset import AssetRepository

router = APIRouter(
    prefix="/asset",
    tags=["Asset"],
)


@router.post("/", status_code=status.HTTP_201_CREATED)
def add_asset_meta(asset_meta: api_model_input.AddAsset) -> api_model_output.AssetMetaResponse:
    return service_layer.add_asset(
        asset_meta=asset_meta,
        asset_repository=AssetRepository(),
    )
```

That's all the setup we need to do, from there we can call functions within our asset repository, and as long as the moto server is running we have our locally mocked DynamoDB table available for our prototyping needs.

### Testing the Endpoint

We now have our Moto server initialised, our mocked table created, and our FastAPI instance running with our `/asset` handler. We can now `POST` some data to our API and have it saved to our mock table.

#### Calling the API
```bash
curl -X POST http://127.0.0.1:8000/asset/ \
  -H "Accept: application/json" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "S&P500",
    "tag": "ISA",
    "liquidityProfile": "HIGH",
    "ccy": "GBP"
  }'
```

As expected, our data was added to the database, and we receive the response;

```json
{
  "identifier": "94e4e459-c7a5-4ed6-b301-6d1c6ac02f17"
}
```

## Summary

Moto's server mode provides developers with a means to rapidly prototype API services without needing to deploy real AWS cloud resources. Whilst Moto is not a replacement for dedicated user testing environments, it is perfect for cases where you need to iterate fast and validate ideas early. 

With Moto you can quickly build small, focused experiments and demonstrations that help facilitate conversations about the direction of the product you are building. In a world where delayed feedback often means building the wrong thing, that is something I believe is very powerful.

---

{{< newsletter >}}

---
