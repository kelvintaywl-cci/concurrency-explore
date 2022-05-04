# concurrency-explore

Exploring the user experience of concurrency limits on CircleCI

## Context

The `kelvintaywl-cci` GitHub organization is on the Free plan for CircleCI.
As such, the concurrency for Linux builds (including Docker executors) is **30**.


## Experiment 1

I tried setting up the following config:

```yaml
version: 2.1

jobs:
  foo:
    # concurrency limit at 30
    parallelism: 31
    docker:
      - image: cimg/base:stable
    resource_class: small
    steps:
      - run:
          command: sleep 10
      - run:
          command: echo "done for Node ${CIRCLE_NODE_INDEX} / ${CIRCLE_NODE_TOTAL}"

workflows:
  main:
    jobs:
      - foo
```

We noticed on CircleCI UI that this job **did not run at all**, understandably.
> Ref: https://app.circleci.com/pipelines/github/kelvintaywl-cci/concurrency-explore/1/workflows/1dd606a4-0b5e-49de-9e9c-9695c2bfb9d8/jobs/1

<img width="758" alt="Screen Shot 2022-05-04 at 15 24 07" src="https://user-images.githubusercontent.com/2164346/166639595-8f1b34ae-4689-45ae-aded-0c5acad9d7cd.png">

## Experiment 2

We then break up `foo` job to be flexible in accepting the parallelism number.

As such, we can create concurrent jobs with the following too:

```yaml
version: 2.1

jobs:
  foo:
    # concurrency limit at 30
    parameters:
      size:
        type: integer
        default: 15
    parallelism: << parameters.size >>
    docker:
      - image: cimg/base:stable
    resource_class: small
    steps:
      - run:
          command: sleep 10
      - run:
          command: echo "done for Node ${CIRCLE_NODE_INDEX} / ${CIRCLE_NODE_TOTAL}"

workflows:
  main:
    jobs:
      - foo:
          size: 15
      - foo:
          size: 16
```

Interestingly, and expectedly, the 2nd `foo` bar with `parallelism: 16` was stopped.
> Ref: https://app.circleci.com/pipelines/github/kelvintaywl-cci/concurrency-explore/3/workflows/0488cade-c0c2-433b-8d83-cb7bcdd7f71b/jobs/4

A banner indicated that this job was waitlisted due to concurrency limit.

<img width="813" alt="Screen Shot 2022-05-04 at 15 28 50" src="https://user-images.githubusercontent.com/2164346/166639956-29f8133f-91ab-42ff-b42c-5c4f8312e2ea.png">
