# NIF Validator
Python NIF Validator


## Build
        docker build -t nif-validator .

## Deliver

        docker tag nif-validator vneves85/nif-validator
        docker login -u username -p password
        docker push vneves85/nif-validator

## Run

        docker run -d --name nif-validator -p 8000:9056 vneves85/nif-validator
