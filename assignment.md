## Assignment

### Brief

You have now successfully configured a workflow that contains CI and CD jobs. 

CI Jobs:
- build
- test

CD Jobs:
- build-and-push
- deploy

In this assignment, let us take a baby step to configure our `config.yml` file a little closer to a production requirement:

1. Only run `build` and `test` jobs when `main` branch is updated.
1. Only run `build-and-push` and `deploy` jobs when a [Git Tag](https://www.atlassian.com/git/tutorials/inspecting-a-repository/git-tag) is being created for Semantic Versioning (covered in lesson 4.4).
1. Include tag (version number) in the `docker/build` and `docker/push` commands under `build-and-push` job. You have to use [Circle CI Parameter](https://circleci.com/docs/pipeline-variables/)

### Answer

```yml
version: 2.1
orbs:
  node: circleci/node@5.0.1
  docker: circleci/docker@2.1.4
  heroku: circleci/heroku@2.0.0
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
          tag: <<pipeline.git.tag>>
      - docker/push:
          image: edisonzsq/education-space
          tag: <<pipeline.git.tag>>

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
      - build:
          filters:
            branches:
              only: main
      - test:
          requires:
            - build 
          filters:
            branches:
              only: main     
      - build-and-push:  
          filters:
            tags:
              only: /^v.*/  
            branches: 
              ignore: /.*/
      - deploy:
          requires:
            - build-and-push
          filters:
            tags:
              only: /^v.*/
            branches: 
              ignore: /.*/

```

### Submission 

- Submit the URL of the GitHub Repository that contains your work to NTU black board.
- Should you reference the work of your classmate(s) or online resources, give them credit by adding either the name of your classmate or URL. 

### References

_Example of Referencing Classmate_

Referenced the code block below from Terence.
```js
    function printMe(){
        console.log("I am a reference example");
    }
```

_Example of Referencing Online Resources_

- https://developer.mozilla.org/en-US/
- https://www.w3schools.com/html/
- https://stackoverflow.com/questions/14494747/how-to-add-images-to-readme-md-on-github

