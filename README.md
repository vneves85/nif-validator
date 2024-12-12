# NIF Validator
Python NIF Validator


## Build
        docker build -t nif-validator .

## Deliver

        docker tag nif-validator vneves85/nif-validator
        docker login -u username -p password
        docker push vneves85/nif-validator