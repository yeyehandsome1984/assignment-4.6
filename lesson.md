## Brief

### Preparation

Write about any preparations needed for the lesson, such as tools, installations, prior-knowledge, etcs.

### Lesson Overview

Write about how instructors can brief the students at the start of the lesson. It is good to guide students through what is going to be covered and the outcome. Setting expectations.

---

## Part 1 - Intro Continuous Delivery

### CD Pipeline

The Continuous Delivery Pipeline typically contains the following jobs:

1. Detect there is a release
1. Pull respective image from container registry
1. Deploy the container to the respective environments

### Environments

Usually, there are three types of environments:

**Staging** - An environment to perform User Acceptance Test by stakeholders before deploying to production.

**Production** - An environment where real users consumes the software (web app or mobile app).

**Disaster Recovery** - In case of natural disaster such as earthquake or fire on the production physical servers, Disaster Recovery enviroment will be activated to replace the production. 


### *Activity - Research on Infrastructure as Code (IaC)*

IaC is a technological tool that enable the CD Pipeline to spin up cloud resources for deployment purpose. Based on your knowledge on how the different environment works, perform a research as a team to answer the following questions:

|Question|Answer|
|-|-|
|What are the technologies for IaC?|Your input|
|What are the use case of IaC in the context of CD Pipeline?|Your Input|

---

## Part 2 - Prepare Heroku

Step 1: Setup an account in [Heroku](https://heroku.com) for deployment. 

> Note that Heroku requires you to pay USD $0.01 per hour (max of $7 per month) as the free tier of Heroku Dyno is no longer available from 28 Nov 2022. Read this [link](https://devcenter.heroku.com/articles/free-dyno-hours). You can simply destroy the resources right after this lesson to pay the minimum (< SGD 0.10). However, if you do not wish to pay for the deployment, you may skip part 3 and observe instructor's demonstration.

Step 2: [Create an app](https://dashboard.heroku.com/new-app?org=personal-apps) and name it in this convention `<your_name>-su-devops`

Step 3: You will need to setup the following environmental variables in [CircieCI](https://app.circleci.com/)

- `HEROKU_APP_NAME` with value `<your_name>-su-devops` (Step 2)
- `HEROKU_API_KEY` by generating the API Key [here](https://dashboard.heroku.com/account)(scroll down).

---

## Part 3 - CD Configuration

### Brief

In the CD Pipeline, we will be using `heroku cli` instead of Circle CI Orbs to perform deployment. 

The ideal CICD workflow (not included in this lesson) is connected by the image registry. The CI workflow ends by publishing an image to container registry, and the CD workflow would pick up the published image from container registry and deploy to a container hosted environment. 

To simplify understand CD, we would be creating a CD workflow that is disconnected with the CI workflow you have learned in the previous lesson. In this CD workflow, we will:

1. Build a fresh copy of image from the `Dockerfile` stored within the root directory of the GIT Repository,

1. Push to the Heroku Container Registry (instead of the Docker Container Registry),

1. Release the last pushed image to the Heroku App.

### Implementing CD Pipeline

Let us create a new job call `deploy` as follows:

```yml
jobs:
    deploy:
        docker:
        - image: cimg/node:16.10
        steps:
        - setup_remote_docker      
        - heroku/install
        - checkout
        - run:
            name: Heroku Container Push
            command: | 
                heroku container:login
                heroku container:push web -a edison-su-devops
                heroku container:release web -a edison-su-devops
```

In this job, we did not use a typical Heroku Orbs approach but using the `run` keyword to enter bash script. In an actual production pipeline config, there will be bound to have mixture of Orbs and bash script usage.

Let us configure the workflow to run the `deploy` job after `build-and-push` job is completed:

```yml
workflows:
  simple_workflow:
    jobs:
      - build
      - test:
          requires:
            - build
      - build-and-push:
          requires:
            - test
      - deploy:
          requires:
            - build-and-push
```

> It is assumed that you have the `build-and-push` job configured successfully from the previous assignment. If otherwise, please take note to include the `build-and-push` script in the following complete `config.yml` file.

The final config.yml file will look like this:

```yml
version: 2.1
orbs:
  node: circleci/node@5.0.1
  docker: circleci/docker@2.1.4
jobs:
  build:
    docker:
      - image: cimg/node:16.10
    steps:
      - checkout
      - node/install-packages:
          pkg-manager: npm
      - run: |
          echo Installing dependencies...”
          npm install
  test:
    docker:
      - image: cimg/node:16.10
    steps:
      - checkout
      - node/install-packages:
          pkg-manager: npm
      - run: |
          echo “Running tests...”
          npm run test
  
  build-and-push:
    executor: docker/docker
    steps:
      - setup_remote_docker
      - checkout
      - docker/check
      - docker/build:
          image: edisonzsq/education-space
      - docker/push:
          image: edisonzsq/education-space
  deploy:
    docker:
        - image: cimg/node:16.10
    steps:
        - setup_remote_docker      
        - heroku/install
        - checkout
        - run:
            name: Heroku Container Push
            command: | 
                heroku container:login
                heroku container:push web -a edison-su-devops
                heroku container:release web -a edison-su-devops

workflows:
  simple_workflow:
    jobs:
      - build
      - test:
          requires:
            - build
      - build-and-push:
          requires:
            - test

```

End