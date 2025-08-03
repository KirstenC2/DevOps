stages:
  - build
  - test
  - deploy

build_job:
  stage: build
  image: alpine:latest
  script:
    - chmod +x run.sh
    - ./run.sh
  tags:
    - test

test_job:
  stage: test
  image: alpine:latest
  script:
    - chmod +x run.sh
    - ./run.sh
  tags:
    - test

deploy_job:
  stage: deploy
  image: alpine:latest
  script:
    - chmod +x run.sh
    - ./run.sh
  tags:
    - test